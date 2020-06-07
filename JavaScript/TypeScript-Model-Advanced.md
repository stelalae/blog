# TypeScript在Model中的高级应用

> 本文[github地址](https://github.com/stelalae/blog/tree/master/JavaScript/TypeScript-Model-Advanced.md)，[掘金地址](https://juejin.im/post/5edcd9bc518825433139feb1)

在MVC、MVVC等前端经典常用开发模式中，V、C往往是重头戏，可能是前端业务主要集中这两块。结合实际业务，笔者更喜欢路由模式、插件式设计，这种在迭代和维护上更能让开发者收益（不过你需要找PM协调这事，毕竟他们理解的简化用户体验，多半是怎么让用户操作简单）。但我们今天来看看Model，看看M有什么扩展的可能。

> 如果读者熟悉iOS开发，应该听过VIPER开发模式，下面推荐
> - [iOS开发 使用viper架构构建复杂页面](https://juejin.im/post/5e1dadbd51882536a627f4f9)
> - [使用 VIPER 构建 iOS 应用](https://objccn.io/issue-13-5/)
> - [积木法搭建 iOS 应用—— VIPER](https://mp.weixin.qq.com/s/ovmd_YTH3I992hO_jBiNog)

## 背景

在读到本文之前，你实际项目（如React+Redux）中请求服务器数据，可能是如下策略：
1. componentDidMount 中发送redux action请求数据；
2. 在action中发起异步网络请求，当然你已经对网络请求有一定封装；
3. 在网络请求内部处理一定异常和边际逻辑，然后返回请求到的数据；
4. 拿到数据this.setState刷新页面，同时可能存一份到全局redux中；

正常情况下，一个接口对应至少一个接口响应Model，万一你还定义了接口请求的Model、一个页面有5个接口呢？

如果项目已经引入TypeScript，结合编写Model，你的编写体验肯定会如行云流水般一气呵成！但实际开发中，你还需要对服务器返回的数据、页面间传递的参数等涉及到数据传递的地方，做一些数据额外工作：
- 对null、undefined等空值的异常处理（在ES最新方案和TS支持里，新增：链式调用?和运算符??，请读者自行查询使用手册）；
- 对sex=0、1、2，time=1591509066等文案转义；
- （还有其他吗？欢迎留言补充）

作为一个优秀且成熟的开发者，你肯定也已经做了上述额外的工作，在utils文件下编写了几十甚至上百的tool类函数，甚至还根据函数用途做了分类：时间类、年龄性别类、数字类、......，接着你在需要的地方import，然后你开始进行传参调用。是的，一切看上去都很完美！

> 上面这个流程说的就是笔者本人，:)。

### 现况

随着项目和业务的迭代，加上老板还是压时间，最坏的情况是你遇到了并没有遵守上述"开发规范"的同事，那结果只能是呵呵呵呵呵了。下面直接切入正题吧！

上述流程虽说有一定设计，但没有做到高内聚、低耦合的原则，个人觉得不利于项目后期迭代和局部重构。
> 推荐另一个设计原则：[面向对象五大原则SOLID](https://juejin.im/post/5af5935d6fb9a07ac480155d)

下面举个例子：
- 接口里字段发生变更时，如性别从Sex改为Gender；
- 前端内部重构，发现数据模型不匹配时，页面C支持从页面A附加参数a、或页面B附加参数b跳入，重构后页面B1附加参数b1也要跳转C。从设计来说肯定是让B1尽量按照以前B去适配时是最好的，否则C会越来越重。

上面提过不管是页面交互，还是业务交互，最根本根本是数据的交换传递，从而去影响页面和业务。 **数据就是串联页面和业务的核心，Model就是数据的表现形式。**

再比如现在前后端分离的开发模式下，在需求确认后，开发需要做的第一件事是数据库设计和接口设计，简单的说就是字段的约定，然后在进行页面开发，最终进行接口调试和系统调试，一直到交付测试。这期间，后端需要执行接口单元测试、前端需要Mock数据开发页面。

## 如何解决

### 接口管理

目前笔记是通过JSON形式来进行接口管理，在项目初始化时，将配置的接口列表借助于 [dva](https://github.com/dvajs/dva) 注册到Redux Action中，然后接口调用就直接发送Action即可。
最终到拿到服务器响应的Data。

接口配置（对应下面第二版）：
```json
list: [
  {
    alias: 'getCode',
    apiPath: '/user/v1/getCode',
    auth: false,
  },
  {
    alias: 'userLogin',
    apiPath: '/user/v1/userLogin',
    auth: false,
    nextGeneral: 'saveUserInfo',
  },
  {
    alias: 'loginTokenByJVerify',
    apiPath: '/user/v1/jgLoginApi',
    auth: false,
    nextGeneral: 'saveUserInfo',
  },
]
```


第一版：
```javascript
import { apiComm, apiMy } from 'services';

export default {
  namespace: 'bill',
  state: {},
  reducers: {
    updateState(state, { payload }) {
      return { ...state, ...payload };
    },
  },
  effects: {
    *findBydoctorIdBill({ payload, callback }, { call }) {
      const res = yield call(apiMy.findBydoctorIdBill, payload);
      !apiComm.IsSuccess(res) && callback(res.data);
    },
    *findByDoctorIdDetail({ payload, callback }, { call }) {
      const res = yield call(apiMy.findByDoctorIdDetail, payload);
      !apiComm.IsSuccess(res) && callback(res.data);
    },
    *findStatementDetails({ payload, callback }, { call }) {
      const res = yield call(apiMy.findStatementDetails, payload);
      !apiComm.IsSuccess(res) && callback(res.data);
    },
  },
};
```


第二版使用高阶函数，同时支持服务器地址切换，减少冗余代码：
```javascript
export const connectModelService = (cfg: any = {}) => {
  const { apiBase = '', list = [] } = cfg;
  const listEffect = {};
  list.forEach(kAlias => {
    const { alias, apiPath, nextGeneral, cbError = false, ...options } = kAlias;
    const effectAlias = function* da({ payload = {}, nextPage, callback }, { call, put }) {
      let apiBaseNew = apiBase;
      // apiBaseNew = urlApi;
      if (global.apiServer) {
        apiBaseNew = global.apiServer.indexOf('xxx.com') !== -1 ? global.apiServer : apiBase;
      } else if (!isDebug) {
        apiBaseNew = urlApi;
      }
      const urlpath =
        apiPath.indexOf('http://') === -1 && apiPath.indexOf('https://') === -1 ? `${apiBaseNew}${apiPath}` : apiPath;
      const res = yield call(hxRequest, urlpath, payload, options);
      const next = nextPage || nextGeneral;
      // console.log('=== hxRequest res', next, res);
      if (next) {
        yield put({
          type: next,
          payload,
          res,
          callback,
        });
      } else if (cbError) {
        callback && callback(res);
      } else {
        hasNoError(res) && callback && callback(res.data);
      }
    };
    listEffect[alias] = effectAlias;
  });
  return listEffect;
};
```

上面看上去还不错，解决了接口地址管理、封装了接口请求，但自己还得处理返回Data里的异常数据。

另外的问题是，接口和对应的请求与响应的数据Model并没有对应起来，后面再次看代码需要一点时间才能梳理业务逻辑。

请读者思考一下上面的问题，然后继续往下看。

### Model管理

一个接口必然对应唯一一个请求Model和唯一一个响应Model。对，没错！下面利用此机制进一步讨论。

所以通过响应Model去发起接口请求，在函数调用时也能利用请求Model判定入参合不合理，这样就把主角从接口切换到Model了。这里个人觉得优先响应Model比较合适，更能直接明白这次请求后拿到的数据格式。

下面先看看通过Model发起请求的代码：

```typescript
SimpleModel.get(
  { id: '1' },
  { auth: false, onlyData: false },
).then((data: ResponseData<SimpleModel>) =>
  setTimeout(
    () =>
      console.log(
        '设置返回全部数据，返回 ResponseData<T> 或 ResponseData<T[]>',
        typeof data,
        data,
      ),
    2000,
  ),
);
```

其中，SimpleModel是定义的响应Model，第一个参数是请求，第二个参数是请求配置项，接口地址被隐藏在SimpleModel内部了。

```typescript
import { Record } from 'immutable';

import { ApiOptons } from './Common';
import { ServiceManager } from './Service';

/**
 * 简单类型
 */
const SimpleModelDefault = {
  a: 'test string',
  sex: 0,
};

interface SimpleModelParams {
  id: string;
}

export class SimpleModel extends Record(SimpleModelDefault) {
  static async get(params: SimpleModelParams, options?: ApiOptons) {
    return await ServiceManager.get<SimpleModel>(
      SimpleModel,
      'http://localhost:3000/test',   // 被隐藏的接口地址
      params,
      options,
    );
  }

  static sexMap = {
    0: '保密',
    1: '男',
    2: '女',
  };

  sexText() {
    return SimpleModel.sexMap[this.sex] ?? '保密';
  }
}
```

这里借助了immutable里的[Record](https://immutable-js.github.io/immutable-js/docs/#/Record)，目的是将JSON Object反序列化为Class Object，目的是提高Model在项目中相关函数的内聚。更多介绍请看我另外一篇文章：[JavaScript的强语言之路—另类的JSON序列化与反序列化](https://github.com/stelalae/blog/blob/master/JavaScript/alternative-json-serialization-deserialization.md)。

```typescript
// utils/tool.tsx
export const sexMap = {
  0: '保密',
  1: '男',
  2: '女',
};

export const sexText = (sex: number) => {
  return sexMap[sex] ?? '保密';
};
```

直接在SimpleModel内部用this访问具体数据，比调用utils/tool函数时传入外部参数，更为内聚和方便维护。通过这种思路，相信你可以创造更多"黑魔法"的语法糖！

接着我们来看看 `Common` 文件内容：

```typescript
/**
 * 接口响应，最外层统一格式
 */
export class ResponseData<T = any> {
  code? = 0;
  message? = '操作成功';
  toastId? = -1;
  data?: T;
}

/**
 * api配置信息
 */
export class ApiOptons {
  headers?: any = {}; // 额外请求头
  loading?: boolean = true; // 是否显示loading
  loadingTime?: number = 2; // 显示loading时间
  auth?: boolean = true; // 是否需要授权
  onlyData?: boolean = true; // 只返回data
}

/**
 * 枚举接口能返回的类型
 * - T、T[] 在 ApiOptons.onlyData 为true时是生效
 * - ResponseData<T>、ResponseData<T[]> 在 ApiOptons.onlyData 为false时是生效
 * - ResponseData 一般在接口内部发生异常时生效
 */
export type ResultDataType<T> =
  | T
  | T[]
  | ResponseData<T>
  | ResponseData<T[]>
  | ResponseData;

```

`Service`文件内部是封装了axios：

```typescript
import axios, { AxiosRequestConfig, AxiosResponse } from 'axios';
import { ApiOptons, ResponseData, ResultDataType } from './Common';

/**
 * 模拟UI loading
 */
class Toast {
  static loading(txt: string, time: number = 3) {
    console.log(txt, time);
    return 1;
  }
  static info(txt: string, time: number = 3) {
    console.log(txt, time);
    return 1;
  }
  static remove(toastId: number) {
    console.log(toastId);
  }
}

/**
 * 未知(默认)错误码
 */
const codeUnknownTask = -999;

/**
 * 接口请求封装基类
 */
export class InterfaceService {
  /**
   * todo
   */
  private static userProfile: { sysToken?: '' } = {};
  public static setUser(_user: any) {
    InterfaceService.userProfile = _user;
  }

  constructor(props: ApiOptons) {
    this.options = props;
  }
  /**
   * 默认配置
   */
  public options = new ApiOptons();

  /**
   * todo
   */
  public get sysToken(): string {
    return InterfaceService.userProfile?.sysToken ?? '';
  }

  /**
   * 构建header
   */
  public get headers(): Object {
    return {
      Accept: 'application/json',
      'Content-Type': 'application/json; charset=utf-8',
      'app-info-key': 'xxx', // 自定义字段
    };
  }

  /**
   * 请求前置条件。可根据自己情况重构此函数
   */
  preCheck() {
    if (this.options.loading && this.options.loadingTime > 0) {
      return Toast.loading('加载中...', this.options?.loadingTime ?? 3);
    }
    return -1;
  }

  /**
   * 下载json，返回对象
   */
  public static async getJSON(url: string) {
    try {
      const res = await fetch(url);
      return await res.json();
    } catch (e) {
      console.log(e);
      return {};
    }
  }
}

/**
 * 接口请求封装(axios版，也可以封装其他版本的请求)
 */
export class InterfaceAxios extends InterfaceService {
  constructor(props: ApiOptons) {
    super(props);
  }

  /**
   * 封装axios
   */
  private request = (requestCfg: AxiosRequestConfig): Promise<ResponseData> => {
    return axios(requestCfg)
      .then(this.checkStatus)
      .catch((err: any) => {
        // 后台接口异常，如接口不通、http状态码非200、data非json格式，判定为fatal错误
        console.log(requestCfg, err);
        return {
          code: 408,
          message: '网络异常',
        };
      });
  };

  /**
   * 检查网络响应状态码
   */
  private checkStatus(response: AxiosResponse<ResponseData>) {
    if (response.status >= 200 && response.status < 300) {
      return response.data;
    }
    return {
      code: 408,
      message: '网络数据异常',
    };
  }

  /**
   * 发送POST请求
   */
  public async post(url: string, data?: any) {
    const toastId = this.preCheck();
    const ret = await this.request({
      url,
      headers: this.headers,
      method: 'POST',
      data: Object.assign({ sysToken: this.sysToken }, data),
    });
    ret.toastId = toastId;

    return ret;
  }

  /**
   * 发送GET请求
   */
  public async get(url: string, params?: any) {
    const toastId = this.preCheck();
    const ret = await this.request({
      url,
      headers: this.headers,
      method: 'GET',
      params: Object.assign({ sysToken: this.sysToken }, params),
    });
    ret.toastId = toastId;
    return ret;
  }
}

export class ServiceManager {
  /**
   * 检查接口数据
   */
  public hasNoError(res: ResponseData) {
    if (res.toastId > 0) {
      Toast.remove(res.toastId);
    }
    if (res?.code !== 0 && res.code !== codeUnknownTask) {
      Toast.info(res?.message ?? '服务器出错');
      return false;
    }
    return true;
  }

  /**
   * 解析响应
   */
  public static parse<T>(
    modal: { new (x: any): T },
    response: any,
    options: ApiOptons,
  ): ResultDataType<T> {
    if (!response || !response.data) {
      response.data = new modal({});
    } else {
      if (response.data instanceof Array) {
        response.data = response.data.map((item: T) => new modal(item));
      } else if (response.data instanceof Object) {
        response.data = new modal(response.data);
      }
      return options.onlyData ? response.data : response;
    }
  }

  /**
   * post接口请求
   */
  public static async post<T>(
    modal: { new (x: any): T },
    url: string,
    body?: any,
    options: ApiOptons = new ApiOptons(),
  ): Promise<ResultDataType<T>> {
    // 使用合并，减少外部传入配置
    options = Object.assign(new ApiOptons(), options);

    const request = new InterfaceAxios(options);
    if (options.auth && !request.sysToken) {
      return {
        code: 403,
        message: '未授权',
      };
    }

    try {
      const response = await request.post(url, body);
      return ServiceManager.parse<T>(modal, response, options);
    } catch (err) {
      // 记录错误日志
      console.log(url, body, options, err);
      return {
        code: codeUnknownTask,
        message: '内部错误，请稍后再试',
      };
    }
  }

  /**
   * get接口请求
   */
  public static async get<T>(
    modal: { new (x: any): T },
    url: string,
    params?: any,
    options: ApiOptons = new ApiOptons(),
  ): Promise<ResultDataType<T>> {
    // 使用合并，减少外部传入配置
    options = Object.assign(new ApiOptons(), options);

    const a = new InterfaceAxios(options);
    const request = new InterfaceAxios(options);
    if (options.auth && !request.sysToken) {
      return {
        code: 403,
        message: '未授权',
      };
    }

    try {
      const response = await a.get(url, params);
      return ServiceManager.parse<T>(modal, response, options);
    } catch (err) {
      // 记录错误日志
      console.log(url, params, options, err);
      return {
        code: codeUnknownTask,
        message: '内部错误，请稍后再试',
      };
    }
  }
}
```

Service文件里内容有点长，主要有下面几个类：

- Toast：模拟请求接口时的loading，可通过接口调用时来配置；
- InterfaceService：接口请求的基类，内部记录当前用户的Token、多环境服务器地址切换（代码中未实现）、单次请求的接口配置、自定义Header、请求前的逻辑检查、直接请求远端JSON配置文件；
- InterfaceAxios：继承于InterfaceService，即axios版的接口请求，内部发起实际请求。你可以封装fetch版本的。
- ServiceManager：提供给Model使用的请求类，传入响应Model和对应服务器地址后，等异步请求拿到数据后再将响应数据Data解析成对应的Model。

下面再贴一下完整的Model发起请求示例：

```typescript
import { ResponseData, ApiOptons, SimpleModel } from './model';

// 接口配置不同的三种请求
SimpleModel.get({ id: '1' }).then((data: ResponseData) =>
  setTimeout(
    () =>
      console.log(
        '因需授权导致内部异常，返回 ResponseData：',
        typeof data,
        data,
      ),
    1000,
  ),
);

SimpleModel.get(
  { id: '1' },
  { auth: false, onlyData: false },
).then((data: ResponseData<SimpleModel>) =>
  setTimeout(
    () =>
      console.log(
        '设置返回全部数据，返回 ResponseData<T> 或 ResponseData<T[]>',
        typeof data,
        data,
      ),
    2000,
  ),
);

SimpleModel.get(
  { id: '1' },
  { auth: false, onlyData: true },
).then((data: SimpleModel) =>
  setTimeout(
    () =>
      console.log(
        '仅返回关键数据data，返回 T 或 T[]：',
        typeof data,
        data,
        data.sexText(),
      ),
    3000,
  ),
);

```

控制台打印结果。注意，返回的 `data` 可能是JSON Object，也可能是 Immutable-js Record Object。
```
加载中... 2
加载中... 2
因需授权导致内部异常，返回 ResponseData： object { code: 403, message: '未授权' }
设置返回全部数据，返回 ResponseData<T> 或 ResponseData<T[]> object {
  code: 0,
  message: '1',
  data: SimpleModel {
    __ownerID: undefined,
    _values: List {
      size: 2,
      _origin: 0,
      _capacity: 2,
      _level: 5,
      _root: null,
      _tail: [VNode],
      __ownerID: undefined,
      __hash: undefined,
      __altered: false
    }
  },
  toastId: 1
}
仅返回关键数据data，返回 T 或 T[]： object SimpleModel {
  __ownerID: undefined,
  _values: List {
    size: 2,
    _origin: 0,
    _capacity: 2,
    _level: 5,
    _root: null,
    _tail: VNode { array: [Array], ownerID: OwnerID {} },
    __ownerID: undefined,
    __hash: undefined,
    __altered: false
  }
} 男
```

最后再补充一个常见的复合类型Model示例：

```typescript
/**
 * 复杂类型
 */

const ComplexChildOneDefault = {
  name: 'lyc',
  sex: 0,
  age: 18,
};

const ComplexChildTwoDefault = {
  count: 10,
  lastId: '20200607',
};

const ComplexChildThirdDefault = {
  count: 10,
  lastId: '20200607',
};

// const ComplexItemDefault = {
//   userNo: 'us1212',
//   userProfile: ComplexChildOneDefault,
//   extraFirst: ComplexChildTwoDefault,
//   extraTwo: ComplexChildThirdDefault,
// };

// 复合类型建议使用class，而不是上面的object。因为object里不能添加可选属性?
class ComplexItemDefault {
  userNo = 'us1212';
  userProfile = ComplexChildOneDefault;
  extraFirst? = ComplexChildTwoDefault;
  extraSecond? = ComplexChildThirdDefault;
}

// const ComplexListDefault = {
//   list: [],
//   pageNo: 1,
//   pageSize: 10,
//   pageTotal: 0,
// };

// 有数组的复合类型，如果要指定数组元素的Model，就必须用class
class ComplexListDefault {
  list: ComplexItemDefault[] = [];
  pageNo = 1;
  pageSize = 10;
  pageTotal = 0;
}

interface ComplexModelParams {
  id: string;
}

// 因为使用的class，所以需要 new 一个去初始化Record
export class ComplexModel extends Record(new ComplexListDefault()) {
  static async get(params: ComplexModelParams, options?: ApiOptons) {
    return await ServiceManager.get<ComplexModel>(
      ComplexModel,
      'http://localhost:3000/test2',
      params,
      options,
    );
  }
}
```

下面是调用代码：

```typescript
ComplexModel.get({ id: '2' }).then((data: ResponseData) =>
  setTimeout(
    () =>
      console.log(
        '因需授权导致内部异常，返回 ResponseData：',
        typeof data,
        data,
      ),
    1000,
  ),
);

ComplexModel.get(
  { id: '2' },
  { auth: false, onlyData: false },
).then((data: ResponseData<ComplexModel>) =>
  setTimeout(
    () =>
      console.log(
        '设置返回全部数据，返回 ResponseData<T> 或 ResponseData<T[]>',
        typeof data,
        data.data.toJSON(),
      ),
    2000,
  ),
);

ComplexModel.get(
  { id: '2' },
  { auth: false, onlyData: true },
).then((data: ComplexModel) =>
  setTimeout(
    () =>
      console.log(
        '仅返回关键数据data，返回 T 或 T[]：',
        typeof data,
        data.toJSON(),
      ),
    3000,
  ),
);
```

接着是打印结果。这次Immutable-js Record Object就调用了`data.toJSON()`转换成原始的JSON Object。

```
加载中... 2
加载中... 2
因需授权导致内部异常，返回 ResponseData： object { code: 403, message: '未授权' }
设置返回全部数据，返回 ResponseData<T> 或 ResponseData<T[]> object {
  list: [ { userNo: '1', userProfile: [Object] } ],
  pageNo: 1,
  pageSize: 10,
  pageTotal: 0
}
仅返回关键数据data，返回 T 或 T[]： object {
  list: [ { userNo: '1', userProfile: [Object] } ],
  pageNo: 1,
  pageSize: 10,
  pageTotal: 0
}
```



## 总结

本文的代码地址：[https://github.com/stelalae/node_demo](https://github.com/stelalae/node_demo)，欢迎follow me~

现在接口调用是不是很优雅？！只关心请求和影响的数据格式，多使用高内聚低耦合，这对项目持续迭代非常有帮助的。使用TypeScript和Immutable-js来处理数据，在大型应用中越来越深入，从数据管理出发可以优化上层UI显示和业务逻辑。



---

参考地址：

1. [TypeScript 在复杂 Immutable.js 数据结构下的用法总结](https://zhuanlan.zhihu.com/p/58679875)
2. [来玩TypeScript啊，机都给你开好了！](https://zhuanlan.zhihu.com/c_206498766)
3. [JavaScript的强语言之路—另类的JSON序列化与反序列化](https://github.com/stelalae/blog/blob/master/JavaScript/alternative-json-serialization-deserialization.md)

