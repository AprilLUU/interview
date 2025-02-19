## Webpack

入口 entry

出口 output

#### 加载CSS

rules配置loader

test 匹配文件

use 使用loader 

postcss-preset-env

less-loader postcss-loader css-loader style-loader

加载其他资源

url-loader（对小于limit指定大小的文件进行base64编码，减少请求次数） file-loader

asset url-loader limit

asset/resource file-loader

asset/inline url-loader

asset/source 导出资源源代码 raw-loader

#### plugin

clean-webpack-plugin 每次打包之前清空文件夹

html-webpack-plugin 生成html文件，根据ejs模板进行编译，可以指定模板，注入ejs变量

copy-webpack-plugin  对指定文件夹内容进行复制 plublic文件夹

resolve 加载文件时如何加载 扩展名 别名

#### vite

原生ESM 请求转发 直接多种文件的模块化开发 只需安装对应的编译器即可无需配置。

webpack过于灵活，只针对打包不预设场景，所有环节几乎都是可配置的

与loader plugin耦合度太高，一旦走vite的非打包无异于重启一个项目。