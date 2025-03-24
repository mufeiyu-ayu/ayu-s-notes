当给变量声明**any**类型时，代表 ts 不会再对这个变量做任何类型检测 ，当其他类型的变量与之交互时均为 any

```ts
let a: any = 1
let b: number = 2
let total = a + b // total:any
//均不报错
```

`unknown`类型跟`any`类型的不同之处在于，它不能直接使用。主要有以下几个限制

`unknown`类型的变量，不能直接赋值给其他类型的变量（除了`any`类型和`unknown`类型）。

`unknown`类型变量能够进行的运算是有限的，只能进行比较运算（运算符`==`、`===`、`!=`、`!==`、`||`、`&&`、`?`）、取反运算（运算符`!`）、`typeof`运算符和`instanceof`运算符这几种，其他运算都会报错。

```ts
let v: unknown = 123
let v1: boolean = v // 报错
let v2: number = v // 报错
```

**那么如何使用 unknown 类型呢？**
**答案**：只有经过“类型缩小”，`unknown`类型变量才可以使用。所谓“类型缩小”，就是缩小`unknown`变量的类型范围，确保不会出错

```ts
let v: unknown = 123

let b: number = 234
//类型缩写
if (typeof v === 'number') {
  b = v
}
```
