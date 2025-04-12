## 状态管理

#### 1. redux

**1.1 核心概念**

- store，存放全局状态的地方，统一管理状态
- action，对应一个JS对象，描述更新的type和content，通过dispacth来派发，让状态更改可追踪
- reducer，对应一个纯函数，将传入的state和action进行结合，返回值作为新的state

**1.2 redux使用**

1. 定义state和reducer

```javascript
import * as actionTypes from "./constants"

const initialState = {
  counter: 100,
  
  banners: [],
  recommends: []
}

function reducer(state = initialState, action) {
  switch (action.type) {
    case actionTypes.ADD_NUMBER:
      return { ...state, counter: state.counter + action.num }
    case actionTypes.SUB_NUMBER:
      return { ...state, counter: state.counter - action.num }
    case actionTypes.CHANGE_BANNERS:
      return { ...state, banners: action.banners }
    case actionTypes.CHANGE_RECOMMENDS:
      return { ...state, recommends: action.recommends }
    default:
      return state
  }
}

export default reducer
```

2. 创建actionCreators

```javascript
import * as actionTypes from "./constants"

export const addNumberAction = (num) => ({
  type: actionTypes.ADD_NUMBER,
  num
})

export const subNumberAction = (num) => ({
  type: actionTypes.SUB_NUMBER,
  num
})


export const changeBannersAction = (banners) => ({
  type: actionTypes.CHANGE_BANNERS,
  banners
})

export const changeRecommendsAction = (recommends) => ({
  type: actionTypes.CHANGE_RECOMMENDS,
  recommends
})
```

3. 创建action类型常量

```javascript
export const ADD_NUMBER = "add_number"
export const SUB_NUMBER = "sub_number"

export const CHANGE_BANNERS = "change_banners"
export const CHANGE_RECOMMENDS = "change_recommends"
```

4. 创建store

```javascript
import { createStore, applyMiddleware, compose } from "redux"
import thunk from "redux-thunk"
import reducer from "./reducer"

const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({trace: true}) || compose;
const store = createStore(reducer, composeEnhancers(applyMiddleware(thunk)))

export default store
```

**1.3 派发异步action**

1. 使用中间件，组合函数

```javascript
import { createStore, applyMiddleware, compose } from "redux"
import thunk from "redux-thunk"
import reducer from "./reducer"

// 正常情况下 store.dispatch(object)
// 想要派发函数 store.dispatch(function)

// redux-devtools
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({trace: true}) || compose;
const store = createStore(reducer, composeEnhancers(applyMiddleware(thunk)))

export default store
```

2. 创建异步函数，接收distpacth和getState，通过异步函数作为中间件获取数据，再通过dispacth派发事件

```javascript
export const fetchHomeMultidataAction = () => {
  // 如果是一个普通的action, 那么我们这里需要返回action对象
  // 问题: 对象中是不能直接拿到从服务器请求的异步数据的
  // return {}

  return function(dispatch, getState) {
    // 异步操作: 网络请求
    // console.log("foo function execution-----", getState().counter)
    axios.get("http://123.207.32.32:8000/home/multidata").then(res => {
      const banners = res.data.data.banner.list
      const recommends = res.data.data.recommend.list

      // dispatch({ type: actionTypes.CHANGE_BANNERS, banners })
      // dispatch({ type: actionTypes.CHANGE_RECOMMENDS, recommends })
      dispatch(changeBannersAction(banners))
      dispatch(changeRecommendsAction(recommends))
    })
  }

  // 如果返回的是一个函数, 那么redux是不支持的
  // return foo
}
```

**1.4 分模块使用**

```javascript
// 将两个reducer合并在一起，将多个reducer合并返回一个新的函数，作为合并之后的reducer
const reducer = combineReducers({
  counter: counterReducer,
  home: homeReducer,
  user: userReducer
})
```

**1.5 在react中使用**

在组件中使用

- 直接使用store

```javascript
this.state = {
  counter: store.getState().counter.counter
}

componentDidMount() {
    store.subscribe(() => {
      const state = store.getState().counter
      this.setState({ counter: state.counter })
    })
}
```

- 通过connect高阶组件进行映射，柯里化

```javascript
const mapStateToProps = (state) => ({
  counter: state.counter
})

const mapDispatchToProps = (dispatch) => ({
  changeBanners(banners) {
    dispatch(changeBannersAction(banners))
  },
  changeRecommends(recommends) {
    dispatch(changeRecommendsAction(recommends))
  }
})

export default connect(mapStateToProps, mapDispatchToProps)(Category)
```

#### 2. reduxtoolkit，简化redux在react中的使用

1. 创建slice

```javascript
import { createAsyncThunk, createSlice } from "@reduxjs/toolkit"
import TEntireState from "./type"
import { getEntireRoomList } from "@/service"
import { TRootState } from "@/store"

export const fetchEntireDataAction = createAsyncThunk(
  "fetchEntireData",
  (page: number, { getState, dispatch }) => {
    const state = getState() as TRootState
    const size = state.entire.pageSize
    const offset = page * size
    dispatch(changeIsLoadingAction(true))
    getEntireRoomList(offset, size).then((res) => {
      dispatch(changeIsLoadingAction(false))
      dispatch(changeCurrentPageAction(page))
      dispatch(changeTotalCountAction(res.totalCount))
      dispatch(changeRoomListAction(res.list))
    })
  }
)

const initialState: TEntireState = {
  currentPage: 0,
  pageSize: 20,
  totalCount: 0,
  roomList: [],
  isLoading: false
}

const entireSlice = createSlice({
  name: "entire",
  initialState,
  reducers: {
    changeCurrentPageAction(state, { payload }) {
      state.currentPage = payload
    },
    changeTotalCountAction(state, { payload }) {
      state.totalCount = payload
    },
    changeRoomListAction(state, { payload }) {
      state.roomList = payload
    },
    changeIsLoadingAction(state, { payload }) {
      state.isLoading = payload
    }
  }
})

export const {
  changeCurrentPageAction,
  changeTotalCountAction,
  changeRoomListAction,
  changeIsLoadingAction
} = entireSlice.actions
export default entireSlice.reducer
```

2. 合并reducer，创建store

```javascript
import { configureStore } from "@reduxjs/toolkit"
import { shallowEqual, useDispatch, useSelector } from "react-redux"
import type { TypedUseSelectorHook } from "react-redux"
import homeReducer from "./modules/home"
import entireReducer from "./modules/entire"
import detailReducer from "./modules/detail"
import mainReducer from "./modules/main"

const store = configureStore({
  reducer: {
    home: homeReducer,
    entire: entireReducer,
    detail: detailReducer,
    main: mainReducer
  }
})

export type TRootState = ReturnType<typeof store.getState>
type AppDispatch = () => typeof store.dispatch

export const useAppSelector: TypedUseSelectorHook<TRootState> = useSelector
export const useAppDispatch: AppDispatch  = useDispatch
export const appShallowequal = shallowEqual

export default store
```

#### 2. pinia

pinia作为Vue的一个插件来使用，核心依赖于Vue的响应式系统进行工作。

- store通过reactive进行创建

- store中的state调用ref和reactive进行依赖追踪
- store中的getters依赖computed进行工作
- store中的actions会集成中store对象中，作为修改state的函数

createPinia => 创建pinia对象

- 核心属性state，存储所有store合并之后的state
- _s，一个Map对象，存储store，storeId => store，缓存store，多次调用useStore返回同一个
- _p，创建plugin

defineStore => return useStore

useStore => createSetupStore

- 拿到setup函数的返回值setupStore，遍历setupStore
  - 把ref和reactive对象添加到pinia.state中，收集到pinia.state
  - actions添加到store对象上
  - getters不处理，依赖computed
- 把setupStore合并到store对象上，store是一个响应式对象

useStore => createOptionsStore，options包裹进setup函数 => createSetupStore 

store其实就是一个单例对象，第一次时会创建并缓存在pinia对象上，所有组件访问的都是同一个store对象，达到状态共享。

#### 3. pinia和vuex和区别

- 移除mutation这个冗余环节
- 更好的TS支持，获得更好的开发体验
