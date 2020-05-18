# umijs框架使用记录

快速创建umi项目：
```JavaScript
$ yarn global add umi
$ umi -v

$ mkdir myapp && cd myapp
$ yarn create umi   // 根据提示选择配置
```

## 配置

参考[官方说明](https://umijs.org/zh-CN/docs/config)。

### 定制化index.html

umi 约定使用 src/pages/document.ejs 作为默认模板，内容需保证有 `<div id="root"></div>`。模板基于 ejs 渲染，参考 [ejs](https://github.com/mde/ejs) 的使用。

模板里可通过 context 来获取到 umi 提供的变量，context 包含：
* route，路由对象，包含 path、component 等；
* config，用户配置信息；
* publicPath，通 webpack 里的 output.publicPath 配置；
* env，环境变量，值为 development 或 production；
* 其他在路由上通过 context 扩展的配置信息；

```
<link rel="icon" type="image/x-icon" href="<%= context.publicPath %>favicon.png" />

<% if(context.env === 'production') { %>
  <h2>生产环境</h2>
<% } else {%>
  <h2>开发环境</h2>
<% } %>

<div><%= JSON.stringify(context.config) %></div> 
```

**重点说下config，包含就是项目文件.umirc.ts里的信息，所以叫做用户配置信息。**
```JavaScript
import { IConfig } from 'umi-types';

// ref: https://umijs.org/config/
const config: IConfig = {};

export default config;
```
如果config为空，则context.config只会输出`{"mountElementId":"root"}`，下面是默认的.umirc.ts配置：
```JavaScript
import { IConfig } from 'umi-types';

// ref: https://umijs.org/config/
const config: IConfig = {
  define: {
    apiServer: 'https://api.gihub.com',
    version: '1.0.0',
    channel: 'github',
  },
  treeShaking: true,
  ssr: false,
  hash: true,
  publicPath: './',
  routes: [
    {
      path: '/',
      component: '../layouts/index',
      routes: [{ path: '/', component: '../pages/index' }],
    },
  ],
  plugins: [
    // ref: https://umijs.org/plugin/umi-plugin-react.html
    [
      'umi-plugin-react',
      {
        antd: false,
        dva: true,
        dynamicImport: { webpackChunkName: true },
        title: 'ssrdemo',
        dll: false,

        routes: {
          exclude: [
            /models\//,
            /services\//,
            /model\.(t|j)sx?$/,
            /service\.(t|j)sx?$/,
            /components\//,
          ],
        },
      },
    ],
  ],
};

export default config;
```
这时 `JSON.stringify(context.config)` 会输出：
```
{"define":{"apiServer":"https://api.gihub.com","version":"1.0.0","channel":"github"},"treeShaking":true,"ssr":false,"hash":true,"publicPath":"./","routes":[{"path":"/","component":"../layouts/index","routes":[{"path":"/","component":"../pages/index"}]}],"plugins":[["umi-plugin-react",{"antd":false,"dva":true,"dynamicImport":{"webpackChunkName":true},"title":"ssrdemo","dll":false,"routes":{"exclude":[{},{},{},{},{}]}}]],"mountElementId":"root"}
```
当一个项目代码会多端复用时，根据官方的配置指导，我们应该根据不同渠道创建不同.umirc.ts，然后在define下指定channel变量等，这是我们就可以直接在业务中直接使用这些变量 console.log(\`${channel}\`) 。
但可能使用不太方便，特别是在ts项目里，所以需要将自定义变量赋值到window对象上，将下面代码加入到`document.ejs`即可：
```html
<script>  
function init() {
var temp = document.createElement("div");
temp.innerHTML = '<%=JSON.stringify(context.config)%>';
try {
  window.g_config = JSON.parse(temp.innerText || temp.textContent);
} catch(e) {
  window.g_config = {'define': {}}; // 避免从g_config再做异常判断
}
temp = null; 
}
init();
</script>
```
注意ejs里JSON.stringify输出的内容被HTML转义过，所以需解码。



参考资料：
* [UmiJS](https://umijs.org/zh-CN)
* [JS对HTML字符的转义和反转义](https://my.oschina.net/u/1168056/blog/1796196)