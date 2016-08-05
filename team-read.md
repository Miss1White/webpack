#WebPack

##1 什么是webpack?

-  一个打包工具
-  一个模块加载工具
-  各种资源都可以当成模块来处理

##2.webpack用来干什么

-  将依赖树拆分，保证按需加载
-  保证初始加载的速度
-  所有静态资源可以被模块化
-  可以整合第三方的库和模块
-  可以构造大系统

##3.特点

-  丰富的插件，方便进行开发工作
-  大量的加载器，包括加载各种静态资源
-  代码分割，提供按需加载的能力
-  发布工具

##4优势

-  webpack 是以 commonJS 的形式来书写脚本滴，但对 AMD/CMD 的支持也很全面，方便旧项目进行代码迁移。
-  能被模块化的不仅仅是 JS 了。
-  开发便捷，能替代部分 grunt/gulp 的工作，比如打包、压缩混淆、图片转base64等。
-  扩展性强，插件机制完善，特别是支持 React 热插拔（见 react-hot-loader ）的功能让人眼前一亮。

##5.安装
```
//安装命令
npm install -g webpack

//使用webpack
$ npm init  # 会自动生成一个package.json文件
$ npm install webpack --save-dev #将webpack增加到package.json文件中
```

##6.配置
每个项目下都必须配置有一个```webpack.config.js```,它的作用如同常规的```gulpfile.js/Gruntfile.js```,就是一个配置项，告诉webpack它需要做什么

###An example
```
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
-plugins 是插件项，这里我们使用了一个 CommonsChunkPlugin的插件，它用于提取多个入口文件的公共脚本部分，然后生成一个```common.js``` 来方便多页面之间进行复用。
-entry 是页面入口文件配置，output 是对应输出项配置 （即入口文件最终要生成什么名字的文件、存放到哪里）
-module.loaders 是最关键的一块配置。它告知 webpack 每一种文件都需要使用什么加载器来处理。 所有加载器需要使用npm来加载
-最后是 resolve 配置，配置查找模块的路径和扩展名和别名（方便书写）

## 7.使用

1.新建```entry.js```   和  ``` mymodule.js```
```
//main.js   入口文件
require('./mymodule.js')();

//mymodule.js
module.exports = function() {
    document.write('hello webpack');
};
```
2.index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
    <script src="./app.js"></script>
</body>
</html>
```
3.执行,可在浏览器中看到预期效果```app.js``` 为生成的打包文件
```
$ webpack ./main.js app.js
```

4.配置文件.每次手动输入源文件名和输出文件名比较麻烦，可以使用配置文件来进行管理。在app目录下新建```webpack.config.js```文件，内容如下：
```
module.exports = {
    entry: './main.js',
    output: {
        filename: 'app.js'
    }
};

//然后执行
webpack
```
就会自动生成打包好的文件了。但是这样每次改了源文件之后都需要手动执行命令，可以通过添加watch来自动检测文件变化并重新打包。配置文件修改如下：
```
module.exports = {
    entry: './main.js',
    output: {
        filename: 'app.js'
    },
    watch: true
};
```

**进行加载器试验 loader **

很多模块打包工具只是针对js文件，而webpack的强大之处在于将模块的概念进行了扩展，认为一切静态文件都是模块，包括css、html模板、字体、CoffeeScript等等。虽然webpack本身依然是只能够处理js文件，但是通过一系列的loader，就可以处理其它文件了。

下面以```css-loader```和```style-loader```为例，演示如何打包样式文件。

1.执行命令安装依赖模块：
```
npm install css-loader style-loader --save-dev      # 安装的时候不使用 -g
```

2.新建```style.css```
```
body {
background: yellow;
}
```
3.修改```main.js```
```
require('./mymodule.js')();
require('style!css!./style.css');
```
因为webpack不能够直接处理css文件，因此在require语句中需要指明需要的loader，一个文件可以经由多个loader依次处理，
loader与loader之间，以及loader与文件名之间用!分隔。在这个例子中，也可以看出，如果使用了多个loader的话，数据流向是从右向左的，
也就是从```style.css```开始，依次经过```css-loader```和```style-loader```。

但是假如有多个css文件的话，每个require语句都需要加上loader说明，很不方便，因此可以在webpack.config.js文件中进行配置，配置如下:
```
loaders: [{
    test: /\.css$/,
    loader: 'style!css'
}]

// or

loaders: [{
    test: /\.css$/,
    loaders: ['style', 'css']
}]
```
[更多关于loader](http://webpack.github.io/docs/using-loaders.html)


**外部依赖**

现在假如该例子中需要用到angular，首先在```index.html```中通过<script>标签引入angular库，然后修改```mymodule.js```如下：

```
var angular = require('angular');
angular.module('MyModule', []);
```
此时如果执行webpack命令会报如下错误：
```
ERROR in ./mymodule.js
Module not found: Error: Cannot resolve module 'angular' in /xxx/xxx/app
 @ ./mymodule.js 1:14-32
 ```
 这是因为webpack无法解析angular依赖模块，此时需要在配置文件中对外部依赖进行配置：
 ```
 externals: {
    'angular': true
}
```
[More>>](http://webpack.github.io/docs/configuration.html#externals)

**输出类型**

现在假如我们希望打包后的文件作为一个单独的库，并且遵循AMD规范可以被被requirejs来使用，可以修改配置文件如下：
```
output: {
    filename: 'app.js',
    library: 'app',
    libraryTarget: 'amd'
}
```
此时输出的app.js结构如下
```
define("app", ["angular"], function( /* ... */ ) {
    /* ... */
});
```
通过配置output.libraryTarget，可以自定义输出的模块类型，包括AMD，CommonJS，变量等多种输出类型。具体可以参考[output](http://webpack.github.io/docs/configuration.html#output)

**多文件**  [Multiple entry points](http://webpack.github.io/docs/multiple-entry-points.html)    [entry](http://webpack.github.io/docs/multiple-entry-points.html)
现在假如项目目录结构如下:
```
/app
  |--components.js
  |--index.html
  |--main.js
  |--mymodule.js
```

其中```mymodule.js```被```main.js```和```components.js```所使用。假如我们希望```main.js输出为app.js```，
而```components```输出为```app.components.js```，则可以修改配置文件如下:
```
entry: {
    app: './main.js',
    'app.coomponents': './components.js'
},
output: {
    filename: '[name].js'
}
```
