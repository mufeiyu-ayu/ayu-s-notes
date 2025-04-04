### 1.常用类型

1. **VxeTableInstance** --- vxetable 的实例对象类型
2. **VxeTableProps** --- 这是表格组件的属性接口，定义了你可以传递给 VxeTable 组件的所有属性。它是一个完整的属性对象类型。
3. **VxeTablePropTypes** --- 这是一个命名空间，包含了各种表格属性的具体类型定义。它包含了更细粒度的类型，如 FooterData（你在代码中使用的）等特定属性的类型。

### 2.踩坑

使用 vxetable 打印功能时需从 vxe-pc-ui 中引入VxePrint 组件并注册
