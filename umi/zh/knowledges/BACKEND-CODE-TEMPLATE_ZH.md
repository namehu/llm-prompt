## 代码编写模板
以下会列出在UMI框架下常用的代码模板。请你根据用户需求，选择合适的模板。严格按照团队编码规范编写。
### 项目结构
项目采用 `PNPM + MONOREPO` 方式管理。主体如下
```tree
.
├── components
│   ├── backend-pro
│   │   ├── src/components
│   │   ├── src/hooks
│   ├── utils
│   └── react-hooks
├── packages
│   ├── backend 后管项目
│   │   ├── config/routes/route.ts 路由配置文件
│   │   ├── src
│   │   │   └── components
│   │   │   ├── enums
│   │   │   └── pages
│   │   └── mock
├── scripts
├── tools
└── tsconfig.json
```

### backend-pro

#### components\backend-pro\src\index.ts

```ts
import { App } from 'antd'
import type { MessageInstance } from 'antd/es/message/interface'
import type { ModalStaticFunctions } from 'antd/es/modal/confirm'
import type { NotificationInstance } from 'antd/es/notification/interface'

let message: MessageInstance
let notification: NotificationInstance
let modal: Omit<ModalStaticFunctions, 'warn'>

export default () => {
  // 初始化 antd 静态方法实例
  return null
}

export { message, notification, modal }
```

#### components\backend-pro\src\hooks\useExport.ts

```typescript
import { useState } from 'react'

/**
 * 导出hook
 */
export default function useExport(url = '') {
  const [loading, setLoading] = useState(false)

  /**
   * 导出表格
   * @param params
   * @returns
   */
  async function exportTable(params: Record<string, any> = {}) {
    // ... 实现逻辑：处理 Authorization，发起 fetch 请求，转换 Blob 并触发下载
  }

  return [exportTable, loading] as const
}
```

#### components\backend-pro\src\hooks\useProTableRequest.ts

```ts
import { ActionType, ProFormInstance, RequestData } from '@ant-design/pro-components'
import { SortOrder } from 'antd/lib/table/interface'
import { useCallback, useLayoutEffect, useRef } from 'react'

type Params<U> = U & {
  pageSize?: number
  current?: number
  keyword?: string
}

type Sort = Record<string, SortOrder>

type Filter = Record<string, React.ReactText[] | null>

type Fn<U, T> = (
  params: any,
  sort: Sort,
  filter: Filter
) => Promise<{ code?: number; msg?: string; data?: { list?: T[]; total?: number } }>

export interface IUseProTableRequestOption<T, U = T> {
  /**
   * 导出接口url
   */
  exportUrl?: string

  /**
   * 是否缓存查询参数
   * 会将查询参数缓存保存在URL上
   * @default true
   */
  filterCache?: boolean
  /**
   * 格式化参数
   *
   * 前置处理请求参数。如果你需要传递给导出时。这会很有用
   * @param params 将要传递给接口的参数
   */
  paramsFormat?(params: any): any
  /**
   * 格式化数据
   * 你可以对返回的数据做一些处理
   */
  dataFormat?(data: T[]): U[]
}

/**
 * antd pro table请求封装钩子
 * @param fn
 * @param option
 * @returns
 */
export default function useProTableRequest<T, U extends Record<string, any> = {}>(
  fn: Fn<U, T>,
  option: IUseProTableRequestOption<T> = {}
) {
  // Table action 的引用，便于自定义触发
  const actionRef = useRef<ActionType>()
  const formRef = useRef<ProFormInstance>()
  // 缓存请求参数
  const requestParams = useRef<Record<string, any>>({})
  // 数据缓存参数
  const dataSourceRef = useRef<T[]>([])

  // 集成导出
  // ... useExport hook usage

  // 表格请求
  const tableRequst = useCallback(async (params: Params<U>, sort: Sort, filter: Filter) => {
     // ... 实现逻辑：处理分页参数，调用 fn 获取数据，处理 filterCache，返回 ProTable 所需数据格式
     return { data: [], success: true, total: 0 } as Partial<RequestData<T>>
  }, [])

  // ... 恢复搜索表单和分页信息的 Effect

  return {
    tableProps: {
      actionRef,
      formRef,
      request: tableRequst,
      rowKey: 'id'
    },
    /**
     * 表格操作Ref
     * @deprecated 建议使用tableProps
     */
    actionRef,
    /**
     * 表格筛选表单Ref
     * @deprecated 建议使用tableProps
     */
    formRef,
    /**
     * 表格请求
     * @deprecated 建议使用tableProps
     */
    request: tableRequst,
    /**
     * 表格query参数
     */
    params: requestParams,
    /**
     * 表格数据缓存
     */
    dataSource: dataSourceRef,
    /**
     * 导出
     * @param params
     * @returns
     */
    exportTable: (params?: Record<string, any>) => {
      // ... implementation
    },
    /**
     * 导出loading
     */
    exportLoading: false
  }
}
```

#### components\backend-pro\src\hooks\useProTableForm.ts

```ts
import { ModalFormProps } from '@ant-design/pro-components'
import { useEffect, useMemo, useRef, useState } from 'react'

export interface IUseProTableFormOption<DataType> {
  title?: (data?: DataType) => string
  modalProps?: Partial<ModalFormProps>
}

export default function useProTableForm<DataType = Record<string, any>>(option: IUseProTableFormOption<DataType> = {}) {
  // ... state definitions: open, editData, form instance

  // ... memo: modalProps configuration

  // ... effect: clean data on close

  /**
   * 设置显示弹窗并设置数据
   */
  function setShowModal(editData?: DataType) {
     // ... implementation
  }

  return {
    // 组合props。 该props适合antd pro Form
    modalProps: {},
    editData: undefined,
    /**
     * 设置编辑数据
     * @deprecated 使用setShowModal
     */
    setEditData: () => {},
    /**
     * 打开弹窗
     */
    setOpen: () => {},
    /**
     * 打开弹窗并可以设置编辑数据
     */
    setShowModal
  }
}
```

#### components\backend-pro\src\hooks\useRouterParams.ts

```ts
import { useMemo, useRef } from 'react'

export type RawParams = Record<string, string>
export type DefaultParams = Partial<RawParams>

/**
 * 获取路由参数
 * @param option.parseFn 解析函数。
 */
export function useRouterParams<T = DefaultParams>(
  option: {
    parseFn?: (data: RawParams) => T
  } = {}
) {
  // ... implementation: 解析 window.location.search 或 hash，支持 parseFn
  const params = {} as T
  return [params] as const
}
```

#### components\backend-pro\src\components\table\OperationsColumns.tsx

```ts
import { Space } from 'antd'
import { FC, memo, ReactNode } from 'react'

export type TOperationsColumnOperation =
  | {
      /** 编辑或者删除 */
      id: 'edit' | 'del'
      /** 文本 */
      text?: ReactNode
      /** 是否显示 */
      show?: boolean
      /** 点击事件 */
      onClick?: () => any
    }
  | {
      /** 唯一id */
      id: string
      /** 文本 */
      text?: ReactNode
      /** 是否显示 */
      show?: boolean
      /** 点击事件 */
      onClick?: () => any
    }

export interface IOperationsColumnsProps {
  /**
   * 操作项
   * 默认是编辑(edit)和删除(del)。你可以自行扩展和声明顺序
   *
   */
  operations?: TOperationsColumnOperation[]
}

/**
 * 表格操作列
 * @param props
 * @returns
 */
const Component: FC<IOperationsColumnsProps> = (props) => {
  // ... implementation: 过滤 operations，处理点击事件（删除弹窗确认）

  return (
    <Space>
      {/* ... 渲染操作链接或文本 */}
    </Space>
  )
}

Component.displayName = 'OperationsColumns'

const OperationsColumns = memo(Component)
export default OperationsColumns
```

#### components\backend-pro\src\components\table\StatusSwitchColumn.tsx

```ts
import { FC, memo } from 'react'
import { Switch, SwitchProps } from 'antd'
export interface IStatusSwitchColumnProps extends SwitchProps {
  onSwitch(checked: boolean): Promise<any>
}

/**
 * 状态切换列
 */
const Component: FC<IStatusSwitchColumnProps> = (props) => {
  // ... implementation: 处理 switch 变更，包含 loading 状态和 try/catch 错误处理

  return <Switch {...props} />
}

Component.displayName = 'StatusSwitchColumn'

const StatusSwitchColumn = memo(Component)
export default StatusSwitchColumn
```

### react-hooks

#### components\react-hooks\src\useSuperLock.ts

```ts
import { useState, useRef } from 'react'

/**
 * 超级锁钩子。未运行完毕锁。500毫秒运行一次锁。运行成功500毫秒后才能运行锁。
 *
 * @param setLoading
 * @param fun
 */
export function useSuperLock<T extends (...args: any) => any>(fun: T, delay = 500) {
  // ... implementation: 锁逻辑控制
  const fn: any = async (...args: Parameters<T>) => {
      // ... logic
  }
  const lock = false
  return [fn, lock] as const
}
```

### 代码模板

提供不同类型[页面模板](./templates/README.md)。其中下面列明部分按需使用。如果不需要可以不在代码中编写

#### [umi-max-function-page.yml umi 页面模板](./templates//umi-max-function-page.yml)

#### [umi-max-function-form-page.yml 表单页面模板](./templates/umi-max-function-form-page.yml)

#### [umi-max-function-detail-page.yml 详情页面模板](./templates//umi-max-function-detail-page.yml)

### 代码要求

输出代码格式要求：

- 页面模板主体格式尤其是jsx部分必须保留布局格式
- 如果是页面代码。需要在`config/routes`中正确配置路由并在`pages`目录下创建对应的页面组件代码。
- 枚举文件/mock接口 等单独编写
- 使用代码块包裹，标明语言（tsx, ts）
- 加必要注释说明关键逻辑
- 枚举文件固定路径。 **~/enums/E{NAME}.ts**。一个枚举一个文件。并且需要写好 M{NAME} 与 O{NAME} 衍生对象。示例如下

```ts
/**
 * 短信code场景值
 */
export const ECodeScene = {
  /** 注册 */
  REG: 'REG',
  /** 忘记密码 */
  FORGOT: 'FORGOT'
} as const 

export type ECodeScene = (typeof ECodeScene)[keyof typeof ECodeScene] // 类型

export const MCodeScene = {
  [ECodeScene.REG]: '注册',
  [ECodeScene.FORGOT]: '忘记密码'
}

export const OCodeScene = [
  { value: ECodeScene.REG, label: '注册' },
  { value: ECodeScene.FORGOT, label: '忘记密码' }
]
```

- 如有多文件协同，明确写好文件路径和引用关系。相对路径alias是 `~` 符号
