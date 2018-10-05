# webpack 入门/简单配置

这篇只是很个人向的博客记录，只是记录了学习 webpack 中的简单配置，关于 webpack 的各种参数 和基础概念都语焉不详，拿来就用。

什么是 webpack：

>本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(static module bundler)。在 webpack 处理应用程序时，它会在内部创建一个依赖图(dependency graph)，用于映射到项目需要的每个模块，然后将所有这些依赖生成到一个或多个bundle。

为什么使用 webpack：

使用 webpack 可以让代码维护、转换更便利，提高开发效率。

关于学习 webpack 更详细的资料可以参考 webpack [文档](https://webpack.js.org/) 或 [中文文档](https://webpack.docschina.org/concepts/)

---

## 安装 node

首先要安装 Node.js， Node.js 自带了软件包管理器 npm

[安装过程](http://www.runoob.com/nodejs/nodejs-install-setup.html)省略~

一般没有什么问题都是一次过，有则 google。

安装完成后在项目根目录下运行 `npm init -y` ，得到一个 package.json，这个文件定义了这个项目所需要的各种模块，以及项目的配置信息（比如名称、版本、许可证等元数据）

---

## 安装 webpack

在项目目录下运行 `npm install webpack --save-dev`

**注意版本号** 可以在安装时使用 安装包@版本号的格式 安装具体版本，如 `npm install webpack@3 --save-dev`；不设置则默认安装最新版本。

在项目根目录新建一个 webpack.config.js 文件

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/css/main.scss',     //入口指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始
  output: {
    filename: 'main.css',  //文件命名
    path: path.resolve(__dirname, 'dist/css/')   //出口告诉 webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件
  }
};
```

---

## 处理 scss

我们先写一小段 scss，以便验证 sass-loader 是否能正常运行：

```scss
// src/css/main.scss
$color-white: #fff;
$color-pink: #ee11ab;
$title-font: normal 24px/1.5 'Open Sans', sans-serif;

$primary-color: $color-pink;

body {
  height: 100vh;
  background-color: $color-pink;
  color: $color-white;
  display: flex;
  justify-content: center;
  align-items: center;
}
```

现在直接按照官方 demo 指示，运行成功再慢慢修改：

使用 [sass-loader](https://github.com/webpack-contrib/sass-loader)

安装 loader `npm install sass-loader node-sass webpack --save-dev`

在 webpack.config.js 中添加相应的规则:

```javascript
// webpack.config.js
module.exports = {
    ...
    module: {
        rules: [{
            test: /\.scss$/,
            use: [
                "style-loader", // creates style nodes from JS strings
                "css-loader", // translates CSS into CommonJS
                "sass-loader" // compiles Sass to CSS, using Node Sass by default
            ]
        }]
    }
};
```

运行 `npx webpack`

如果成功的话，页面按照 CSS 来改变他的样式。

他就相当于把 sacc 转换成了 css，再把 css 转换成了 js，最后再由 js 转换成了标签内 style，然后再把页面渲染出来。

不过肯定不会一次成功的，根据报错信息处理即可~(如提示node-modules缺失某些包，则用`npm install XXX`安装缺失的包即可)

---

## 处理 js

我们还可以使用 babel 来处理 js 代码，把更高级的语法转换成 ES5 语法。

先写个 ES6 的语法：

```javascript
// src/js/main.js

let a = 1
console.log(a)
```

[babel-loader](https://github.com/babel/babel-loader)官方文档

按照官方文档：

运行 `npm install -D babel-loader @babel/core @babel/preset-env`

添加代码:

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/js/main.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist/js/')
  },
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: [
            "style-loader", // creates style nodes from JS strings
            { loader: 'css-loader', options: { importLoaders: 1 } },
            'postcss-loader',
            // "css-loader", // translates CSS into CommonJS
            "sass-loader" // compiles Sass to CSS, using Node Sass by default
        ]
      }
    ]
  }
};
```

运行 `npx webpack`，不幸成功的话，在 `dist/js/` 目录下，会出现一个 bundle.js ，原文件里的 let 已经被改成了 var，实现了语法的转换。

---

## 说说模块化

这是我的粗浅理解：利用新语法 `import/export` 可以实现每个文件的互相调用（模块），而在 webpack 中，可以设置一个「入口文件」，这个「入口文件」包含了每个模块的调用，而且这个「入口文件」经过了 webpack 的处理，可以转化为另一个包含了所有被调用模块的「出口文件」

应该还是 demo 好理解一点：

```javascript
// src/js/module-1.js
function fn(){
    console.log(1)
}
export default fn
```

```javascript
// src/js/module-2.js
function fn(){
    console.log(2)
}
export default fn
```

```javascript
// src/js/modules.js
import module_1 from './module-1'
import module_2 from './module-2'

module_1()

console.log('this\'s modules')

module_2()
```

我们分别设置了三个文件，其中两个文件通过 `export default fn` 输出了两个函数，而总文件 通过 `import`引入了这两个文件并且调用了他们，这就是 webpack 中的模块化，相当于把三个文件都汇集输出成了一个文件，及最后的 bundle.js。

现在我们再加上原来的 main.scss 文件：

```javascript
// src/js/modules.js
import module_1 from './module-1'
import module_2 from './module-2'
import '../css/main.scss'

module_1()

console.log('this\'s modules')

module_2()
```

webpack.config.js

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/js/modules.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist/js/')
  },
  module: {
    rules: [
      {
        test: /\.m?js$/,
        exclude: /(node_modules|bower_components)/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      },
      {
        test: /\.scss$/,
        use: [
            "style-loader", // creates style nodes from JS strings
            { loader: 'css-loader', options: { importLoaders: 1 } },
            'postcss-loader',
            // "css-loader", // translates CSS into CommonJS
            "sass-loader" // compiles Sass to CSS, using Node Sass by default
        ]
      }
    ]
  } 
};
```

运行`npx webpack`，成功之后生成了 bundle.js，它里面包含了三个 JS 文件加上 Scss 的内容：

```javascript
...
(0, _module.default)();
console.log('this\'s modules');
(0, _module2.default)();
...
exports.push([module.i, "body{height:100vh;background-color:#ee11ab;color:#fff;display:flex;justify-content:center;align-items:center;text-align:center}", ""]);
...
```

页面样式也可以正常显示。