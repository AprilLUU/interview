## 接入文生图 AI扩图大模型

- 创建AI任务，获取任务ID
- 根据任务ID轮询获取结果
- 注意：清理定时器，在获取到任务结果之后，组件销毁时都要清理定时器，避免消耗性能

```javascript
/**
 * 创建任务
 */
const createTask = async () => {
  if (!prompt.value) return
  const code = await pictureStore.createTextToImageTask(prompt.value)
  // 创建成功 开始轮询
  if (code === 0) startPolling()
}

// 轮询定时器
let pollingTimer: ReturnType<typeof setInterval> | null = null
// 清理轮询定时器
const clearPolling = () => {
  if (pollingTimer) {
    clearInterval(pollingTimer)
    pollingTimer = null
    taskId.value = undefined
  }
}

// 开始轮询
const startPolling = () => {
  if (!taskId.value) return

  pollingTimer = setInterval(async () => {
    try {
      const res = await pictureStore.getTextToImageTask()
      if (res?.taskStatus === "SUCCEEDED") {
        clearPolling()
        const params: API.PictureUploadRequest = {
          picName: prompt.value! + Date.now(),
          fileUrl: res.results[0].url,
          spaceId: props.spaceId
        }
        const data = await pictureStore.uploadPitureByUrl(params)
        props.onSuccess?.(data ?? {})
        loading.value = false
      } else if (res?.taskStatus === "FAILED") {
        loading.value = false
        clearPolling()
      }
    } catch (err) {
      loading.value = false
      clearPolling()
    }
  }, 3000) // 每隔 3 秒轮询一次
}

// 清理定时器，避免内存泄漏
onUnmounted(() => clearPolling())
```

## number类型溢出问题

long类型为8个字节的整型，JS number类型采用双精度浮点型，64位，1位符号，11位指数，52位尾数加上隐式的一位表示，最大能安全表示 -(2^53 - 1) ~ 2^53 - 1之间的数

由于后端id字段为long类型，到前端js会发生溢出，后端响应中将long以及int等number类型转换成了string类型
某些非id字段在请求时为number类型，会发生类型不匹配问题

1. 在响应拦截器中将非id字段的为数字的string类型字段转换成number类型
2. 借助AI工具修改openapi生成的TS类型，将id字段改为string

```javascript
// 全局响应拦截器
myAxios.interceptors.response.use(
  function (response) {
    const { data } = response
    // 未登录
    if (data.code === 40100) {
      // 不是登录请求，并且用户目前不是已经在用户登录页面，则跳转到登录页面
      if (
        !response.request.responseURL.includes("user/get/login") &&
        !window.location.pathname.includes("/user/login")
      ) {
        message.warning("请先登录")
        setTimeout(() => {
          window.location.href = `/user/login`
        }, 500)
      }
    }

    // 由于后端id字段为long类型 到前端js会发生溢出 因此在响应中转换成了string类型
    // 但某些字段在请求时为number类型 会发生类型不匹配问题
    // 因此 在响应拦截器中将非id字段的为数字的string类型字段转换成number类型
    if (data.code === 0) {
      // 在某些接口 如注册 添加用户 直接返回的是用户id 此时 不进行转换
      if (typeof data.data !== "object") return data
      const numReg = /^[0-9]+$/
      const idReg = /id/i
      for (const key in data.data) {
        // 含id的字段不转换
        if (idReg.test(key)) continue
        const value = data.data[key]
        if (typeof value === "string" && numReg.test(value)) {
          data.data[key] = Number(value)
        }
      }
    }

    return data
  },
  function (error) {
    message.error("请求失败")
    return error
  }
)
```

## 图片跨域问题

图片编辑组件访问对象存储图片时出现跨域问题（内部使用canvas获取图像导致跨域，img标签的src属性不受跨域限制），在对象存储配置跨域之后仍然出现跨域问题，经过排查是浏览器HTTP缓存的问题，禁用缓存即可恢复使用

## 截图实现原理 html2canvas

1. 获取img元素
2. 创建canvas元素操作img
3. 使用canvas裁剪图片 canvas.drawImage(img)
   截图的区域可以根据截图框相对于图片的偏移量来计算
4. 返回截图的二进制流Blob，canvas.toBlob

- Blob是二进制对象，由type和blob part组成，type标识类型

## Websocket实时编辑

- 采用事件总线的方式，以消息作为事件，监听对应的事件，当收到消息时触发对应的事件处理器
- WebSocket消息类型
  - 五种消息类型
    - 通知，加入编辑
    - 错误
    - 进入编辑
    - 退出编辑
    - 编辑操作
      - 放大，缩小，左旋，右旋

```javascript
export const PICTURE_EDIT_MESSAGE_TYPE_MAP = {
  INFO: "发送通知",
  ERROR: "发送错误",
  ENTER_EDIT: "进入编辑状态",
  EXIT_EDIT: "退出编辑状态",
  EDIT_ACTION: "执行编辑操作"
}

export const PICTURE_EDIT_ACTION_MAP = {
  ZOOM_IN: "放大操作",
  ZOOM_OUT: "缩小操作",
  ROTATE_LEFT: "左旋操作",
  ROTATE_RIGHT: "右旋操作"
}
```

- WebSocket建立连接并监听对应的事件
  - 监听连接成功事件，onopen，触发open事件，心跳机制可以在这里设置，设置定时器，发送空数据包
  - 监听收到消息事件，onmessage，根据消息类型触发对应的事件
  - 监听连接关闭事件，onclose，触发close事件，断线重连可以在这里设置，尝试重新连接，每隔一段时间尝试一次，超过最大次数没有连接成功就停止
  - 监听错误事件，onerror，触发error事件

```javascript
/**
* 初始化 WebSocket 连接
*/
connect() {
    const url = `${WS_BASE_URL}/api/ws/picture/edit?pictureId=${this.pictureId}`
    this.socket = new WebSocket(url)
	
    // 设置二进制数据类型
    this.socket.binaryType = 'blob'

    // 监听连接成功事件
    this.socket.onopen = () => {
      console.log('WebSocket 连接已建立')
      this.triggerEvent('open')
    }

    // 监听消息事件
    this.socket.onmessage = (event) => {
      const message = JSON.parse(event.data)
      console.log('收到消息:', message)

      // 根据消息类型触发对应事件
      const type = message.type
      this.triggerEvent(type, message)
    }

    // 监听连接关闭事件
    this.socket.onclose = (event) => {
      console.log('WebSocket 连接已关闭:', event)
      this.triggerEvent('close', event)
    }

    // 监听错误事件
    this.socket.onerror = (error) => {
      console.error('WebSocket 发生错误:', error)
      this.triggerEvent('error', error)
    }
}
```

- WebSocket断开连接

```javascript
disconnect() {
    if (this.socket) {
      this.socket.close()
      console.log('WebSocket 连接已手动关闭')
    }
}
```

- WebSocket发送消息
  - 连接建立成功才能发送消息，readyState === WebSocket.OPEN

```javascript
sendMessage(message: object) {
    if (this.socket && this.socket.readyState === WebSocket.OPEN) {
      this.socket.send(JSON.stringify(message))
      console.log('消息已发送:', message)
    } else {
      console.error('WebSocket 未连接，无法发送消息:', message)
    }
}
```

- 打开弹窗 => 建立websocket连接 => 后台广播给所有用户加入编辑消息
  - 建立编辑锁，防止冲突，有用户在编辑图片时其他用户禁用图片操作按钮
  - 设置editingUser，根据editingUser是否有值设置编辑权限
    - 没有值，可以进入编辑
    - 编辑用户是本人可以编辑
    - 编辑用户是本人可以退出编辑


```javascript
const loginUserStore = useLoginUserStore()
const loginUser = loginUserStore.loginUser
// 正在编辑的用户
const editingUser = ref<API.UserVO>()
// 没有用户正在编辑中，可进入编辑
const canEnterEdit = computed(() => !editingUser.value)
// 正在编辑的用户是本人，可退出编辑
const canExitEdit = computed(() => editingUser.value?.id === loginUser.id)
// 可以编辑
const canEdit = computed(() => {
  // 私人空间默认可编辑
  if (!isTeamSpace.value) return true
  return editingUser.value?.id === loginUser.id
})

const openModal = () => {
  if (isTeamSpace.value) websocket.connect()
  modalRef.value?.openModal()
}

// 监听通知消息
websocket.on(PICTURE_EDIT_MESSAGE_TYPE_ENUM.INFO, (msg) => {
  console.log("收到通知消息：", msg)
  message.info(msg.message)
})

```

- 开始编辑, 退出编辑 => 发送消息 => 后台广播给所有用户开始/退出编辑消息

```javascript
// 进入编辑状态
const enterEdit = () => {
  if (websocket) {
    // 发送进入编辑状态的消息
    websocket.sendMessage({
      type: PICTURE_EDIT_MESSAGE_TYPE_ENUM.ENTER_EDIT
    })
  }
}

// 退出编辑状态
const exitEdit = () => {
  if (websocket) {
    // 发送退出编辑状态的消息
    websocket.sendMessage({
      type: PICTURE_EDIT_MESSAGE_TYPE_ENUM.EXIT_EDIT
    })
  }
}

// 监听进入编辑状态消息
websocket.on(PICTURE_EDIT_MESSAGE_TYPE_ENUM.ENTER_EDIT, (msg) => {
    console.log("收到进入编辑状态消息：", msg)
    message.info(msg.message)
    editingUser.value = msg.user
})

// 监听退出编辑状态消息
websocket.on(PICTURE_EDIT_MESSAGE_TYPE_ENUM.EXIT_EDIT, (msg) => {
    console.log("收到退出编辑状态消息：", msg)
    message.info(msg.message)
    editingUser.value = undefined
})
```

- 点击编辑操作 => 发送消息 => 后台广播给除编辑用户以外的所有用户编辑操作消息, 前端调用对应方法进行同步

```javascript
// 编辑图片操作
const editAction = (action: string) => {
  if (websocket) {
    // 发送编辑操作的请求
    websocket.sendMessage({
      type: PICTURE_EDIT_MESSAGE_TYPE_ENUM.EDIT_ACTION,
      editAction: action
    })
  }
}
// 缩放
const changeScale = (num: number) => {
  if (num > 0) {
    editAction(PICTURE_EDIT_ACTION_ENUM.ZOOM_IN)
  } else {
    editAction(PICTURE_EDIT_ACTION_ENUM.ZOOM_OUT)
  }
}

// 监听编辑操作消息
websocket.on(PICTURE_EDIT_MESSAGE_TYPE_ENUM.EDIT_ACTION, (msg) => {
    console.log("收到编辑操作消息：", msg)
    message.info(msg.message)
    const cropperRef = pictureEditAreaRef.value?.cropperRef
    switch (msg.editAction) {
      case PICTURE_EDIT_ACTION_ENUM.ROTATE_LEFT:
        cropperRef.value?.rotateLeft()
        break
      case PICTURE_EDIT_ACTION_ENUM.ROTATE_RIGHT:
        cropperRef.value.rotateRight()
        break
      case PICTURE_EDIT_ACTION_ENUM.ZOOM_IN:
        cropperRef.value.changeScale(1)
        break
      case PICTURE_EDIT_ACTION_ENUM.ZOOM_OUT:
        cropperRef.value.changeScale(-1)
        break
    }
})
```

## ESM静态分析
- 应用初始化时，所有import模块均会被加载和执行，直接定义会导致在初始化Pinia之前访问store，导致报错
- 解决：将store的引用放在函数内部，只有在调用时才会执行 （或采用动态加载模块的方式）

```javascript
const getOptions = () => {
  const homeStore = useHomeStore()
  const { tagList, categoryList } = storeToRefs(homeStore)
  const tagOptions = tagList.value.map((data: string) => ({
    label: data,
    value: data
  }))
  const categoryOptions = categoryList.value.map((data: string) => ({
    label: data,
    value: data
  }))

  return { tagOptions, categoryOptions }
}
```

## 颜色搜图，以图搜图

- 颜色搜图，转成RGB的颜色，作为三维向量，计算欧式距离

```java
public static double calculateSimilarity(String hexColor1, String hexColor2) {
    Color color1 = Color.decode(hexColor1);
    Color color2 = Color.decode(hexColor2);
    return calculateSimilarity(color1, color2);
}

int r1 = color1.getRed();
int g1 = color1.getGreen();
int b1 = color1.getBlue();

int r2 = color2.getRed();
int g2 = color2.getGreen();
int b2 = color2.getBlue();

// 计算欧氏距离
double distance = Math.sqrt(Math.pow(r1 - r2, 2) + Math.pow(g1 - g2, 2) + Math.pow(b1 - b2, 2));

// 计算相似度
return 1 - distance / Math.sqrt(3 * Math.pow(255, 2));
```

- 以图搜图，转成向量，计算向量相似度

## 表单组件

- 接收表单数据，与服务器数据相对应
- 接收表单配置，对应一个数组，每项对应一个表单项，定义TS类型向外暴露类型，可进一步扩展
  - type：对应表单类型，和ant-design表单类型对应
  - field：绑定的表单数据字段，与服务器字段一致
- 把接收的表单prop绑定到表单上
- 遍历表单配置，生成表单项，并将传递过来的配置选项绑定到表单项
  - 为了不破坏单向数据流的原则，将表单双向绑定的语法糖进行拆分
    - 直接双向绑定，当表单值改变，会直接修改prop的值
  - v-model拆分为v-bind，当表单值改变时触发change事件，把对应的字段和表单值通过emit触发事件暴露给父组件，由父组件完成修改
- 联动修改
  - 增加依赖的字段，对应一个数组
  - 增加一个onChange函数，传入改变的字段和值，返回需要改变的新值
  - 在依赖的字段发生改变时，遍历配置，找到依赖该字段的表单项，回调onChange函数


```typescript
// 依赖字段
dependencies?: string[]
// 处理联动
onChange?: (field: any, value: any) => any

const handleValueChange = (value: string, field: string) => {
  props.formList.forEach(((item) => {
    if (item.dependencies) {
      if (item.dependencies.includes(field)) {
        const effectVal = item.onChange?.(field, value)
        emit("update:formData", item.field, effectVal)
      }
    }
  }))
  emit("update:formData", field, value)
}
```

- 表单校验
  - 二次封装，默认组件库已支持的功能也可以支持
  - 最简单的方法且最灵活的方式，增加一个校验函数validator，把表单项的值传入进行自定义校验
  - 增加校验规则字段rules，描述校验的规则，根据校验的规则对表单项进行校验
  - 暴露valid方法，验证表单是否校验成功

```typescript
validator: (value: any) => boolean
rules: any[]

rules: [
  { required: true, message: "请输入密码" },
  { min: 8, message: "密码长度不能小于 8 位" }
]

const valid = () => {
    let isValid = true
    
    formList.forEach((item) => {
        const value = formData[field]
        if (item.validator) {
            isValid = item.validator(value)
        }
        if (item.rules) {
            for (const rule of rules) {
                if (rule.required && !value) {
                    isValid = false
                }
                if (rule.min) isValid = Number(value) < min
                // TODO 校验其他规则
                ....
            }
        }
    })
    
    return isValid
}
```

```vue
<script setup lang="ts">
import { ref } from "vue"
import { type FormList } from "../types"

interface Props {
  formData?: any
  formList?: FormList[]
  formLayout?: string
  loading?: boolean
  isShowReset?: boolean
  autoComplete?: string
}

withDefaults(defineProps<Props>(), {
  formData: () => {},
  formList: () => [],
  formLayout: "inline",
  loading: false,
  isShowReset: false,
  autoComplete: "on"
})
    
const emit = defineEmits([
  "update:formData",
  "submit:formSubmit",
  "update:editTime",
  "reset:formData"
])

const handleValueChange = (value: string, field: string) => {
  emit("update:formData", field, value)
}

const handleSubmit = (values: any) => {
  emit("submit:formSubmit", values)
}

const dateRange = ref<any[]>([])

const handleRangeChange = (dates: any[]) => {
  emit("update:editTime", dates)
}

const handleResetClick = () => {
  dateRange.value = []
  emit("reset:formData")
}
</script>

<template>
  <div class="form-area">
    <a-form
      :layout="formLayout"
      :model="formData"
      :autoComplete="autoComplete"
      @finish="handleSubmit"
    >
      <template v-for="item in formList" :key="item.label">
        <a-form-item :label="item.label" :name="item.name" :rules="item.rules">
          <template v-if="item.type === 'input'">
            <a-input
              :value="formData[item.field!]"
              :placeholder="item.placeholder"
              :allow-clear="item.allowClear"
              :style="item.styleObj"
              @update:value="
                (value: string) => handleValueChange(value, item.field!)
              "
            />
          </template>
          <template v-if="item.type === 'input-number'">
            <a-input-number
              :value="formData[item.field!]"
              :placeholder="item.placeholder"
              :allow-clear="item.allowClear"
              :style="item.styleObj"
              :min="item.min"
              :max="item.max"
              @update:value="
                (value: string) => handleValueChange(value, item.field!)
              "
            />
          </template>
          <template v-if="item.type === 'input-password'">
            <a-input-password
              :value="formData[item.field!]"
              :placeholder="item.placeholder"
              :allow-clear="item.allowClear"
              :style="item.styleObj"
              @update:value="
                (value: string) => handleValueChange(value, item.field!)
              "
            />
          </template>
          <template v-if="item.type === 'textarea'">
            <a-textarea
              :value="formData[item.field!]"
              :placeholder="item.placeholder"
              :allow-clear="item.allowClear"
              :style="item.styleObj"
              :autoSize="item.sizeObj"
              @update:value="
                (value: string) => handleValueChange(value, item.field!)
              "
            />
          </template>
          <template v-if="item.type === 'auto-complete'">
            <a-auto-complete
              :value="formData[item.field!]"
              :placeholder="item.placeholder"
              :allow-clear="item.allowClear"
              :style="item.styleObj"
              :options="item.options"
              @update:value="
                (value: string) => handleValueChange(value, item.field!)
              "
            />
          </template>
          <template v-if="item.type === 'select'">
            <a-select
              :value="formData[item.field!]"
              :mode="item.mode"
              :options="item.options"
              :placeholder="item.placeholder"
              :allow-clear="item.allowClear"
              :style="item.styleObj"
              @update:value="
                (value: string) => handleValueChange(value, item.field!)
              "
            />
          </template>
          <template v-if="item.type === 'date-range-picker'">
            <a-range-picker
              :style="item.styleObj"
              v-model:value="dateRange"
              :show-time="item.dateRangePicker?.showTime"
              :placeholder="item.dateRangePicker?.placeholder"
              :format="item.dateRangePicker?.format"
              :presets="item.dateRangePicker?.rangePresets"
              @change="handleRangeChange"
            />
          </template>
          <template v-if="item.type === 'button'">
            <a-button
              :style="item.styleObj"
              type="primary"
              html-type="submit"
              :loading="loading"
            >
              {{ item.name }}
            </a-button>
            <a-button
              v-if="isShowReset"
              class="reset-btn"
              :style="item.styleObj"
              html-type="reset"
              @click="handleResetClick"
            >
              重置
            </a-button>
          </template>
        </a-form-item>
      </template>
    </a-form>
  </div>
</template>

<style scoped>
.reset-btn {
  margin-left: 16px;
}
</style>
```

```typescript
export interface FormList {
  // common
  label?: string
  name?: string
  type?: string
  field?: string
  placeholder?: string
  allowClear?: boolean
  styleObj?: any
  // select
  mode?: string
  options?: any
  // textarea
  sizeObj?: any
  // input-number
  min?: number
  max?: number
  // form-item
  rules?: any
  // date-range-picker
  dateRangePicker?: IDateRangePicker
}



interface IDateRangePicker {
  dateRange?: any[]
  rangePresets?: any[]
  format?: string
  placeholder?: string[]
  showTime?: boolean
}
```

## 权限管理

## Git提交规范

<type>(<scope>)：<subject>

提交的类型

- feat，增加新功能
- fix，bug修复
- doc，修改文档
- style，格式化
- refactor，重构代码

## 性能优化

- 图片压缩，将图片压缩成较小的格式webp
- 缩略图，主页展示缩略图，详情页再展示高清图，减少流量传输，加快传输速度
- 组件懒加载，对应构建工具分包加载，加快首页加载速度
- 浏览器缓存，缓存提高加载速度