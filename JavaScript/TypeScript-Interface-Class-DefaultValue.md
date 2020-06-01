# TypeScript使用Interface与Class的默认值细节

最近编写Modal，犯下一个错误：把接口返回的数据，直接丢到数据模型中：
![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/06/01/ustgME.png)

```typescript
address: new Array(5).fill(new Address()),
```
然后循环address显示时，每个属性居然都是undefined
![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/06/01/py6dyI.png)

才发现Modal里那种写法不是默认值，而是定义的字面量类型（Literal Type）：
```typescript
type A = 38;
const c: A = 38; // ok
// const c: A = 28; // error

const g: 'github' = 'github'; // ok
// const g: 'github' = 'pithub'; // error
const gg: 1 = 1;
```

同样，在interface和class里
```typescript
export interface ITest {
  a: 28; // ok，定义字面量类型
  // b = 28; // error，interface里不支持=
}

const a: ITest = { a: 28 };
a.a = 28; // ok
a.a = 38; // error

export class CTest {
  a: 28;  // ok，定义字面量类型
  b = 28; // ok，自动推导是number类型，且默认值是28
}

let b: CTest = { a: 38, b: 38 };
b.a = 48; // error
b.b = 48; // ok
```

所以以后，切记不要偷懒。
- interface就使用string、number等基础类型
- class使用=去设置默认值，让其自动推导类型