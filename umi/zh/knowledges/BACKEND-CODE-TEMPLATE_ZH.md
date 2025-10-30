## 代码模板

以下会列出在UMI框架下常用的代码模板。请你根据用户需求，选择合适的模板。严格按照团队编码规范编写。

### 项目结构

项目采用 `PNPM + MONOREPO` 方式管理。主体如下

```tree
.
├── components
│   ├── backend-pro
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

#### backend-pro

- components\backend-pro\src\index.ts

```ts
import { App } from 'antd'
import type { MessageInstance } from 'antd/es/message/interface'
import type { ModalStaticFunctions } from 'antd/es/modal/confirm'
import type { NotificationInstance } from 'antd/es/notification/interface'

let message: MessageInstance
let notification: NotificationInstance
let modal: Omit<ModalStaticFunctions, 'warn'>

export default () => {
  const staticFunction = App.useApp()
  message = staticFunction.message
  modal = staticFunction.modal
  notification = staticFunction.notification
  return null
}

export { message, notification, modal }
```

- components\react-hooks\src\useSuperLock.ts

```ts
import { useState, useRef } from 'react'

/**
 * 超级锁钩子。未运行完毕锁。500毫秒运行一次锁。运行成功500毫秒后才能运行锁。
 *
 * @param setLoading
 * @param fun
 */
export function useSuperLock<T extends (...args: any) => any>(fun: T, delay = 500) {
  const [lock, setLock] = useState(false)
  const lastDate = useRef<Date>()

  const fn: any = async (...args: Parameters<T>) => {
    if (lock) {
      return
    }

    const nowDate = new Date()
    if (lastDate.current && nowDate.getTime() - lastDate.current.getTime() <= delay) {
      return
    }

    lastDate.current = nowDate
    setLock(true)

    let returnValue: any
    try {
      returnValue = await fun.apply(this, args)
    } catch (error) {
      setLock(false)
      throw error
    }

    setTimeout(() => {
      setLock(false)
    }, delay)

    return returnValue
  }
  return [fn, lock] as const
}
```

- components\backend-pro\src\hooks\useExport.ts

```typescript
import { useState } from 'react'
import { getAuthorization } from '../utils/authorization'

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
    if (!url) {
      return
    }
    const Authorization = getAuthorization()
    try {
      const headers = new Headers()
      setLoading(true)
      if (Authorization) {
        headers.set('Authorization', Authorization)
      }

      const fetchUrl = jointQuery(url, { ...params, pageNum: 1, pageSize: undefined })

      const option: any = { method: 'GET', headers, responseType: 'blob' }
      const res = await fetch(fetchUrl, option)
      const blobData = await res.blob()

      const blob = new Blob([blobData], { type: 'application/vnd.ms-excel; charset=UTF-8' })
      // 创建下载的链接
      const downloadElement = document.createElement('a')
      const href = window.URL.createObjectURL(blob)
      downloadElement.href = href

      const contentDisposition = res.headers!.get('content-disposition') || ''
      const [_, fileName = ''] = contentDisposition.split('filename=')
      downloadElement.download = decodeURI(fileName) || '' // 下载后文件名
      document.body.appendChild(downloadElement)
      downloadElement.click() // 点击下载
      document.body.removeChild(downloadElement) // 下载完成移除元素
      window.URL.revokeObjectURL(href) // 释放掉blob对象
    } catch (error) {
      console.error(`导出失败`, error)
    }
    setLoading(false)
  }

  return [exportTable, loading] as const
}

function jointQuery(url: string, params: { [i: string]: any } = {}) {
  // 是否携带query
  const query = Object.keys(params)
    .filter((key) => ![undefined, null].includes(params[key])) // 排除掉无效值
    .map((key) => `${encodeURIComponent(key)}=${encodeURIComponent(params[key])}`)
    .join('&')

  if (!query) {
    return url
  }
  return url + (url.search(/\?/) === -1 ? '?' : '&') + query
}
```

- components\backend-pro\src\hooks\useProTableRequest.ts

```ts
import { ActionType, ProFormInstance, RequestData } from '@ant-design/pro-components'
import { SortOrder } from 'antd/lib/table/interface'
import { useCallback, useLayoutEffect, useRef } from 'react'
import useExport from './useExport'

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
  const { filterCache = false, dataFormat } = option

  // Table action 的引用，便于自定义触发
  const actionRef = useRef<ActionType>()

  const formRef = useRef<ProFormInstance>()

  // 缓存请求参数
  const requestParams = useRef<Record<string, any>>({})
  // 数据缓存参数
  const dataSourceRef = useRef<T[]>([])

  // 集成导出
  const [exportTable, exportLoading] = useExport(option.exportUrl)

  // 表格请求
  const tableRequst = useCallback(async (params: Params<U>, sort: Sort, filter: Filter) => {
    const { current, ...rest } = params as Record<string, any>
    let newParams: any = { ...rest, pageNum: current ?? params?.pageNum ?? 1 }

    // 重置数据
    let total = 0
    let data: T[] = []

    requestParams.current = option.paramsFormat ? option.paramsFormat(newParams) : newParams

    try {
      // 参数长度过长不处理
      if (JSON.stringify(requestParams.current).length < 1000) {
        const res = await fn(requestParams.current, sort, filter)
        // 如果当前列表为空并且pageNum不为1.则重新发起请求
        if (!res.data?.list?.length && requestParams.current.pageNum !== 1) {
          setTimeout(() => {
            actionRef.current?.reload(true)
          })
        }
        const { list = [] as T[] } = res.data || {}
        total = res.data?.total || 0
        data = dataFormat ? dataFormat(list) : list
        // 修改路由缓存筛选参数
        // oxlint-disable-next-line no-unused-expressions
        filterCache && updateSearchParams({ filter: JSON.stringify(requestParams.current) })

        dataSourceRef.current = data
      }
    } catch (error) {
      // eslint-disable-next-line no-console
      console.error(error)
    }

    return { data, success: true, total } as Partial<RequestData<T>>
  }, [])

  useLayoutEffect(() => {
    const value = getFilterParams()
    if (value) {
      formRef.current?.setFieldsValue({ ...value }) // 恢复搜索表单
      actionRef.current?.setPageInfo?.({ current: value.current ?? value.pageNum ?? 1, pageSize: value.pageSize }) // 恢复分页信息
    }
  }, [])

  function getFilterParams() {
    if (filterCache) {
      const search = getSearchParams<{ filter?: string }>()
      if (search.filter) {
        try {
          return JSON.parse(search.filter) ?? {}
        } catch (error) {}
      }
    }
    return null
  }

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
      return exportTable({ ...requestParams.current, ...params })
    },
    /**
     * 导出loading
     */
    exportLoading
  }
}

/**
 * 获取浏览器 URL 中的 search 参数
 * @template T - 返回的参数对象类型，默认为 Record<string, string>
 * @returns {T} 包含所有 search 参数的对象
 */
function getSearchParams<T = Record<string, string>>(): T {
  // 处理普通 URL 和 hash 模式的路由
  let search = window.location.search
  const hashIndex = window.location.href.indexOf('#')

  // 如果是 hash 路由且 search 参数在 hash 后面
  if (hashIndex !== -1) {
    const hashPart = window.location.href.slice(hashIndex)
    const hashSearchIndex = hashPart.indexOf('?')

    if (hashSearchIndex !== -1) {
      search = hashPart.slice(hashSearchIndex)
    }
  }

  const params = new URLSearchParams(search)
  const result = {} as T

  // 将 URLSearchParams 转换为泛型对象
  for (const [key, value] of params.entries()) {
    ;(result as Record<string, string>)[key] = value
  }

  return result
}

/**
 * 更新 URL 的 search 参数（兼容 hash 路由模式）
 * @param params - 要更新的参数对象，值为 null 或 undefined 时删除该参数
 */
function updateSearchParams(params: Record<string, string | number | boolean | null | undefined>): void {
  const url = new URL(window.location.href)
  const hash = url.hash

  let hashPath = ''
  let hashSearchParams: URLSearchParams | null = null

  // 分解 hash 部分
  if (hash) {
    const [path, search] = hash.split('?', 2)
    hashPath = path
    hashSearchParams = new URLSearchParams(search || '')
  }

  // 判断参数是否在 hash 中
  // const isHashSearch = hash.includes('?')
  const isHashSearch = !!hashPath
  const currentSearchParams = isHashSearch ? hashSearchParams : url.searchParams

  if (!currentSearchParams) {
    return
  }

  // 合并参数（处理值类型和删除逻辑）
  Object.entries(params).forEach(([key, value]) => {
    if (value === null) {
      currentSearchParams.delete(key)
    } else {
      currentSearchParams.set(key, String(value))
    }
  })

  // 重新构建 URL
  if (isHashSearch && hashSearchParams) {
    const newHash = hashPath + (hashSearchParams.toString() ? `?${hashSearchParams.toString()}` : '')
    url.hash = newHash
  } else {
    url.search = currentSearchParams.toString()
  }

  // 更新 URL 而不刷新页面
  window.history.replaceState(null, '', url)
}
```

- components\backend-pro\src\hooks\useProTableForm.ts

```ts
import { ModalFormProps } from '@ant-design/pro-components'
import { useForm } from 'antd/lib/form/Form'
import { useEffect, useMemo, useRef, useState } from 'react'

export interface IUseProTableFormOption<DataType> {
  title?: (data?: DataType) => string

  modalProps?: Partial<ModalFormProps>
}

export default function useProTableForm<DataType = Record<string, any>>(option: IUseProTableFormOption<DataType> = {}) {
  const { title = (data) => (data ? '编辑' : '新增') } = option

  const [open, setOpen] = useState(false)
  const [editData, setEditData] = useState<DataType>()
  const [form] = useForm()
  const initedRef = useRef(false) // [fix Instance created by useForm is not connect to any Form element. Forget to pass form prop](https://github.com/ant-design/ant-design/issues/21543)

  const modalProps = useMemo(() => {
    return {
      layout: 'horizontal',
      labelCol: { span: 4 },
      wrapperCol: { span: 12 },
      ...option.modalProps,
      title: title(editData),
      open,
      form,
      onVisibleChange: (value) => setOpen(value)
    } as Omit<ModalFormProps, 'onFinish' | 'title'>
  }, [editData, open, form, title, option.modalProps])

  // 弹窗关闭清除数据
  useEffect(() => {
    if (!open && initedRef.current) {
      setEditData(undefined)
      // FIXED: 设置延时清空。防止弹窗里面存在request组件导致发出请求
      setTimeout(() => {
        form.resetFields()
      }, 300)
    }
    initedRef.current = true
  }, [open])

  /**
   * 设置显示弹窗并设置数据
   */
  function setShowModal(editData?: DataType) {
    if (editData) {
      setEditData(editData)
      form.setFieldsValue(editData)
    }
    setOpen(true)
  }

  return {
    // 组合props。 该props适合antd pro Form
    modalProps,
    editData,
    /**
     * 设置编辑数据
     * @deprecated 使用setShowModal
     */
    setEditData,
    /**
     * 打开弹窗
     */
    setOpen,
    /**
     * 打开弹窗并可以设置编辑数据
     */
    setShowModal
  }
}
```

- components\backend-pro\src\components\table\OperationsColumns.tsx

```ts
import { Space } from 'antd'
import { FC, memo, ReactNode, useMemo } from 'react'
import { createStyles } from 'antd-style'
import { modal } from '../../index'

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

const useStyles = createStyles(({ token }) => ({
  operationsColumnsStyle: {
    '& a': {
      whiteSpace: 'nowrap'
    }
  },
  item: {
    color: token.colorLink,
    whiteSpace: 'nowrap',
    cursor: 'pointer'
  }
}))

/**
 * 表格操作列
 * @param props
 * @returns
 */
const Component: FC<IOperationsColumnsProps> = (props) => {
  const { operations } = props

  const { styles } = useStyles()

  const _operations = useMemo(
    () =>
      (operations || [])
        .filter((item) => item.show !== false)
        .map((item) => {
          let { id, text } = item
          text = text || { edit: '编辑', del: '删除' }[id] || id
          return { ...item, text }
        }),
    [operations]
  )

  function handleClick({ id, onClick }: TOperationsColumnOperation) {
    if (id === 'del') {
      modal.confirm({ title: '确定删除?', onOk: onClick })
    } else {
      onClick?.()
    }
  }

  return (
    <Space className={styles.operationsColumnsStyle}>
      {_operations.map((ops) => {
        const { id, text } = ops
        if (typeof text === 'string') {
          return (
            <a key={id} className={styles.item} onClick={() => handleClick(ops)}>
              {text}
            </a>
          )
        }
        return (
          <span key={id} className={styles.item} onClick={() => handleClick(ops)}>
            {text}
          </span>
        )
      })}
    </Space>
  )
}

Component.displayName = 'OperationsColumns'

const OperationsColumns = memo(Component)
export default OperationsColumns
```

- components\backend-pro\src\components\table\StatusSwitchColumn.tsx

```ts
import { FC, memo, useState } from 'react'
import { Switch, SwitchProps } from 'antd'
export interface IStatusSwitchColumnProps extends SwitchProps {
  onSwitch(checked: boolean): Promise<any>
}

/**
 * 状态切换列
 */
const Component: FC<IStatusSwitchColumnProps> = (props) => {
  // oxlint-disable-next-line no-unused-vars
  const { onChange, onSwitch, ...rest } = props
  const [loading, setLoading] = useState(false)

  const handleChange = async (checked: boolean) => {
    setLoading(true)
    try {
      await onSwitch(checked)
    } catch (error) {
      // oxlint-disable-next-line no-console
      console.error(error)
    }
    setLoading(false)
  }

  return <Switch {...rest} loading={loading} onChange={handleChange} />
}

Component.displayName = 'StatusSwitchColumn'

const StatusSwitchColumn = memo(Component)
export default StatusSwitchColumn
```

- umi-max-function-page.yml umi 页面模板

```yml
name: '@wmeimob/umi-max-function-page'
description: |
  Umi4 函数式页面模板，其中
  文件中有诸如[:=xxx:]的字符。这是模板变量。在生成模板的过程中。你可以通过模板变量来进行更加细致的控制。支持的变量有: CamelCaseName 组件驼峰命名;PascalName 组件帕斯卡命名;KebabCaseName 组件横杠线命名;UnderlineCase 下划线命名;dirname 组件目录名
tpl:
  index.tsx: >
    import { FC, memo, useState } from 'react'\n
    import { Button } from 'antd'\n
    import { history } from '@umijs/max'\n
    import { createStyles } from 'antd-style'\n
    import { PageContainer, ProColumns, ProTable, ModalForm, ProFormText } from
    '@ant-design/pro-components'\n
    import useProTableRequest from
    '@wmeimob/backend-pro/src/hooks/useProTableRequest'\n
    import OperationsColumns from
    '@wmeimob/backend-pro/src/components/table/operationsColumns'\n
    import { useSuperLock } from '@wmeimob/react-hooks/src/useSuperLock'\n
    import useProTableForm from '@wmeimob/backend-pro/src/hooks/useProTableForm'\n
    import { message } from '@wmeimob/backend-pro'\n
    import { api } from '~/request'\n\n
    const useStyles = createStyles(() => ({
      [:=CamelCaseName:]Style: { }
    }))\n\n
    const Component: FC = () => {
      const { styles } = useStyles()
      const columns: ProColumns[] = [
        // 搜索区域
        {
          title: '列1',
          dataIndex: 'id',
          fieldProps: {
            placeholder: '请输入'
          }
        },
        // 表格区域
        { title: '列1', dataIndex: 'id', hideInSearch: true  },
        { title: '列2', dataIndex: '2', hideInSearch: true },
        { title: '列3', dataIndex: '3', hideInSearch: true },
        { title: '列4', dataIndex: '4', hideInSearch: true },
        { title: '列5', dataIndex: '5', hideInSearch: true },
        { title: '列6', dataIndex: '6', hideInSearch: true },
        {
          title: '操作',
          dataIndex: 'option',
          valueType: 'option',
          render: (_, record) => {
            return (
              <OperationsColumns
                operations={[
                  { id: 'edit', onClick: () => history.push(`/edit-page/${record.id}`) },
                  { id: 'del', onClick: async () => {} }
                ]}
              />
            )
          }
        }
      ]

      const { tableProps, exportTable, exportLoading } = useProTableRequest(api.get['/mock/sysUser'], {
        exportUrl: '/mock/export' // 导出URL
      })

      // 新增编辑弹窗
      const { modalProps, setShowModal, editData } = useProTableForm<any>()

      // 处理保存
      const [handleFinish] = useSuperLock(async (values: any) => {
        try {
          // await api.post['/mock/url'](values)
          message.success('操作成功')
          tableProps.actionRef.current?.reload()
          return true // 返回true关闭弹窗
        } catch (_error) {
          return false
        }
      })

      return (
        <PageContainer className={styles.[:=CamelCaseName:]Style}>
          <ProTable
            {...tableProps}
            columns={columns}
            search={{
              defaultCollapsed: false,
              labelWidth: 'auto',
              optionRender: (_searchConfig, _formProps, dom) => [
                ...dom,
                <Button type="primary" key="add" onClick={() => setShowModal()}>
                  新增
                </Button>
              ]
            }}
            toolBarRender={() => [
              <Button key="export" loading={exportLoading} onClick={() => exportTable()}>
                导出
              </Button>
            ]}
          />

          <ModalForm {...modalProps} onFinish={handleFinish}>
            <ProFormText name="name" label="名称" placeholder="请输入名称" fieldProps={{ maxLength: 200 }} />
          </ModalForm>
        </PageContainer>
      )
    }\n\n
    const [:=PascalName:] = memo(Component)\n
    export default [:=PascalName:]
  # index.module.less: |
  #   .[:=CamelCaseName:]Style {
  #   }
```

其中下面列明部分按需使用。如果不需要可以不在代码中编写
  - createStyles 用于创建样式
  - toolBarRender 用于自定义工具栏。比如导出按钮。
  - ModalForm 用于新增编辑操作。

## 代码要求

输出代码格式要求：

- 如果是页面代码。需要在`config/routes`中正确配置路由并在`pages`目录下创建对应的页面组件代码。
- 枚举文件/mock接口 等单独编写
- 使用代码块包裹，标明语言（tsx, ts）
- 加必要注释说明关键逻辑
- 枚举文件固定路径。 **~/enums/E{NAME}.ts**。一个枚举一个文件。并且需要写好 M{NAME} 与 O{NAME} 衍生对象。示例如下

```ts
/**
 * 短信code场景值
 */
export enum ECodeScene {
  /** 注册 */
  REG = 'REG',
  /** 忘记密码 */
  FORGOT = 'FORGOT'
}
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