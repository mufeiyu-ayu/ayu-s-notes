### 1.需求

1. 验证规则 rules
2. 组件的布局方式
3. 支持分组
4. 支持重置，提交
5. 表单项类型

### 1.验证规则

验证规则采用官网内置的校验规则参见 [async-validator ](https://github.com/yiminghe/async-validator)

```typescript
type BaseFieldConfig = {
  // ...
  rules?: FormItemRule[]
}
```

### 2.组件布局方式

```
type GroupType = 'default' | 'tab' | 'collapse'
```

### 3.是否分组

```typescript
interface FormProps {
  //...
  isGroup: boolean
}
```

### 4.重置提交

#### 5.表单类型
