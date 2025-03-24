```typescript
import type { FormRules, FormItemRule } from 'element-plus'

// 完整的验证规则类型示例
const rulesExample: FormRules = {
  // 基础验证
  name: [
    { required: true, message: '请输入名称', trigger: 'blur' },
    { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' }
  ],

  // 正则验证
  phone: [
    { required: true, message: '请输入手机号码', trigger: 'blur' },
    {
      pattern: /^1[3-9]\d{9}$/,
      message: '请输入正确的手机号码',
      trigger: 'blur'
    }
  ],

  // 自定义验证函数
  email: [
    { required: true, message: '请输入邮箱', trigger: 'blur' },
    { type: 'email', message: '请输入正确的邮箱格式', trigger: 'blur' },
    {
      validator: (rule: FormItemRule, value: string, callback: (error?: Error) => void) => {
        if (value === '') {
          callback(new Error('请输入邮箱'))
        } else {
          callback()
        }
      },
      trigger: 'blur'
    }
  ],

  // 数字验证
  age: [
    { required: true, message: '请输入年龄', trigger: 'blur' },
    { type: 'number', message: '年龄必须为数字', trigger: 'blur' },
    {
      type: 'number',
      min: 0,
      max: 120,
      message: '年龄必须在0-120之间',
      trigger: 'blur'
    }
  ],

  // 数组验证
  hobbies: [
    {
      type: 'array',
      required: true,
      message: '请至少选择一个爱好',
      trigger: 'change'
    },
    {
      type: 'array',
      min: 1,
      max: 3,
      message: '请选择1-3个爱好',
      trigger: 'change'
    }
  ],

  // 对象验证
  address: [
    { required: true, message: '请输入地址信息', trigger: 'blur' },
    {
      type: 'object',
      fields: {
        province: { required: true, message: '请选择省份' },
        city: { required: true, message: '请选择城市' }
      }
    }
  ]
}

// 完整的类型定义
export interface FieldConfig {
  label: string
  field: string
  type: string
  group?: string
  rules?: FormItemRule[] // 单个字段的验证规则
}

export interface FormProps {
  fieldConfig: FieldConfig[]
  layout?: 'horizontal' | 'vertical' | 'inline'
  isGroup?: boolean
  rules?: FormRules // 整个表单的验证规则
}

// FormItemRule 的完整类型（参考）
interface FormItemRule {
  type?: 'string' | 'number' | 'boolean' | 'array' | 'object' | 'email' | 'url' | 'date'
  required?: boolean
  message?: string
  trigger?: 'blur' | 'change' | ['blur', 'change']
  min?: number
  max?: number
  length?: number
  pattern?: RegExp
  validator?: (rule: FormItemRule, value: any, callback: (error?: Error) => void) => void
  fields?: Record<string, FormItemRule>
}
```
