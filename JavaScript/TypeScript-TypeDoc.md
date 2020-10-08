# TypeScript 文档化工具: typedoc

## 背景

最开始受到[使用react-docgen自动生成组件文档](https://aotu.io/notes/2020/07/27/how-to-use-react-docgen/index.html)的触发，思考到工作中是外包型多项目并行开发，比产品型项目要更加考虑人员和周期对项目的风险。接手技术管理后，了解到已有文档的项目是寥寥无几，所以强烈感觉需要建立项目文档规范。

目前考虑从下面两个方面考虑：

- 项目设计文档：从需求到项目开发的前期讨论资料、后期设计文档，重点包含需求拆分、难点业务设计等。
- 代码开发文档：即代码中的文档，如函数、类等的说明。

> 本文内容讨论的是第二点，即代码开发文档。

按照开发习惯来说，代码开发文档直接写成代码注释即可，但注释不方便浏览。所以如果能把注释从代码中提出来，生成带有目录、且能内部跳转HTML文件就太好了，如[iOS开发的UIApplicationDelegate](https://developer.apple.com/documentation/uikit/uiapplicationdelegate?language=objc)。

### 前期工作

从[react-docgen](https://github.com/reactjs/react-docgen)开始，接连了解到类似的工具有：

- [react-docgen](https://github.com/reactjs/react-docgen)

  - > react-docgen is a low level tool to extract information about react components. If you are searching for a more high level styleguide with nice interface try [react-styleguidist](https://github.com/styleguidist/react-styleguidist) or any of the other tools listed in the [wiki](https://github.com/reactjs/react-docgen/wiki).
  - [使用react-docgen自动生成组件文档](https://aotu.io/notes/2020/07/27/how-to-use-react-docgen/index.html) 显示要额外写propTypes和defaultProps，放弃！

- [react-styleguidist](https://github.com/styleguidist/react-styleguidist)

  - > React Styleguidist is a component development environment with hot reloaded dev server and a living style guide that you can share with your team. It lists component `propTypes` and shows live, editable usage examples based on Markdown files. Check out [**the demo style guide**](https://react-styleguidist.js.org/examples/basic/).
  - 同样要额外写propTypes，文档是是通过额外的markdown文件来解析的，更适合物料库的文档编写，工作量太大，放弃！

- [dumi](https://github.com/umijs/dumi)

  - > A Umi-based doc tool can assist you to develop libraries & write docs.
  - 和umijs是同一家族的，但两者之间没有内在直接关联。个人感觉同[react-styleguidist](https://github.com/styleguidist/react-styleguidist)，都是通过markdown来实现文档的，区别在于md文件存放位置不一样。这对开发人员来说，需要维护两套"代码"，是不可持续的行为，放弃！

- [docz](https://github.com/doczjs/docz)

  - > Docz makes it easy to write and publish beautiful interactive documentation for your code.
  - 同[react-styleguidist](https://github.com/styleguidist/react-styleguidist)，区别在于md文件换成了mdx文件，感觉功能上更强大一点。

- [jsdoc](https://github.com/jsdoc/jsdoc)

  - > An API documentation generator for JavaScript. 

  - 通过js代码注释生成文档，对现有项目影响小，支持目录和跳转，第一感觉是想要的，且 vs code 内置 jsdoc 的支持（在文件里 输入`/**`就会看到提示）。*可在块级注释中使用`@`进行快速关键字提示，也支持文件源码查看和行数定位，下载[Demo](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/10/08/Lx1gAb.zip)。*

  - 但不支持jsx、ts、tsx等TypeSrcipt后缀名相关的文件，仅对js，放弃！

- [tsdoc](https://github.com/Microsoft/tsdoc)

  - > A doc comment standard for the TypeScript language.

  -  [在线地址](https://microsoft.github.io/tsdoc/)，你会发现这是一个ts的文档标准，而不是tool，[有哪些参与呢？](https://github.com/Microsoft/tsdoc#whos-involved)。实际就是一个注释标准，然后可以生成HTML、DOM、Lines、AST、Emitter等格式。说实话，个人是非常欣赏这种做法的！加油~

- [TypeDoc](http://typedoc.org/)，来源于上面ts的参与者列表[participant list](https://github.com/Microsoft/tsdoc#whos-involved)

  - > TypeDoc converts comments in TypeScript source code into rendered HTML documentation or a JSON model. It is extensible and supports a variety of configurations. Available as a CLI or Node module.

  - 没错，这就是本文要重点说的，目前看来满足我的需求，还能支持主题自定义。下载[Demo](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/10/08/OXf51L.zip)



## [TypeDoc](http://typedoc.org/)

### 安装

```bash
// 二选一（我使用的方式2）
$ yarn add -D typedoc // in a project
$ yarn global add typedoc // as a global module

$ typedoc --version
TypeDoc 0.19.2
Using TypeScript 4.0.3 from /Users/xxx/.config/yarn/global/node_modules/typescript/lib
```

> *typedoc依赖与[tsc](https://www.typescriptlang.org/#installation).*

### 配置

支持命令式和配置式两种，命令式可通过`typedoc --help`查看帮助。

```bash
typedoc --out docs src
```

下面说明配置式，支持两种模式：

```json
// typedoc.json
{
    "mode": "file",
    "out": "docs"
}
```

```json
// tsconfig.json（推荐）
{
  "typedocOptions": {
    "name": "测试",
    "mode": "modules",
    "inputFiles": ["./src"],
    "out": "docs",
    // "theme": "minimal",
    "excludeExternals": true,
    "exclude": ["./src/.umi", "**/*.tsx", "**/*.less"]
  }
}
```

> 支持自写脚本配置，提供高级写法，参考[这里](https://typedoc.org/guides/installation/#node-module)。

其他[配置Options](https://typedoc.org/guides/options/)，重点关注里面的[Input Options](https://typedoc.org/guides/options/#input-options)，但怎么过滤掉less文件呢？？？

### 注释规范

> vs code 内置 jsdoc 的支持（在文件里输入`/**`就会看到提示），注释块内使用`@`进行Tags关键字快速提示。

支持块注释，内嵌下面插件：

- [Marked](https://github.com/chjj/marked)：支持markdown写法
- [HighlightJS](https://github.com/isagalaev/highlight.js)：代码高亮
- [Symbol References](https://typedoc.org/guides/link-resolution/)：自定义跳转

```typescript
/**
 * This comment _supports_ [Markdown](https://marked.js.org/) and Code blocks.
 *
 * ```
 * <my-custom-element>Highlight JS will auto detect the language</my-custom-element>
 * ```
 *
 * ```typescript
 * // Or you can specify the language explicitly
 * const instance = new Foo();
 * ```
 *
 * Standard links:
 * {@link Foo} or {@linkplain Foo} or [[Foo]]
 *
 * Code links: (Puts Foo inside <code> tags)
 * {@linkcode Foo} or [[`Foo`]]
 */
export class Bar implements Foo {}

/** More details */
interface Foo {
    member: boolean;
}
```

支持下面的Tags：

- [`@param `](https://typedoc.org/guides/doccomments/#param-param-name)
- [`@typeParam `](https://typedoc.org/guides/doccomments/#typeparam-param-name)：用于模板参数
- [`@return(s)`](https://typedoc.org/guides/doccomments/#returns)
- [`@event`](https://typedoc.org/guides/doccomments/#event)
- [`@hidden and @ignore`](https://typedoc.org/guides/doccomments/#hidden-and-ignore)
- [`@internal`](https://typedoc.org/guides/doccomments/#internal)
- [`@category`](https://typedoc.org/guides/doccomments/#category)

