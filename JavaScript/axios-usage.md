# Axios使用记录

[Axios github](https://github.com/axios/axios)，[中文说明](https://www.kancloud.cn/yunye/axios/234845)

> [Fetch](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch)是JavaScript原生能力，而Axios是一个基于Promise的库，在浏览器中封装的[XMLHttpRequests](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)，在Node中封装的[http](https://nodejs.org/api/http.html)。

Axios主要功能：
- 从浏览器中创建 XMLHttpRequests
- 从 node.js 创建 http 请求
- 支持 Promise API（重要！）
- 拦截请求和响应（常用！）
- 转换请求数据和响应数据（常用！）
- 取消请求（重要！）
- 自动转换 JSON 数据（常用！）
- 客户端支持防御 XSRF

下面开始详细记录试用细节。