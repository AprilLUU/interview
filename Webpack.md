## Webpack

#### 1. 打包CSS，less

loader加载顺序为从右到左

1.1 style-loader将css注入DOM

1.2 css-loader处理css文件

1.3 less-loader处理less文件，转换为css文件，传递给css-loader

1.4 postcss-loader将一些现代的CSS特性，转成大多数浏览器认识的CSS，并且会根据目标浏览器或者运行时环境添加所需的polyfill，自动帮助我们添加autoprefixer

```javascript
{
    test: /\.css$/,
    use: [ "style-loader", "css-loader", "postcss-loader" ]
},
{
    test: /\.less$/,
    use: [ "style-loader", "css-loader", "less-loader", "postcss-loader" ]
},
```

#### 2. 打包图片

asset模块，配置较小的图片转为base64（减少请求数，当图片过大时base64过大，反而影响加载速度），较大的图片存为文件

responsive-loader：根据不同的视口大小生成多种尺寸的图像，适合响应式设计。支持自动生成适合不同屏幕的图片格式。

```javascript
{
    test: /\.(png|jpe?g|svg|gif)$/,
    type: "asset",
    parser: {
      dataUrlCondition: {
        maxSize: 60 * 1024
      }
    },
    generator: {
      filename: "img/[name]_[hash:8][ext]"
    }
}
```

#### 3. babel

js，ts，jsx，tsx均可以通过babel转换，添加pollyfill（core-js）

babel编译原理，通过babel-loader处理js文件，可通过加载预设配置预设好的插件

1. 词法分析解析字符并标记，语法分析生成AST
2. 遍历AST，对节点应用插件
3. 生成新AST
4. 生成新代码

#### 4. vue文件

vue模板依赖vue的编译器解析生成js代码，之后通过vue-loader加载

#### 5. 常见插件

- **CleanWebpackPlugin**每次打包清除之前的打包结果
- **HtmlWebpackPlugin**创建一个index.html，可以自定义EJS模板
- **DefinePlugin**配置编译全局常量
- **MiniCssExtractPlugin**将CSS代码提取成单独的文件
- **CSSMinimizerPlugin**压缩CSS代码
- **TerserPlugin**压缩代码
- **BundleAnalyzerPlugin**可视化分析代码打包体积

#### 6. 开发服务器devServer

- **HMR热模块替换**，开发模式下，webpack把打包结果存储在内存中，代码发生变化时，只替换变化的文件

- **proxy**：配置前端代理，解决跨域

- **historyApiFallback**：配置前端history模式路由刷新页面404的问题，与nginx配置类似，匹配不到路径时尝试返回index.html，然后交由前端路由控制

#### 7. source map作用

可以指定让webpack在打包时生成源文件映射，方便调试定位的源文件的代码处

1. 生成map文件
2. 在打包的文件加入map文件注释
3. 浏览器读取注释，映射到源文件

最佳实践：开发测试阶段使用source-map，生产环境不使用，减少代码传输体积

## vite和webpack的区别

- vite基于原生ESM，由浏览器来加载模块
- vite开发时不打包，按需加载，由esbuild编译成浏览器支持的代码，esbuild使用Go语言编写，构建速度很快，在浏览器请求源代码文件进行转发，获取编译好的文件执行，因此启动速度很快
- vite开发构建使用esbuild，打包使用rollup
- vite使用http缓存来优化加载速度，依赖的源码文件（不会经常发生更改）采用强缓存，项目的源码文件采用协商缓存
- webpack基于内存打包，将整个项目打包后存储在内存，随着项目的增长，启动速度很慢，但webpack生态较好，支持很多优化插件

## Loader和Plugin的区别

