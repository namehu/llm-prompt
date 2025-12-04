# 模板

配合插件快捷创建模板代码

## 规定

### 模板约定

- createStyles 用于创建样式
- toolBarRender 用于自定义工具栏。比如导出/下载按钮。
- ModalForm 用于新增编辑操作。

- [:=xxx:]格式占位是模板变量。在生成模板的过程中。可以通过模板变量来进行更加细致的控制。支持的变量有:
  - CamelCaseName 组件驼峰命名
  - PascalName 组件帕斯卡命名
  - KebabCaseName 组件横杠线命名
  - UnderlineCase 下划线命名
  - dirname 组件目录名
