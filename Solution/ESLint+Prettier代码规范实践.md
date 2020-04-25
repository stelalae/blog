# ESLint+Prettier代码规范实践

[Demo地址](https://github.com/stelalae/rn_demo)

## 介绍

代码规范校验使用 ESLint，但是一开始 ESLint 只有检测告诉你哪里有问题，常常出现的情况就是一堆 warning，改起来很痛苦。后来 ESLint 提供了 `$ ESLint filename --fix ` 的命令可以自动帮你修复一些不符合规范的代码。

Prettier 是一个代码格式化工具，可以帮你把代码格式化成可读性更好的格式，最典型的就是一行代码过长的问题。

### eslint和prettier区别

eslint（包括其他一些 lint 工具）的主要功能包含代码格式的校验，代码质量的校验。
prettier 只是代码格式的校验（并格式化代码），不会对代码质量进行校验。
代码格式问题通常指的是：单行代码长度、tab长度、空格、逗号表达式等问题。而代码质量问题指的是：未使用变量、三等号、全局变量声明等问题。

### eslint和prettier配合使用

为什么要两者配合使用？因为，第一在 ESLint 推出 --fix 参数前，ESLint 并没有自动化格式代码的功能，要对一些格式问题做批量格式化只能用 Prettier 这样的工具。第二 ESLint 的规则并不能完全包含 Prettier 的规则，两者不是简单的谁替代谁的问题。但是在 ESLint 推出 --fix 命令行参数之后，如果你觉得 ESLint 提供的格式化代码够用了，也可以不使用 Prettier。

ESLint 和 Prettier 相互合作的时候有一些问题，对于他们交集的部分规则，ESLint 和 Prettier 格式化后的代码格式不一致。导致的问题是：当你用 Prettier 格式化代码后再用 ESLint 去检测，会出现一些因为格式化导致的 warning。
这个时候有两个解决方案：
1. 运行 Prettier 之后，再使用 eslint --fix 格式化一把，这样把冲突的部分以 ESLint 的格式为标准覆盖掉，剩下的 warning 就都是代码质量问题了。
2. 在配置 ESLint 的校验规则时候把和 Prettier 冲突的规则 disable 掉，然后再使用 Prettier 的规则作为校验规则。那么使用 Prettier 格式化后，使用 ESLint 校验就不会出现对前者的 warning。

为什么不能先使用 ESLint 再使用 Prettier？
针对方案1，如果你后使用 Prettier，那么格式化后提交的代码在下一次或者别人 checkout 代码后是通不过 lint 校验的。
针对方案2，其实是可以的，但是本人在实践中看社区方案的时候有提到某些情况下 eslint --fix 和 prettier 混用会出现格式问题。所以保险起见还是先用 perttier 格式化，再用 eslint 命令校验，而不用 eslint --fix 命令去格式化。

### 使用总结

1、前端项目必然要使代码规范，不论是 js 还是 ts 项目。
2、如前面说说 prettier 是擅长代码格式的校验，eslint擅长代码质量的校验。虽说 eslint 也能格式校验，但是并没有 prettier 那么强势。对于 eslint 和 prettier 使用顺序问题，前面已做解释。
3、eslint 和 prettier 的配置和依赖一定要跟随依赖，比如环境差异导致无谓的git commit。比如 vscode 及其插件（prittier）很容易做到编码格式风格统一，逐步强化团队协作中代码统一问题，再结合 git hook 做到 eslint 检验通过，这时团队代码风格和质量就能保持一致。

## 项目实践

目前有三个流行方案，分别是 [Airbnb](https://github.com/airbnb/javascript) 、[Standard](https://standardjs.com/readme-zhcn.html)、[Google](https://github.com/google/eslint-config-google)，严格程度对比是 Airbnb > Google > Standard，所以普遍使用Airbnb的多。

package.json内容概览：
```JSON
{
  "scripts": {
    "prettier": "prettier --write --single-quote \"src/**/*.tsx\"",
    "lint": "eslint --fix --ext .tsx ./src",
    "format": "yarn prettier && yarn lint "
  }
  "lint-staged": {
    "src/**/*.tsx": [
      "yarn format",
      "git add"
    ]
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "devDependencies": {
    "@typescript-eslint/eslint-plugin": "^2.13.0",
    "@typescript-eslint/parser": "^2.13.0",
    "babel-eslint": "^10.0.1",
    "eslint": "^6.8.0",
    "eslint-config-prettier": "^6.9.0",
    "eslint-import-resolver-babel-module": "^5.1.0",
    "eslint-plugin-import": "^2.19.1",
    "eslint-plugin-prettier": "^3.1.2",
    "eslint-plugin-react": "^7.17.0",
    "lint-staged": "^8.1.7",
    "prettier": "^1.19.1",
    "react-native-typescript-transformer": "^1.2.12",
    "typescript": "^3.7.4"
  },
}
```
首先安装基础依赖：`yarn add -D prettier eslint`，安装到项目中的目的是，在其他电脑上免得再配置环境。

### .eslintrc.js

```JavaScript
module.exports = {
  env: {
    browser: true,
    es6: true,
    node: true,
  },
  extends: [
    'airbnb-base',
  ],
  globals: {
    Atomics: 'readonly',
    SharedArrayBuffer: 'readonly',
  },
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 2018,
    sourceType: 'module',
  },
  plugins: [
    '@typescript-eslint',
  ],
  rules: {
  },
};
```
**env** 用于指定环境，每个环境都有自己预定义的全局变量，可以同时指定多个环境，不矛盾。如console属性只有在browser环境下才会存在，如果不设置支持browser，那么可能会console is undefined。
```JavaScript
env: {
    browser: true,
    es6: true,
    node: true,
    commonjs: true,
    mocha: true,
  },
```

**extends** 属性值可以是一个字符串或字符串数组，数组中每个配置项继承它前面的配置，可以对继承的规则进行修改、覆盖和拓展。比如我们的 extends 就继承了 Airbnb 的配置规则。
```JavaScript
extends: [
    'airbnb-base',
    // 'eslint:recommended',
    // 'plugin:react/recommended',
    // 'plugin:@typescript-eslint/eslint-recommended',
    // 'plugin:@typescript-eslint/recommended',
    // 'prettier/@typescript-eslint',
    // 'plugin:prettier/recommended',
  ]
```

**globals** 脚本在执行期间访问的额外的全局变量。（好像没啥用）

**parser** 指定一个不同的解析器。ESLint默认使用Espree作为其解析器， TypeScript项目使用 TypeScript 团队与 ESLint 联合发布的 typescript-eslint 解析器，它非常好地兼容了 TypeScript 和 eslint 的解析特性。还有Esprima、Babel-ESLint等解析器。
```bash
yarn add -D @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

**parserOptions** 指定解析器要想使用的环境配置参数。
```JavaScript
parserOptions: {
    ecmaVersion: 2019,  // 默认是5，在我们这里是用了最新的 2019。
    sourceType: 'module',   // 默认'script'，但一般使用ECMAScript模块形式''module
    ecmafeatures: {
      globalReturn: false,  //允许在全局作用域下使用return语句
      impliedStrict: false,  //启用全局strict模式（严格模式）
      jsx: false,   //启用JSX
      experimentalObjectRestSpread: false   //启用对实验性的objectRest/spreadProperties的支持
    }
  }
```

**plugins** 添加第三方插件，记得提前安装对应devDependencies。extend 提供的是 eslint 现有规则的一系列预设，而 plugin 则提供了除预设之外的自定义规则，当你在 eslint 的规则里找不到合适的的时候，就可以借用插件来实现。
```JavaScript
plugins: ['prettier', '@typescript-eslint'],
```

**rules** 额外的ESLint具体规则，[完整的规则列表](https://cn.eslint.org/docs/rules/)。

### prettier.config.js

[官方配置说明](https://prettier.io/docs/en/options.html)，入门可看[prettier指北](https://juejin.im/post/5d4304be51882553ec6eefa8)。

如前面的介绍，prettier只是用来对代码的格式进行校验，并格式化代码，所以规则比eslint简单很多，所以才有`eslint --fix --ext .tsx ./src`能取代preittier的说法。
在vscode中安装插件`Prettier - Code formatter`后，开启vscode的保存时格式化，可自动帮你完成preittier的格式化。我一般配置prettier.config.js，而不是vscode-prettier插件的配置，这样子能做到项目独立性。

### 实战

因为是React项目，所以需要添加React相关的eslint扩展和插件。
```bash
yarn add -D eslint prettier
yarn add -D @typescript-eslint/parser @typescript-eslint/eslint-plugin
yarn add -D eslint-plugin-react
yarn add -D eslint-config-prettier eslint-plugin-prettier
```
依赖说明：
- @typescript-eslint/parser：将 TypeScript 转换为 ESTree，使 eslint 可以识别。
- @typescript-eslint/eslint-plugin：可以打开或关闭的规则列表。
- eslint-plugin-react：检测和规范React代码。
- eslint-config-prettier：解决ESLint中的样式规范和prettier中样式规范的冲突，以prettier的样式规范为准，使ESLint中的样式规范自动失效。
- eslint-plugin-prettier：将prettier作为ESLint规范来使用。

**prettier.config.js**如下：
```JavaScript
module.exports = {
  printWidth: 120, // 每行代码最大长度
  tabWidth: 2, //一个tab代表几个空格数，默认为80
  useTabs: false, //是否使用tab进行缩进，默认为false，表示用空格进行缩减
  semi: true, // 声明后带分号
  singleQuote: true, // 使用单引号
  jsxSingleQuote: true, // 使用单引号
  jsxBracketSameLine: true, // 启用jsx语法，> 放在末尾
  trailingComma: 'all',
};
```

**.eslintrc.js**如下：
```JavaScript
module.exports = {
  env: {
    //指定代码的运行环境
    browser: true,
    node: true,
  },
  settings: {
    //自动发现React的版本，从而进行规范react代码
    react: {
      pragma: 'React',
      version: 'detect',
    },
  },
  parser: '@typescript-eslint/parser',
  parserOptions: {
    //指定ESLint可以解析JSX语法
    ecmaVersion: 2019,
    sourceType: 'module',
    ecmaFeatures: {
      jsx: true,
    },
  },
  extends: [
    //使用推荐的React代码检测规范
    'plugin:react/recommended',
    'plugin:@typescript-eslint/recommended',
    'prettier/@typescript-eslint',
    'plugin:prettier/recommended',
  ],
  plugins: ['@typescript-eslint'],
  rules: {},
};
```

新增的extends的配置中：
- prettier/@typescript-eslint：使得@typescript-eslint中的样式规范失效，遵循prettier中的样式规范
- plugin:prettier/recommended：使用prettier中的样式规范，且如果使得ESLint会检测prettier的格式问题，同样将格式问题以error的形式抛出

### vscode插件

为了开发方便，我们常常在VSCode中集成一些辅助插件。如prettier、eslint，使得代码在保存或者代码变动的时候自动进行prettier、eslint --fix等。
在settings.json文件中修改eslint配置：
```JavaScript
{
  'eslint.enable': true, //是否开启vscode的eslint
  'eslint.autoFixOnSave': true, //是否在保存的时候自动fix eslint
  'eslint.options': {
    //指定vscode的eslint所处理的文件的后缀
    extensions: ['.js', '.ts', '.tsx'],
  },
  'eslint.validate': [
    //确定校验准则
    'javascript',
    'javascriptreact',
    {
      language: 'html',
      autoFix: true,
    },
    {
      language: 'typescript',
      autoFix: true,
    },
    {
      language: 'typescriptreact',
      autoFix: true,
    },
  ],
};
```


参考：
* [ESLint+Prettier代码规范实践](https://zhuanlan.zhihu.com/p/68026905)
* [使用ESLint+Prettier来统一前端代码风格](https://juejin.im/post/5b27a326e51d45588a7dac57)
* [在Typescript项目中，如何优雅的使用ESLint和Prettier](https://juejin.im/post/5d1d5fe96fb9a07eaf2bae29)

