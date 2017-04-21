# 0. 服务端渲染简介

服务端渲染不是一个新的技术；在 Web 最初的时候，页面就是通过服务端渲染来返回的，用 PHP 来说，通常是使用 Smarty 等模板写模板文件，然后 PHP 服务端框架将数据和模板渲染为页面返回，这样的服务端渲染有个缺点就是一旦要查看新的页面，就需要请求服务端，刷新页面。

但如今的前端，为了追求一些体验上的优化，通常整个渲染在浏览器端使用 JS 来完成，配合 history.pushState 等方式来做单页应用（SPA: Single-Page Application），也收到不错的效果，但是这样还是有一些缺点：第一次加载过慢，用户需要等待较长时间来等待浏览器端渲染完成；对搜索引擎爬虫等不友好。这时候就出现了类似于 React，Vue 2.0 等前端框架来做服务端渲染。

使用这些框架来做服务端渲染的兼顾了上面的几个优点，而且写一份代码就可以跑在服务端和浏览器端。Vue 2.0 发布了也有一段时间了，新版本比较大的更新就是支持服务端渲染，最近有空折腾了下 Vue 的服务端渲染，记录下来。

# 1. 在 Vue 2.0 中使用服务端渲染

官方文档给了一个简单的例子来做服务端渲染：
```javascript
// 步骤 1:创建一个Vue实例
var Vue = require('vue')
var app = new Vue({
  render: function (h) {
    return h('p', 'hello world')
  }
})
// 步骤 2: 创建一个渲染器
var renderer = require('vue-server-renderer').createRenderer()
// 步骤 3: 将 Vue实例 渲染成 HTML
renderer.renderToString(app, function (error, html) {
  if (error) throw error
  console.log(html)
  // => <p server-rendered="true">hello world</p>
})
```
这样子，配合通常的 Node 服务端框架就可以简单来实现服务端渲染了，可是，在真实场景中，我们一般采用 .vue 文件的模块组织方式，这样的话，服务端渲染就需要使用 webpack 来将 Vue 组件进行打包为单个文件。

# 2. 配合 Webpack 渲染 .vue 文件

先建立一个服务端的入口文件 server.js
```javascript
import Vue from 'vue';

import App from './vue/App';

export default function (options) {
    const VueApp = Vue.extend(App);

    const app = new VueApp(Object.assign({}, options));

    return new Promise(resolve => {
        resolve(app);
    });
}
```
这里和浏览器端的入口文件大同小异，只是默认导出了一个函数，这个函数接收一个服务端渲染时服务端传入的一些配置，返回一个包含了 app 实例的 Promise；

简单写一个 App.vue 的文件
```html
<template>
    <h1>{{ title }}</h1>
</template>

<script>
module.exports = {
    props: ['title']
</script>
```
这里将会读取服务端入口文件传入 options 的 data 属性，取到 title 值，渲染到对应 DOM 中；

再看看 Webpack 的配置，和客户端渲染同样是大同小异：
```javascript
const webpack = require('webpack');
const path = require('path');
const projectRoot = __dirname;

const env = process.env.NODE_ENV || 'development';

module.exports = {
    target: 'node', // 告诉 Webpack 是 node 代码的打包
    devtool: null, // 既然是 node 就不用 devtool 了
    entry: {
        app: path.join(projectRoot, 'src/server.js')
    },
    output: Object.assign({}, base.output, {
        path: path.join(projectRoot, 'src'),
        filename: 'bundle.server.js',
        libraryTarget: 'commonjs2' // 和客户端不同
    }),
    plugins: [
        new webpack.DefinePlugin({
            'process.env.NODE_ENV': JSON.stringify(env),
            'process.env.VUE_ENV': '"server"' // 配置 vue 的环境变量，告诉 vue 是服务端渲染，就不会做耗性能的 dom-diff 操作了
        })
    ],
    resolve: {
        extensions: ['', '.js', '.vue'],
        fallback: [path.join(projectRoot, 'node_modules')]
    },
    resolveLoader: {
        root: path.join(projectRoot, 'node_modules')
    },
    module: {
        loaders: [
            {
                test: /\.vue$/,
                loader: 'vue'
            },
            {
                test: /\.js$/,
                loader: 'babel',
                include: projectRoot,
                exclude: /node_modules/
            }
        ]
    }
};
```
其中主要就是三处不同：声明 node 模块打包；修改打包后模块加载方式为 commonjs（commonjs2 具体可以看 Webpack 官方文档）；再就是 vue 的服务端打包优化了，这部分如果不传的话后面 vue 服务端渲染会慢至几十秒，一度以为服务端代码挂了。

最后就是服务端载入生成的 bundle.server.js 文件：

```javascript
const fs = require('fs');
const path = require('path');
const vueServerRenderer = require('vue-server-renderer');
const filePath = path.join(__dirname, 'src/bundle.server.js');

// 读取 bundle 文件，并创建渲染器
const code = fs.readFileSync(filePath, 'utf8');
const bundleRenderer = vueServerRenderer.createBundleRenderer(code);

// 渲染 Vue 应用为一个字符串
bundleRenderer.renderToString(options, (err, html) => {
    if (err) {
        console.error(err);
    }

    content.replace('<div id="app"></div>', html);
});
这里 options 可以传入 vue 组件所需要的 data 等信息；下面还是以官方实例中的 express 来做服务端示例下：

const fs = require('fs');
const path = require('path');
const vueServerRenderer = require('vue-server-renderer');
const filePath = path.join(think.ROOT_PATH, 'view/bundle.server.js');
global.Vue = require('vue')

// 读取 bundle 文件，并创建渲染器
const code = fs.readFileSync(filePath, 'utf8');
const bundleRenderer = vueServerRenderer.createBundleRenderer(code);

// 创建一个Express服务器
var express = require('express');
var server = express();

// 部署静态文件夹为 "assets" 文件夹
server.use('/assets', express.static(
    path.resolve(__dirname, 'assets');
));

// 处理所有的 Get 请求
server.get('*', function (request, response) {
    // 设置一些数据，可以是数据库读取等等
    const options = {
        data: {
            title: 'hello world'
        }
    };

    // 渲染 Vue 应用为一个字符串
    bundleRenderer.renderToString(options, (err, html) => {
        // 如果渲染时发生了错误
        if (err) {
            // 打印错误到控制台
            console.error(err);
            // 告诉客户端错误
            return response.status(500).send('Server Error');
        }

        // 发送布局和HTML文件
        response.send(layout.replace('<div id="app"></div>', html));
    });

// 监听5000端口
server.listen(5000, function (error) {
    if (error) throw error
    console.log('Server is running at localhost:5000')
});
```
这样子基本就是 Vue 服务端渲染的整个流程了，这样子和使用普通的模板渲染并没有什么其他的优势，可是当渲染完成后再由客户端接管渲染后就可以做到无缝切换了，下面我们来看看和客户端结合渲染；

# 3. 和浏览器渲染无缝集合

为了和客户端渲染结合，我们将 webpack 配置文件分为三部分，base 共用的配置，server 配置，client 浏览器端配置，如下：

webpack.base.js
```javascript
const path = require('path');
const projectRoot = path.resolve(__dirname, '../');

module.exports = {
    devtool: '#source-map',
    entry: {
        app: path.join(projectRoot, 'src/client.js')
    },
    output: {
        path: path.join(projectRoot, 'www/static'),
        filename: 'index.js'
    },
    resolve: {
        extensions: ['', '.js', '.vue'],
        fallback: [path.join(projectRoot, 'node_modules')],
        alias: {
            'Common': path.join(projectRoot, 'src/vue/Common'),
            'Components': path.join(projectRoot, 'src/vue/Components')
        }
    },
    resolveLoader: {
        root: path.join(projectRoot, 'node_modules')
    },
    module: {
        loaders: [
            {
                test: /\.vue$/,
                loader: 'vue'
            },
            {
                test: /\.js$/,
                loader: 'babel',
                include: projectRoot,
                exclude: /node_modules/
            }
        ]
    }
};
```
webpack.server.js
```javascript
const webpack = require('webpack');
const base = require('./webpack.base');

const path = require('path');
const projectRoot = path.resolve(__dirname, '../');

const env = process.env.NODE_ENV || 'development';

module.exports = Object.assign({}, base, {
    target: 'node',
    devtool: null,
    entry: {
        app: path.join(projectRoot, 'view/server.js')
    },
    output: Object.assign({}, base.output, {
        path: path.join(projectRoot, 'view'),
        filename: 'bundle.server.js',
        libraryTarget: 'commonjs2'
    }),
    plugins: [
        new webpack.DefinePlugin({
            'process.env.NODE_ENV': JSON.stringify(env),
            'process.env.VUE_ENV': '"server"',
            'isBrowser': false
        })
    ]
});
```
服务端的配置，和之前多了一个 isBrowser 的全局变量，用于在 Vue 模块中做一些差异处理；

webpack.client.js
```javascript
const webpack = require('webpack');
const ExtractTextPlugin = require('extract-text-webpack-plugin');

const base = require('./webpack.base');

const env = process.env.NODE_ENV || 'development';

const config = Object.assign({}, base, {
    plugins: [
        new webpack.DefinePlugin({
            'process.env.NODE_ENV': JSON.stringify(env),
            'isBrowser': true
        })
    ]
});

config.vue = {
    loaders: {
        css: ExtractTextPlugin.extract({
            loader: 'css-loader',
            fallbackLoader: 'vue-style-loader'
        }),
        sass: ExtractTextPlugin.extract('vue-style-loader', 'css!sass?indentedSyntax'),
        scss: ExtractTextPlugin.extract('vue-style-loader', 'css!sass')
    }
};

config.plugins.push(new ExtractTextPlugin('style.css'));

if (env === 'production') {
    config.plugins.push(
        new webpack.LoaderOptionsPlugin({
            minimize: true
        }),
        new webpack.optimize.UglifyJsPlugin({
            compress: {
                warnings: false
            }
        })
    );
}

module.exports = config;
```
服务端打包时候会忽略 css 等样式文件，浏览器端打包时候就将样式文件单独打包到一个 css 文件。这样在执行 webpack 时候就需要指定 --config 参数来编译不同的版本了，如下：
```
# 编译 客户端
webpack --config webpack.client.js

# 编译 服务器端
webpack --config webpack.server.js
```
同样，入口文件也提出三个文件，index.js, server.js, client.js

index.js
```javascript
import Vue from 'vue';

import App from './vue/App';
import ClipButton from 'Components/ClipButton';
import Toast from 'Components/Toast';

Vue.filter('byte-format', value => {
    const unit = ['Byte', 'KB', 'MB', 'GB', 'TB'];
    let index = 0;
    let size = parseInt(value, 10);

    while (size >= 1024 && index < unit.length) {
        size /= 1024;
        index++;
    }

    return [size.toString().substr(0, 5), unit[index]].join(' ');
});

Vue.use(Toast);
Vue.component('maple-clip-button', ClipButton);

const createApp = function createApp(options) {
    const VueApp = Vue.extend(App);

    return new VueApp(Object.assign({}, options));
};

export {Vue, createApp};
```
index.js 中做一些通用的组件、插件加载，一些全局的设置，最后返回一个可以生成 app 实例的函数供不同环境来调用；
server.js
```javascript
import {createApp} from './index';

export default function (options) {
    const app = createApp(options);

    return new Promise(resolve => {
        resolve(app);
    });
}
```
大部分逻辑已经抽为共用了，所以服务端这里就是简单将 app 实例通过 promise 返回；

client.js
```javascript
import VueResource from 'vue-resource';
import {createApp, Vue} from './index';

Vue.use(VueResource);
const title = 'Test';

const app = createApp({
    data: {
        title
    },
    el: '#app'
});

export default app;
```
客户端也类似，这里在客户端加载 VueResource 这个插件，用于客户端的 ajax 请求；通常通过 ajax 请求服务端返回数据再初始化 app，这样基本就是一个单页的服务端渲染框架了，一般情况下，我们做单页应用还会配合 history.pushState 等通过 URL 做路由分发；这样，我们服务端也最好使用同一套路由来渲染不同的页面。

# 4. 服务端和浏览器路由共用

路由这里使用 vue-router 就可以了，浏览器端还是通过正常的方式载入路由配置，服务端同样载入路由配置，并在渲染之前使用 router.push 渲染需要展现的页面，所以，在通用的入口文件加入路由配置：
```javascript
import Vue from 'vue';
import router from './router';
import App from './vue/App';

const createApp = function createApp(options) {
    const VueApp = Vue.extend(App);

    return new VueApp(Object.assign({
        router
    }, options));
};

export {Vue, router, createApp};
```
路由文件是这样子的：
```javascript
import Vue from 'vue';
import VueRouter from 'vue-router';

import ViewUpload from '../vue/ViewUpload';
import ViewHistory from '../vue/ViewHistory';
import ViewLibs from '../vue/ViewLibs';

Vue.use(VueRouter);

const routes = [
    {
        path: '/',
        component: ViewUpload
    },
    {
        path: '/history',
        component: ViewHistory
    },
    {
        path: '/libs',
        component: ViewLibs
    }
];

const router = new VueRouter({mode: 'history', routes, base: __dirname});
export default router;
```
这里路由的使用 HTML5 的 history 模式；

服务端入口文件这样配置：
```javascript
import {createApp, router} from './index';

export default function (options) {
    const app = createApp({
        data: options.data
    });

    router.push(options.url);
    return new Promise(resolve => {
        resolve(app);
    });
}
```
这里在初始化 app 实例后，调用 router.push(options.url) 将服务端取到的 url push 到路由之中；

# 5. 使用中遇到的坑

整个过程还算顺利，其中遇到最多的问题就是有些模块只能在服务端或者浏览器来使用，而使用 ES6 模块加载是静态的，所以需要将静态加载的模块改为动态加载，所以就有了上面配置 isBrowser 这个全局属性，通过这个属性来判断模块加载了，比如我项目中用到的 clipboard 模块，之前是直接使用 ES6 加载的；
```html
<template>
    <a @click.prevent :href="text"><slot></slot></a>
</template>

<script>
import Clipboard from 'clipboard';

export default {
    props: ['text'],

    mounted() {
        return this.$nextTick(() => {
            this.clipboard = new Clipboard(this.$el, {
                text: () => {
                    return this.text;
                }
            });

            this.clipboard.on('success', () => {
                this.$emit('copied', this.text);
            });
        });
    }
};
</script>
```
这样就会在服务端渲染时候报错，将加载改为动态加载就可以了：
```javascript
let Clipboard = null;

if (isBrowser) {
    Clipboard = require('clipboard');
}
```
如果这个模块在服务端并不会渲染，那后面的代码并不需要更改；

还有 VueResource 插件也是需要浏览器环境的，所以需要将它单独配置在 client.js 中；

6. 总结

同样的路由通过 Vue 服务端渲染后的 HTML 总是一样的，这和 React 渲染后会加上哈希不同，所以可以做渲染后结果的缓存优化，这部分可以参考官方文档的做法，总的来说，Vue 服务端渲染沿袭了 Vue 客户端的轻量做法，也显得比较轻量，唯一不足之处可能就是服务端也同样需要 webpack 来完成。