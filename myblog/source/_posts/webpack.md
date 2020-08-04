### webpack作为前端构建工具的用途

- 代码转换：ts => js    sass/less => css
- 文件优化：压缩js/css/html  合并
- 代码分割：公共代码提取
- 模块合并
- 自动刷新
- 代码校验：配置单元测试
- 自动发布



**webpack初始化**

```javascript
新建文件夹 webpack-test
cd webpack-test
npm init -y   //初始化 => 生成package.json文件
npm install webpack webpack-cli -D  
```



**关于commonJS和esModule**

commonJS:  require   module.exports

esModule:   import  export

```javascript
export {sum};   //引用只能用sum
export default sum;   //引入可以随意起名
```



打包命令：npm 版本5之后执行下面命令打包

```
npx webpack
```



**配置webpack.config.js文件**

规范：node   commonJS规范

**单入口配置**

```javascript
const path = require('path');
module.exports = {
    entry:'./src/index.js',   //单个入口文件
    output:{
        //[hash、chunkHash、contentHash]
        filename: 'bundle.[contentHash:8].js',   //生成的文件名,加了8位的hash值 
        path: path.resolve(__dirname, "dist")    // => 需要引入path,该配置表明生成文件夹dist
    }
}
```

**多入口多出口文件的配置**

```javascript
const path = require('path');
module.exports = {
    entry:{
        index:'./src/index.js',
        other:'./src/other.js'
    },
    output:{
        filename:'[name].js',
        path: path.resolve(__dirname, "dist")
    }
}
```



**插件安装**

安装在开发环境：npm install  xxx  -D 

安装在生产环境：npm  install xxx

```
//把打包后的文件自动引入指定html文件中
npm install html-webpack-plugin --D

//删除打包文件
npm install clean-webpack-plugin --D

//自动化
npm install webpack-dev-server --D  //创建本地服务器，自动重新构建，自动打开浏览器并刷新
```



html-webpack-plugin的相关配置：

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    ...
    plugins:[
        new HtmlWebpackPlugin({
            template: './index.html',   //依赖的模板文件
            hash: true,   //是否加hash值
            filename: 'index.html',  //打包后的html文件名
            chunks: ['index'],
            minify: {
                removeRedundantAttributes: true, //删除引号
                collapseWhitespacee: true,  //删除空格
            }
    	})
    ]
}

多个文件的处理：
let htmlPlugin = ['index', 'other'].map((chunkName) => {
  return new HtmlWebpackPlugin({
    template: `./${chunkName}.html`,
    filename: `${chunkName}.html`,
    chunks: [chunkName]
  })
})

module.exports = {
    ...
    plugins:[
        ...htmlPlugin
    ]
}

```

clean-webpack-plugin的相关配置：

```javascript
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
    ...
    plugins:[
        new CleanWebpackPlugin(),   //清空输出的目录  ---清空dist下所有文件
        //清空指定文件
        new CleanWebpackPlugin({
    		cleanOnceBeforeBuildPatterns: ['test/*', '!test/a.js']  //清空test文件夹下不是a.js的文件
    	})
    ]
}
```

dev-server配置：

```javascript
devServer: {   //在内存中打包，所有内容是在根目录
    port: 8181,
    open: true,
    compress: true,
    contentBase: 'static',   //启动static下的静态资源文件
    hot: true,  //自动刷新
    before: function (app, server) {    //app表示启动一个端口为8181的服务
      app.get('/api/user', function (req, res) {
        res.json({ custom: 'reponse' })
      })
    }
    // proxy: {  //启动代理
    //   '/api': {
    //     target: 'http://localhost:8000',
    //     secure: false,  //若为ture代表以https开头
    //     pathRewrite: {
    //       '^/api': ''
    //     },  //重写地址
    //     changeOrigin: true     //把请求头当中的host改成请求的服务器地址
    //   }
    // }
}
```



**解析css文件等**

```javascript
解析css:   css-loader     style-loader
解析less:  less   less-loader
解析sass:  node-sass   sass-loader
解析stylus:  stylus   stylus-loader
```

```javascript
module: {
    rules: [
      //从下往上运行
      // {
      //   test: /\.css$/,
      //   use: 'css-loader'
      // },
      // {
      //   test: /\.css$/,
      //   use: 'style-loader',
      //   enforce: 'post'   //pre-优先加载  post-最后加载
      // },

      //从右往左运行
      //css
      {
        test: /\.css$/,
        use:['style-loader','css-loader'],
      },
      //less
      {
        test: /\.less$/,
        use: ['style-loader', 'css-loader', 'less-loader']
      }
    ]
  },
```



**处理css私有前缀**

插件：postcss-loader 【样式处理工具】         autoprefixer【私有前缀的插件】

配置postcss.config.js

```javascript

```

配置.browserslistrc文件：cover 99.5%



**分离css**

插件：mini-css-extract-plugin

```javascript
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

module.exports = {
    ...
    module:{
        rules:[
            {
    			test: /\.css$/,
    			use: [
                    {
                          loader: MiniCssExtractPlugin.loader,
                    },
                    {
                          loader: 'css-loader',
                          options: {
                               importLoaders: 1, //用后面的1个加载器来解析
                          }
                    }, 'postcss-loader', 'less-loader']
			},
        ]
    },
    plugins:[
        new MiniCssExtractPlugin({
          filename: 'css/main.css'   //设置分离出的css所存放的目录及文件名
        })
    ]
}
```



**压缩css文件和js文件**

压缩css：npm install optimize-css-assets-webpack-plugin【生产环境】

压缩js：npm install terser-webpack-plugin 【生产环境】

```javascript
const OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin')
const TerserJSPlugin = require('terser-webpack-plugin')

module.exports = {
  optimization: {
    minimizer: [   //压缩的css/js
      new OptimizeCssAssetsPlugin(),
      new TerserJSPlugin()
    ]
  },
  ...
}
```



**处理图片**

插件：file-loader  url-loader  

file-loader把图片解析成文件输出，url-loader把小图片解析成base64插入；

```javascript
//file-loader
{
	test: /\.(jpg|jpeg|gif)$/,
	//use: 'file-loader'  //名字改变
    use:{
          loader:'file-loader',
          options:{
            name:'img/[name].[ext]'   //img文件夹下 命名一致
          }
    }
}

//url-loader
{
      test: /\.(jpg|jpeg|gif)$/,
      use: {
        loader: 'url-loader',  //小于100kb作为base64输出，若大于100kb会自动调用file-loader,打包成文件输出
        options: {
           limit: 100 * 1024,  //100kb
           outputPath: 'img'   //输出路径
           //publicPath: 'http://www.baidu.com'   //服务器地址
        }
     }
}
```

小icon配置：

```javascript
{
    test:/\.(eot|svg|ttf|woff|woff2)$/,
    use:'file-loader'
}
```



**babel**

@babel/core   babel的核心模块

babel-loader 解析js代码，webpack和babel的桥梁

@babel/preset-env    es6转化成es5插件的集合

```javascript
npm install @babel/core babel-loader @babel/preset-env -D


{
      test: /\.js$/,
      use: 'babel-loader',
      include: path.resolve(__dirname, 'src'),   //需要编译的js文件的目录
      exclude: /node_modules/   //排除需要编译js文件的目录
}
```

npm install core-js@3  --save

文件.babelrc配置：

```javascript
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage",          //只转化使用的api
        "corejs":3        //@babel/pollyfill    转化高版本的api
      }
    ]
  ], //预设（插件集合）   从下往上
  "plugins": [] //一个个插件    从上往下
}
```



装饰器：语法糖的写法



**跨域**

插件：

```
npm install express -D
```

配置server.js模拟后台接口：

```javascript
let express = require('express')
let app = express();

app.get('/user', function (req, res) {
  res.json({
    name: 'tt'
  })
})

app.listen(8000, function () {
  console.log('8000这个端口的服务已启动！')
})
```

此时，到页面中输入localhost:8000/user就可以拿到{name:tt}

在webpack.config.js中配置代理：

```javascript
devServer: {  
    port: 8181,
    open: true,
    compress: true,
    contentBase: 'static',   //启动static下的静态资源文件
    hot: true,  //自动刷新
    proxy: {  //启动代理
      '/api': {
        target: 'http://localhost:8000',
        secure: false,  //若为ture代表以https开头
        pathRewrite: {
          '^/api': ''
        },  //重写地址
        changeOrigin: true     //把请求头当中的host改成请求的服务器地址
        //原本host:localhost:8181   设置该属性后为localhost:8000
      }
    }
}
```

也可以使用before，这样就不需要服务自己去造数据做一些处理。

```javascript
devServer: {   
    port: 8181,
    ...
    hot: true,  //自动刷新
    //在服务之前执行
    before: function (app, server) {    //app表示启动一个端口为8181的服务
      app.get('/api/user', function (req, res) {
        res.json({ custom: 'reponse' })
      })
    }
}
```





**暴露全局变量**

- 直接使用cdn方式；
- providePlugin——给每个模块注入变量；
- 暴露在全局下expose-loader；

1、add-asset-html-cdn-webpack-plugin

```javascript
const AddAssetHtmlCdnWebpackPlugin  = require('add-asset-html-cdn-webpack-plugin')

modules.export = {
    externals:{
        'jquery':'$'   //$外部变量，不需要打包
    },
    plugins:[
        new AddAssetHtmlCdnWebpackPlugin(true,{
            'jquery':'https://code.jquery.com/jquery-3.4.1.min.js'
        })
    ]
}
```

2、providePlugin

```javascript
npm install lodash -D

import _ from 'lodash'

//配置
const webpack = require('webpack');
plugins:[
    new Webpack.ProvidePlugin({
        "$":'jquery',   //$来自于jquery，每个模块都注入变量$，但不是注入在全局下
        "_map":['lodash','map']
    })
]
```

3、expose-loader

插件：

```
npm  install  expose-loader  -D
```

```javascript
//配置
rules:[
    {
        test:require.resolve('jquery'),
        options:'$'
    }
]

//使用
import $ from 'jquery'
console.log($,window.$)
//------
require('expose-loader?$!jquery')        //jquery放在全局的$
```



**eslint.js**

引入.eslintrc.json文件

插件：eslint  eslint-loader

```javascript
rules:[
    {
        test:/\.json$/,
        use:'eslint-loader',
        enforce:'pre'  //优先级最高
    }
]
```

npx eslint  --init  => 生成配置文件



**sourceMap（代码排查）**

开发环境：cheap-module-eval-source-map

生产环境：cheap-module-source-map

```javascript
module.exports = {
    devtool:'cheap-module-eval-source-map'   //根据环境切换
}
```

可以定位到报错具体原因和行数