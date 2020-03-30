# vue-ssr-dem
## vue服务端渲染探索
优势：1、利于SEO 2、缩短首屏渲染时间

劣势：1、学习成本增高 2、服务端增加压力

## 一、服务端渲染的关键点

服务端渲染主要体现在2方面

1. 服务端输出html字符串

2. 服务端和客户端部分代码同构

## 二、服务端渲染的方式

服务端渲染和SPA不同，需要有client和server俩个入口进行打包，因为需要在服务端和客户端分别渲染

![avatar](/user/desktop/doge.png)

1、entry-client.js(客户端)中简单的挂载#app



2、entry-server.js（服务端）中是一个简单的promise回调函数，主要是保证在服务端渲染之前获取到初始数据然后返回app组件实例，



router.onReady()为了保证router中的异步懒加载组件加载完成

router.getMatchedComponents()用于获取与上下文访问的url匹配的组件



利用组件中的asyncData属性来判断哪些组件需要预获取数据，通过promise.all的回调保证所有的异步请求数据获取到后，

会把数据存到store中，同时把store.state的值赋值给context.state，context.state这个属性很重要，这个会在服务端渲染时借助vue-server-renderer插件在HTML模板中添加

window.__INITIAL_STATE__属性。asyncData组件中的位置如下：



3、客户端index.html模板文件



前端资源挂载到app的dom中。

服务端ndex.ssr.html模板文件



服务端渲染的html字符串会通过vue-server-renderer插件识别出<!--vue-ssr-outlet-->标记的占位符，插入到此位置（所以模板中这个占位符可不要删除哦）

4、store文件



store中会在浏览器端渲染的时候对windows中的__INITIAL_STATE__属性做判断，主要是为了保证服务端和前端的state状态一致，前端激活服务端渲染的

页面后可直接从store中取值。

三、打包配置：

1、添加3个配置文件webpack.base.config.js、webpack.client.config.js、webpack.server.config.js

webpack.base.config.js是基础配置文件，里面配置一些常用的loader，这里就不展示了。

webpack.client.config.js文件：



merge了base文件，入口为客户端的entry-client.js

VueSSRClientPlugin插件作用是生成客户端json文件vue-ssr-client-manifest.json，为后续服务端渲染插入index.ssr.html中使用。

webpack.server.config.js文件：



同样merge了base文件，入口为客户端的entry-server.js



VueSSRServerPlugin插件作用是生成服务端的json文件vue-ssr-server-bundle.json，作用server.js文件中会讲



target: 'node'的作用是让webpack以适用node的形式处理导入文件方式



libraryTarget: 'commonjs2'使用node风格导出模块，用于server端渲染

2、server.js文件

server文件为后端服务文件，其中主要处理服务端渲染，以及页面请求的输出等



这段代码主要是使用vue-server-renderer插件，来处理服务端渲染，插件中的createBundleRenderer方法可以将服务端的bundle.json文件

和服务端的模板index.ssr.html以及客户端的json资源文件融合，serverBundle文件是服务端导出的app实例组价，在这里会被插入进模板中，

clientManifest静态资源也会在适当机会defer进模板中。



在render中融合的资源通过render的renderToStream方法输出html字符串返回给浏览器端

然后在浏览器端渲染输出。

服务端渲染ssr页面展示如下：



图中1的位置为server端打包的json文件渲染得出的

图中2的位置为context.state = store.state的值，在服务端渲染时复制给window中__INITIAL_STATE__属性

并插入模板

图中3的位置为client端生成的json文件vue-ssr-client-manifest.json，在服务端解析渲染时插入的

首屏渲染后，后面页面的操作信息由客户端接管。

四、总结

1、服务端渲染有些学习成本，对node要有基础

2、最好只用于首屏渲染（其他地方使用，感觉作用不大，还浪费资源）

3、服务端渲染首屏初始化数据不要太多（服务端渲染请求数据的时间也会导致白屏显示时间过长）
