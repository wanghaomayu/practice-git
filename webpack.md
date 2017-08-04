#  **什么是webpack？**
1. 相信所有接触过vue或react的朋友们都接触过脚手架，可能我们能想到脚手架？顾名思义呗，联想中国第一艘航母在大连造船厂被脚手架快速搭建，也就是说建造新产品的机器，没错！**webpack**就是为你的项目款速搭建的一款模块加载器兼打包工具，它能把各种资源，例如JS（含JSX）、coffee、样式（含less/sass）、图片等都作为模块来使用和处理。



2. 我们可以直接使用 require(XXX) 的形式来引入各模块，即使它们可能需要经过编译（比如JSX和sass），但我们无须在上面花费太多心思，因为 webpack 有着各种健全的加载器（loader）在默默处理这些事情

   ![](http://wiki.hrsoft.net/uploads/201707/5959157433c41_59591574.png)
   
   比如webpack模块就在*../node_modules/.bin/webpack.js* 地址
   



3. 你可以不打算将其用在你的项目上，但没有理由不去掌握它，因为以近期 Github 上各大主流的（React相关）项目来说，它们仓库上所展示的示例已经是基于 webpack 来开发的，比如 React-Boostrap 和 Redux。webpack的官网是 http://webpack.github.io/ ，文档地址是 http://webpack.github.io/docs/ ，想对其进行更详细了解的可以点进去瞧一瞧。

------------


## **webpack 的优势**
其优势主要可以归类为如下几个：
1. webpack 是以 commonJS 的形式来书写脚本滴，但对 AMD/CMD 的支持也很全面，方便旧项目进行代码迁移。
2. 能被模块化的不仅仅是 JS 了。
3. 开发便捷，能替代部分 grunt/gulp 的工作，比如打包、压缩混淆、图片转base64等。
4. 扩展性强，插件机制完善，特别是支持 React 热插拔（见 [react-hot-loader](https://github.com/gaearon/react-hot-loader "react-hot-loader") ）的功能让人眼前一亮。

------------

我们谈谈第一点。以 AMD/CMD 模式来说，鉴于模块是异步加载的，所以我们常规需要使用 define 函数来帮我们搞回调：
```javascript
define(['package/lib'], function(lib){
    function foo(){
        lib.log('hello world!');
    } 
    return {
        foo: foo
    };
});
```
另外为了可以兼容 commonJS 的写法，我们也可以将 define 这么写：
```javascript
define(function (require, exports, module){
    var someModule = require("someModule");
    var anotherModule = require("anotherModule");    
 
    someModule.doTehAwesome();
    anotherModule.doMoarAwesome();
 
    exports.asplode = function (){
        someModule.doTehAwesome();
        anotherModule.doMoarAwesome();
    };
});
```
然而对 webpack 来说，我们可以直接在上面书写 commonJS 形式的语法，无须任何 define （毕竟最终模块都打包在一起，webpack 也会最终自动加上自己的加载器）：
```javascript
var someModule = require("someModule");
    var anotherModule = require("anotherModule");    
 
    someModule.doTehAwesome();
    anotherModule.doMoarAwesome();
 
    exports.asplode = function (){
        someModule.doTehAwesome();
        anotherModule.doMoarAwesome();
 };
```
这样的代码看起来简单又自然，可以和各种回调说拜拜啦

不过即使你保留了之前 define 的写法也是可以滴，毕竟 webpack 的兼容性相当出色，方便你旧项目的模块直接迁移过来。

------------



# **安装和配置**
### **一、安装**
我们常规直接使用 npm 的形式来安装：
```bash
$ npm install webpack -g
```
当然如果常规项目还是把依赖写入 package.json 包去更人性化：
```bash
$ npm init
$ npm install webpack --save-dev
```
### **二、配置**
每个项目下都必须配置有一个 webpack.config.js ，它的作用如同常规的 gulpfile.js/Gruntfile.js ，就是一个配置项，告诉 webpack 它需要做什么。

看一下一下代码：
```javascript
var webpack = require('webpack');
var commonsPlugin = new webpack.optimize.CommonsChunkPlugin('common.js');
 
module.exports = {
    //插件项
    plugins: [commonsPlugin],
    //页面入口文件配置
    entry: {
        index : './src/js/page/index.js'
    },
    //入口文件输出配置
    output: {
        path: 'dist/js/page',
        filename: '[name].js'
    },
    module: {
        //加载器配置
        loaders: [
            { test: /\.css$/, loader: 'style-loader!css-loader' },
            { test: /\.js$/, loader: 'jsx-loader?harmony' },
            { test: /\.scss$/, loader: 'style!css!sass?sourceMap'},
            { test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'}
        ]
    },
    //其它解决方案配置
    resolve: {
        root: 'E:/github/flux-example/src', //绝对路径
        extensions: ['', '.js', '.json', '.scss'],
        alias: {
            AppStore : 'js/stores/AppStores.js',
            ActionType : 'js/actions/ActionType.js',
            AppAction : 'js/actions/AppAction.js'
        }
    }
};
```

⑴ plugins 是插件项，这里我们使用了一个 CommonsChunkPlugin 的插件，它用于提取多个入口文件的公共脚本部分，然后生成一个 common.js 来方便多页面之间进行复用。
⑵ entry 是页面入口文件配置，output 是对应输出项配置（即入口文件最终要生成什么名字的文件、存放到哪里），其语法大致为：

```javascript
{
    entry: {
        page1: "./page1",
        //支持数组形式，将加载数组中的所有模块，但以最后一个模块作为输出
        page2: ["./entry1", "./entry2"]
    },
    output: {
        path: "dist/js/page",
        filename: "[name].bundle.js"
    }
}
```
该段代码最终会生成一个 page1.bundle.js 和 page2.bundle.js，并存放到 ./dist/js/page 文件夹下。bundle.js文件是打包文件，是浏览器可以识别的文件，我们平时模块化开发，将代码写成各组件浏览器是不认识的，所以
```javascript
module: {

        //加载器配置
    rules: [
	 //.jsx文件使用 'react-hot-loader和babel-loader 来编译处理（热替代和js转换编译模块）
      {
        test: /\.jsx?$/,
        loaders: 'react-hot-loader!babel-loader',
        exclude: /node_modules/,
      },
	  //.css 文件使用 style-loader 和 css-loader和postcss-loader 来处理
      {
        test: /\.css$/,
        loaders: ['style-loader', 'css-loader', 'postcss-loader']
      },
	 //.less文件用'style-loader和css-loader和postcss-loader和less-loader处理
      {
        test: /\.less/,
        loaders: ['style-loader', 'css-loader', 'postcss-loader', 'less-loader']
      },
	  //图片用url-loader处理
      {
        test: /\.(png|jpg|gif|woff|woff2|svg)$/,
        loaders: [
          'url-loader?limit=10000&name=[hash:8].[name].[ext]',
        ]
      }
    ]
  }
```
如上，多个loader之间用“!”连接起来。
注意所有的加载器都需要通过 npm 来加载，并建议查阅它们对应的 readme 来看看如何使用。
拿最后一个 [url-loader](https://github.com/webpack/url-loader "url-loader") 来说，它会将样式中引用到的图片转为模块来处理，使用该加载器需要先进行安装：
```bash
npm install url-loader -save-dev
```
配置信息的参数“?limit=1000”表示将所有小于10kb的图片都转为base64形式（其实应该说超过10kb的才使用 url-loader 来映射到文件，否则转为data url形式）。

⑷ 最后是 resolve 配置，这块很好理解，直接写注释了：
```javascript
resolve: {
        //查找module的话从这里开始查找
        root: 'E:/github/flux-example/src', //绝对路径
        //自动扩展文件后缀名，意味着我们require模块可以省略不写后缀名
        extensions: ['', '.js', '.json', '.scss'],
        //模块别名定义，方便后续直接引用别名，无须多写长长的地址
        alias: {
            AppStore : 'js/stores/AppStores.js',//后续直接 require('AppStore') 即可
            ActionType : 'js/actions/ActionType.js',
            AppAction : 'js/actions/AppAction.js'
        }
    }
```
关于 webpack.config.js 更详尽的配置可以参考[这里](http://webpack.github.io/docs/configuration.html "这里")。

------------

## **运行**

webpack 的执行也很简单，直接执行
```bash
$ webpack --display-error-details
```
即可，后面的参数“--display-error-details”是推荐加上的，方便出错时能查阅更详尽的信息（比如 webpack 寻找模块的过程），从而更好定位到问题。
其他主要的参数有：
```bash
$ webpack --config XXX.js   //使用另一份配置文件（比如webpack.config2.js）来打包
 
$ webpack --watch   //监听变动并自动打包
 
$ webpack -p    //压缩混淆脚本，这个非常非常重要！
 
$ webpack -d    //生成map映射文件，告知哪些模块被最终打包到哪里了
```
其中的 -p 是很重要的参数，曾经一个未压缩的 700kb 的文件，压缩后直接降到 180kb（主要是样式这块一句就独占一行脚本，导致未压缩脚本变得很大）。

------------

# **模块引入**
上面唠了那么多配置和执行方法，下面开始说说寻常页面和脚本怎么使用吧。
###** 一、HTML**
直接在页面引入 webpack 最终生成的页面脚本即可，不用再写什么 data-main 或 seajs.use 了：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>React</title>
    <meta charset="utf-8" />
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <link rel="shortcut icon" href="./favicon.ico">
</head>
<body>
<div id="app">
</div>
<script src="/assets/bundle.js"></script>
<script src="https://cdn.bootcss.com/fetch/2.0.3/fetch.min.js"></script>
</body>
</html>
```
可以看到我们连样式都不用引入，毕竟脚本执行时会动态生成<style>并标签打到head里。
### **二、JS**
各脚本模块可以直接使用 commonJS 来书写，并可以直接引入未经编译的模块，比如 JSX、sass、coffee等（只要你在 webpack.config.js 里配置好了对应的加载器）。
我们再看看编译前的页面入口文件（index.js）：
```javascript
require('../../css/reset.scss'); //加载初始化样式
require('../../css/allComponent.scss'); //加载组件样式
var React = require('react');
var AppWrap = require('../component/AppWrap'); //加载组件
var createRedux = require('redux').createRedux;
var Provider = require('redux/react').Provider;
var stores = require('AppStore');
 
var redux = createRedux(stores);
 
var App = React.createClass({
    render: function() {
        return (
            <Provider redux={redux}>
                {function() { return <AppWrap />; }}
            </Provider>
        );
    }
});
 
React.render(
    <App />, document.body
);
```
注意：上述写法是在webpack.config.js中没有require，否则直接在index.js中引入就可以了，比如：
```javascript
import React from 'react'
import ReactDOM from 'react-dom'
import CommentApp from './components/CommentApp.js'
import './index.css'

ReactDOM.render(
    <CommentApp />,
    document.getElementById('root')
)

```
一切都是so  easy~ 后续各种有的没的，webpack 都会帮你进行处理

------------
# **总结**

至此我们已经基本上手了 webpack 的使用，大致理解了webpack是什么，做什么的。是的，他就是依赖各种包（工具）在webpack.config.js中引入（登记），各种依赖就可以助webpack正常构建你写的代码了（脚手架工作），这时候你就可以安心的造航母了！

