### 1.打包发布的细节

当基于 mor 架构打包发包时，可能会引用 package 目录其他包或者第三方包的依赖，当我们发布这个包然后在项目中安装使用这个包的时候，会将这个包的 package.json 的 dependencies依赖安装到.pnpm 目录(使用 pnpm 安装时)
**举例**

```vue
<script lang="ts" setup>
import { AyuForm, type FormProps } from '@ayu-mu/core-common'

import { ref, onMounted, reactive } from 'vue'

const formRef = ref<InstanceType<typeof AyuForm>>()

const formProps = reactive<FormProps>({
  fieldConfig: [
    {
      label: '用户名',

      field: 'username',

      defaultValue: 'ayu',

      type: 'input',

      group: '基础信息',

      colSize: 12,

      rules: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
      componentProps: {
        disabled: true,

        validateEvent: true,

        placeholder: 'hello world'
      }
    }
  ],

  isGroup: true,

  isExpand: false,

  groupType: 'tab',

  tabType: 'border-card',

  maxPaneHeight: '200px',

  onSubmit: async (params: any) => {
    console.log('submit111111!', params)
  }
})
</script>

<template>
  <div class="w-screen h-screen flex justify-center items-center">
    <div class="w-[800px]">
      <AyuForm ref="formRef" v-bind="formProps"></AyuForm>
    </div>
  </div>
</template>

<style lang="scss" scoped></style>
```

componentProps的类型并不是在'@ayu-mu/core-common'包里定义的，而是在'@ayu-mu/model'包中定义的，因此查找规则是先查 node_modules=>'@ayu-mu/core-common'=>.pnpm=>'@ayu-mu/model',这个是生效的，需要注意

### 2.如何构建自己的包

vite 是这样描述的当你开发面向浏览器的库时，你可能会将大部分时间花在该库的测试/演示页面上。在 Vite 中你可以使用 `index.html` 获得如丝般顺滑的开发体验。当这个库要进行发布构建时，请使用 [`build.lib` 配置项](https://cn.vite.dev/config/build-options.html#build-lib)，以确保将那些你不想打包进库的依赖进行外部化处理，例如 `vue` 或 `react`：

```js
import { dirname, resolve } from 'node:path'
import { fileURLToPath } from 'node:url'
import { defineConfig } from 'vite'

const __dirname = dirname(fileURLToPath(import.meta.url))

export default defineConfig({
  build: {
    lib: {
      entry: resolve(__dirname, 'lib/main.js'),
      name: 'MyLib', //打包后的名字
      // 将添加适当的扩展名后缀
      fileName: 'my-lib'
    },
    rollupOptions: {
      // 确保外部化处理那些
      // 你不想打包进库的依赖
      external: ['vue'],
      output: {
        // 在 UMD 构建模式下为这些外部化的依赖 umd 格式支持 cdn 引入
        // 提供一个全局变量
        globals: {
          vue: 'Vue'
        }
      }
    }
  }
})
```

### 3.minify

- **类型：** `boolean | 'terser' | 'esbuild'`
- **默认：** 客户端构建默认为`'esbuild'`，SSR构建默认为 `false`

设置为 `false` 可以禁用最小化混淆，或是用来指定使用哪种混淆器。默认为 [Esbuild](https://github.com/evanw/esbuild)，它比 terser 快 20-40 倍，压缩率只差 1%-2%。[Benchmarks](https://github.com/privatenumber/minification-benchmarks)
注意，在 lib 模式下使用 `'es'` 时，`build.minify` 选项不会缩减空格，因为会移除掉 pure 标注，导致破坏 tree-shaking。当设置为 `'terser'` 时必须先安装 Terser。
