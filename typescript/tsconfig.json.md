以在配置均为自身在项目中深入使用且研究体会的

### **配置文件解析**

```json
{
  "extends": "@ayu-mu/tsconfig/tsconfig.base.json", // 用来指定所要继承的配置文件。它可以是本地文件。
  "compilerOptions": {

    "target": "ESNext", // 编辑后产物对应的 js 版本
    "module":"ESNext",//module指定编译产物的模块格式，
    "lib": ["ESNext", "DOM", "DOM.Iterable"],// 告诉TypeScript 编译器你的代码会使用哪些环境的哪些版本的 JavaScript API，从而提供相应的类型检查和开发支持
    "baseUrl": ".", // 根目录
    "rootDir": ".", // 输入文件的根目录
    "outDir": "./dist",// 编译后的路径
    "paths": {
      "@/*": [
        "src/*" // 用于配置模块解析的别名
      ]
    },
    "noImplicitAny":"true",//禁止隐式推导出 any 类型
    "moduleResolution":"bundler"// moduleResolution 是 TypeScript 中用于配置模块解析策略的选项，它决定了编译器如何查找导入模块（bundler|classic|node|,,,,）
     // bundler=>当前代码会被各种打包工具处理，从而放宽加载规则。要求 module 为 es2015 或更高版本，
    "types": ["vite/client","node"], // 接受一个字符串数组作为值，用于指定需要包含的类型包名称，切会自动包含 package.json 中声明的依赖项的类型定义，主要影响那些不需要显式导入就能使用的类型（比如 "node" 和 "vite/client"）
    "declaration": true, // 开启生成声明文件的功能（默认false）
    "emitDeclarationOnly": true, // 只生成.d.ts文件，不生成 js 输出
    "outDir": "temp", // 输出目录
    "stripInternal": true, // 去除内置模块
    "useDefineForClassFields": "true", // 用于解决类字段类型不正确的问题,使用最先版现代的 js 标准行为，
    "noEmit": false // 是否禁止生成编译后的.js,.ts 建,
    "forceConsistentCasingInFileNames":"true", // 强制文件名大小写一致
    "skipLibCheck":true,// 运行跳过对.d.ts 的类型检查
    "esModuleInterop":true,//允许使用 import 导入 CommonJS 模块
    "strict":true,// 启用严格模式
    "removeComments":true, // 移除 TypeScript 脚本里面的注释，默认false
    "resolveJsonModule":true,// 是否允许导入 json 文件


  },
  "references": [{ "path": "./tsconfig.app.json" }, { "path": "./tsconfig.node.json" }] // 用于 建立项目间的依赖关系，适用于需要指定编译顺序或依赖的场景 , 适合在大型项目或 monorepo 中使用，因为它会告诉 TypeScript 编译器哪些项目之间存在依赖关系。编译时指定不同路径即可
}
```

### 我的配置

**tsconfig.base.json**

```json
{
  "compilerOptions": {
    // 目标语言的版本

    "target": "esnext",

    // TS需要引用的库，即声明文件，es5 默认引用dom、es5、scripthost

    "lib": ["DOM", "ESNext", "DOM.Iterable"],

    // 用于解析非绝对模块名的基本目录

    "baseUrl": ".",

    // 生成代码的模板标准

    "module": "ESNext",

    // 指定模块解析策略

    "moduleResolution": "Bundler",

    // 是否解析 JSON 模块

    "resolveJsonModule": true,

    "types": ["vite/client", "node"],

    "allowImportingTsExtensions": true,

    // 是否启动所有严格检查的总开关

    "strict": true,

    // 检查switch中是否含有case没有使用break跳出

    "noFallthroughCasesInSwitch": true,

    // 是否检查未使用的局部变量

    "noUnusedLocals": true,

    // 是否检查未使用的参数

    "noUnusedParameters": true,

    // 生成声明文件，开启后会自动生成声明文件

    "declaration": true,

    // 为声明文件生成sourceMap

    "declarationMap": true,

    // 只生成声明文件，而不会生成js文件

    "emitDeclarationOnly": true,

    // 发送错误时不输出任何文件

    "noEmitOnError": true,

    // 生成目标文件的sourceMap文件

    "sourceMap": true,

    // 是否允许从没有默认导出的模块中默认导入

    "allowSyntheticDefaultImports": true,

    // 是否将每个文件转换为单独的模块

    // "isolatedModules": true,

    // 允许export=导出，由import from 导入

    "esModuleInterop": true,

    // 是否跳过声明文件的类型检查

    "skipLibCheck": true
  },

  "exclude": ["node_modules", "dist"]
}
```

**tsconfig.json**

```json
/*

配置编译器选项和指定要包含或排除的文件。

*/

{
  "extends": "./tsconfig.base.json",

  "compilerOptions": {
    "target": "esnext", //目标语言的版本

    "useDefineForClassFields": true, //是否使用类字段初始化器语法

    "module": "ESNext", //指定生成代码的模板标准

    "sourceMap": true, //生成目标文件的sourceMap文件

    "skipLibCheck": true, //跳过库文件的类型检查。

    "jsx": "preserve", //JSX 代码的处理方式。

    /* Bundler mode */

    "moduleResolution": "Bundler", // 模块解析策略

    "allowSyntheticDefaultImports": true, // 允许从没有默认导出的模块中导入类型。

    "resolveJsonModule": true, // 是否解析 JSON 模块。

    "isolatedModules": true, // 是否将每个模块单独编译。

    /* Linting 是否启用严格类型检查、是否检查未使用的局部变量或参数等。*/

    "strict": false,

    "noUnusedLocals": false,

    "noUnusedParameters": false,

    "noFallthroughCasesInSwitch": false,

    "experimentalDecorators": false,

    "noImplicitAny": false,

    "suppressImplicitAnyIndexErrors": false,

    // "allowJs": true, // 是否允许编译器编译 JS 文件。

    "baseUrl": ".",

    "paths": {
      "@/*": ["./src/*"]
    },

    "lib": ["esnext", "dom", "dom.iterable", "scripthost"] // 指定要包含的库文件。
  },

  // 指定要包含或排除的文件或文件夹。

  "include": ["src/**/*.ts", "src/**/*.vue", "src/**/*.tsx"],

  "exclude": [
    "node_modules"

    // 忽略 react 文件检查
  ]
}
```

### eslint相关 rules

```js
export const rules = {
  '@typescript-eslint/no-explicit-any': 'error' //使用 any 就报错
}
```

### 生态包

#### 1.SimplyTyped:

- 是一个轻量级的 TypeScript 类型工具库 [2](https://stackoverflow.com/questions/9338709/what-is-dependent-typing)
- 安装方式：`npm install --save-dev simplytyped` [2](https://stackoverflow.com/questions/9338709/what-is-dependent-typing)
- 支持 Deno 环境使用 [2](https://stackoverflow.com/questions/9338709/what-is-dependent-typing)
- 持续维护和更新，有定期的版本发布 [3](https://www.reddit.com/r/Python/comments/qofmt3/is_there_any_language_that_is_as_similar_as/)

#### 2.utility-types:

- 提供了完整的内置类型声明 [2](https://stackoverflow.com/questions/9338709/what-is-dependent-typing)\_2
- 支持 npm 和 yarn 两种安装方式 [2](https://stackoverflow.com/questions/9338709/what-is-dependent-typing)\_2
- 被推荐作为开发依赖（devDependencies）安装 [3](https://www.reddit.com/r/Python/comments/qofmt3/is_there_any_language_that_is_as_similar_as/)\_2
- 在社区中使用广泛，文档完善 [4](https://github.com/andnp/SimplyTyped/releases)\_2

#### 3.[ts-toolbelt](https://millsp.github.io/ts-toolbelt/):

- 提供更多高级类型操作功能 [5](https://news.ycombinator.com/item?id=15384351)\_2
- 通过 `npm install ts-toolbelt --save` 安装 [4](https://github.com/andnp/SimplyTyped/releases)\_2
- 包含完整的类型声明文件 [2](https://stackoverflow.com/questions/9338709/what-is-dependent-typing)\_2
- 在现代 TypeScript 项目中应用广泛 [3](https://www.reddit.com/r/Python/comments/qofmt3/is_there_any_language_that_is_as_similar_as/)\_2

比较和选择建议：

- SimplyTyped 相对更轻量，适合简单项目 [2](https://stackoverflow.com/questions/9338709/what-is-dependent-typing)
- utility-types 和 ts-toolbelt 功能更全面，适合大型项目 [5](https://news.ycombinator.com/item?id=15384351)\_2
- 三个库都可以同时使用，互不冲突 [4](https://github.com/andnp/SimplyTyped/releases)\_2
- 选择哪个主要取决于项目需求和团队偏好 [3](https://www.reddit.com/r/Python/comments/qofmt3/is_there_any_language_that_is_as_similar_as/)\_2

所有这些包都是纯类型工具包，不会增加运行时负担，主要用于开发阶段的类型检查和类型推导。
