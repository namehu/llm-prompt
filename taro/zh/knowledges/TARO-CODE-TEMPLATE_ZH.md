## 代码编写模板

以下会列出在UMI框架下常用的代码模板。请你根据用户需求，选择合适的模板。严格按照团队编码规范编写。

### 项目结构

项目采用 `PNPM + MONOREPO` 方式管理。主体如下

```tree
.
├── components
│   ├── taro-design
│   │   ├── src/components // 组件库
│   │   ├── src/hooks
│   ├── utils
│   └── react-hooks
├── packages
│   ├── taro-template 后管项目
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

### taro-design

列出taro-design组件库的代码模板。

#### components\taro-design\src\components\button

```txt
// index.tsx
/**
 * @name 按钮
 * @description 按钮组件是用户界面中常见的交互元素，用于触发特定的操作或执行特定的功能。
 */
/* eslint-disable no-nested-ternary */
import { View } from '@tarojs/components'
import { ITouchEvent } from '@tarojs/components/types/common'
import { FC, PropsWithChildren, ReactNode, memo, useMemo } from 'react'
import classnames from 'classnames'
import MMLoading from '../loading'
import { MMButtonSize, MMButtonType } from './const'
import styles from './index.modules.less'
import { IComponentProps } from '../types'
import { useLockFunction } from '@wmeimob/react-hooks/src/useLockFunction'

export interface IButtonProps extends IComponentProps {
  /**
   * 按钮颜色
   */
  color?: string

  /** 是否为幽灵按钮 */
  ghost?: boolean

  /**
   * 加载状态
   */
  loading?: boolean

  /**
   * 按钮类型
   */
  type?: MMButtonType | keyof typeof MMButtonType

  /**
   * 按钮大小
   */
  size?: MMButtonSize | keyof typeof MMButtonSize

  /**
   * 禁用
   */
  disabled?: boolean

  /** 是否为block元素 */
  block?: boolean

  /**
   * 文字
   */
  text?: ReactNode

  /**
   * 圆角
   *
   * @description 设置为true.则携带默认圆角。 传递为数字则表示为指定值
   */
  radius?: boolean | number

  /**
   * 点击事件 返回的是promise 未运行完毕不会触发第二次
   */
  onClick?: (event: ITouchEvent) => void | Promise<any>
}

const Component: FC<PropsWithChildren<IButtonProps>> = (props) => {
  const { type = MMButtonType.primary, size = MMButtonSize.default, ghost = false, color, block, radius, style, loading, className, text, disabled } = props

  /**
   * 按钮颜色
   * 参数优先级 color > type
   * ghost 参数会改变背景、字体和边框颜色
   *
   */
  const colorStyle = useMemo(() => {
    let { background, fontColor, borderColor } = {
      [MMButtonType.primary]: {
        background: styles.primaryColor,
        borderColor: styles.primaryColor,
        fontColor: '#ffffff'
      },
      [MMButtonType.warning]: {
        background: styles.yellow,
        borderColor: styles.yellow,
        fontColor: '#ffffff'
      },
      [MMButtonType.default]: {
        background: '#ffffff',
        borderColor: styles.gray4,
        fontColor: styles.gray6
      }
    }[type!]

    if (color) {
      background = color
      borderColor = color
    }

    if (ghost) {
      fontColor = borderColor
      background = '#ffffff'
    }

    return {
      background,
      color: fontColor,
      borderColor
    }
  }, [color, ghost, type])

  const buttonStyle = {
    display: block ? 'block' : 'inline-block',
    borderRadius: typeof radius === 'number' ? radius : radius === false ? 0 : undefined,
    ...colorStyle,
    ...(style as any)
  }

  const rootClass = classnames(styles.MMButton, styles[size!], disabled && styles.disabled, className)

  const onClick = useLockFunction((event: ITouchEvent) => {
    if (disabled || loading) {
      return
    }
    return props.onClick?.(event)
  })

  return (
    <View className={rootClass} style={buttonStyle} onClick={onClick}>
      <View className={styles.MMButton_content}>
        {loading && (
          <View className={styles.MMButton_loading}>
            <MMLoading gray size={styles.loadingSize} />
          </View>
        )}

        <View>{props.children || text}</View>
      </View>
    </View>
  )
}

/**
 * @name 按钮
 */
const MMButton = memo(Component)
export default MMButton

// const.ts
export enum MMButtonType {
  /** 默认 */
  default = 'default',

  /** 主色 */
  primary = 'primary',

  /** 警告 */
  warning = 'warning'
}

export enum MMButtonSize {
  large = 'large',
  default = 'default',
  small = 'small',
  tiny = 'tiny'
}

// index.module.less
@import '../styles/themes/default.less';

@loadingSize: 20;

:export {
  loadingSize: @loadingSize;
}

.MMButton_content {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
}

.MMButton_loading {
  width: 20px;
  height: 20px;
  margin-right: @spacingBase;
}

.MMButton {
  box-sizing: border-box;
  text-align: center;
  color: @gray1;
  border-radius: @buttonBorderRadius;
  border-width: 1px;
  border-style: solid;
  padding: 0 @spacingBase * 3;
  // display: inline-block;

  &:active {
    opacity: 0.8;
  }

  &.disabled {
    opacity: 0.4 !important;

    &:active {
      // background: @buttonColor !important;
      opacity: 0.4 !important;
    }
  }

  &.large {
    height: @buttonHeight + 4px;
    font-size: @buttonFontSize + 2;
  }

  &.default {
    height: @buttonHeight;
    font-size: @buttonFontSize;
  }

  &.small {
    height: @buttonHeight - 4px;
    font-size: @buttonFontSize - 1px;
  }

  &.tiny {
    height: @buttonHeight - 8px;
    font-size: @buttonFontSize - 3px;
    padding: 0 @spacingBase;
  }
}
```

#### components\taro-design\src\layout\pageContainer\index.tsx

```ts
import { View } from '@tarojs/components'
import { getCurrentInstance } from '@tarojs/taro'
import ContainerDialog from '../../components/dialog/ContainerDialog'
import MMToast from '../../components/toast'
import { isNewIphone } from '../../components/utils'
import { CSSProperties, FC, memo, PropsWithChildren, useMemo } from 'react'
import { setDialog, setToast, useDialog, useToast } from './const'

interface IPageContainerProps {
  /**
   * 是否需要垫高
   */
  noPlace?: boolean

  /**
   * 是否是tab页
   *
   * 设置为tab页面。会额外提供一个tabbar的高度
   */
  isTab?: boolean

  className?: string

  style?: CSSProperties | string
}

/**
 * 通用页面容器
 *
 * 给页面添加安全底部
 */
const Component: FC<PropsWithChildren<IPageContainerProps>> = (props) => {
  const { noPlace = false, isTab = false, ...rest } = props

  const memoToast = useMemo(() => {
    const path = getCurrentInstance().router?.path || ''
    return <MMToast ref={(ref) => setToast(path, ref as any)} />
  }, [])

  const memoDialog = useMemo(() => {
    const path = getCurrentInstance().router?.path || ''
    return <ContainerDialog ref={(ref) => setDialog(path, ref as any)} />
  }, [])

  return (
    <View {...rest}>
      {props.children}

      {isTab && <View style={{ height: 50 }} />}

      {isNewIphone && !noPlace && <View className="spacingIphone" />}

      {memoToast}

      {memoDialog}
    </View>
  )
}

const PageContainer = memo(Component)
export default PageContainer

export { useDialog, useToast }

```

#### components\taro-design\src\layout\pageContainer\const.ts

```ts
import Taro, { getCurrentInstance } from '@tarojs/taro'
import { useState, useRef, useEffect, useCallback } from 'react'
import { IToastRef } from '../../components/toast/const'
import { IContainerDialogRef } from '../../components/dialog/ContainerDialog'

/** toast弹窗缓存对象 */
const toastMap: Record<string, IToastRef | null> = {}

/**
 * 设置toast
 * 会触发taro消息。通知所有的useToast更新toast
 *
 * 并且由于PageContainer中使用的是回调ref。在页面销毁时会自动将ref设置为
 * 所以有个自动销毁机制。你并不需要额外处理销毁
 */
export const setToast = (key: string, value: IToastRef | null) => {
  toastMap[key] = value
  // console.log(toastMap, value, 'setToast', `toast_${key}`)
  if (value) {
    Taro.eventCenter.trigger(`toast_${key}`)
  }
}

/**
 * 获取toast实例
 * 在useEffect中首次并不能保证已经存在
 *
 * @warning 必须与PageContainer组件一起使用！！！
 */
export function useToast() {
  const [, setFlag] = useState(false)
  // WARNNING: 这里H5 可能有bug。随时关注
  const pathRef = useRef<string>(getCurrentInstance().router?.path || '')

  const eventHanlder = useCallback(() => {
    setFlag((pre) => !pre)
  }, [])

  Taro.eventCenter.on(`toast_${pathRef.current}`, eventHanlder)

  useEffect(() => {
    return () => {
      Taro.eventCenter.off(`toast_${pathRef.current}`, eventHanlder)
    }
  })

  return [toastMap[pathRef.current]]
}

/** toast弹窗缓存对象 */
const dialogMap: Record<string, IContainerDialogRef | null> = {}
/**
 * 设置dialogMap
 * 会触发taro消息。通知所有的useDialog更新dialog
 *
 * 并且由于PageContainer中使用的是回调ref。在页面销毁时会自动将ref设置为null
 * 所以有个自动销毁机制。你并不需要额外处理销毁
 */
export const setDialog = (key: string, value: IContainerDialogRef | null) => {
  dialogMap[key] = value
  // console.log(dialogMap, value, 'dialog_', `dialog_${key}`)
  if (value) {
    Taro.eventCenter.trigger(`dialog_${key}`)
  }
}

/**
 * 获取对话框
 * 在useEffect中首次并不能保证已经存在
 *
 * @warning 必须与PageContainer组件一起使用！！！
 */
export function useDialog() {
  const [_, setFlag] = useState(false)
  // WARNNING: 这里H5 可能有bug。随时关注
  const pathRef = useRef<string>(getCurrentInstance().router?.path || '')

  const eventHanlder = useCallback(() => {
    setFlag((pre) => !pre)
  }, [])

  Taro.eventCenter.on(`dialog_${pathRef.current}`, eventHanlder)

  useEffect(() => {
    return () => {
      Taro.eventCenter.off(`dialog_${pathRef.current}`, eventHanlder)
    }
  })

  return dialogMap[pathRef.current]
}
```

#### components\taro-design\src\components\navigation\index.tsx

```ts
/**
 * @name 导航栏
 * @description 导航栏组件是一个常见的用户界面元素，用于在网页或应用程序中提供导航链接和菜单选项，帮助用户快速浏览和访问不同的页面或功能。
 */
import { View } from '@tarojs/components'
import { CSSProperties, FC, memo, NamedExoticComponent, PropsWithChildren, ReactNode, useMemo } from 'react'
import Taro, { getCurrentPages, getMenuButtonBoundingClientRect, getSystemInfoSync } from '@tarojs/taro'
import classnames from 'classnames'
import MMIconFont from '../icon-font'
import styles from './index.modules.less'
import MMIconFontName from '../icon-font/const'
import { IComponentProps } from '../types'

 enum MMNavigationType {
  /** 白色背景，黑色字体 */
  Default = 'Default',
  /** 透明背景。白色字体 */
  Transparent = 'Transparent',

  Primary = 'Primary'
}

export interface IMMNavigationProps extends IComponentProps {
  // 中间显示的标题
  title?: string
  // 渲染左边的元素
  renderLeft?: ReactNode
  // 类型
  type?: MMNavigationType | keyof typeof MMNavigationType

  /**
   * 是否占据高度 当有通栏的背景图时 或许比较有用
   * @default true
   */
  place?: boolean

  /**
   * 是否显示导航阴影
   * 默认情况下导航组件在下面会有一个box-shadow阴影。有时候你需要关掉它
   * @default true
   */
  shadow?: boolean

  /**
   * 导航条样式
   */
  contentClass?: any

  /**
   * 导航条样式
   */
  contentStyle?: CSSProperties

  /**
   * 点击返回之前处理函数
   *
   * @description 你可以通过此函数在返回之前做拦截处理操作。 支持返回一个者异步函数 。结果为true时可以返回。为false时阻止返回
   */
  beforeNavBack?: () => Promise<boolean> | boolean
}

const isWeapp = process.env.TARO_ENV === 'weapp'
// h5暂时不支持 API getMenuButtonBoundingClientRect, 模拟导航栏iphone6/7/8固定高度
const statusBarHeight = isWeapp ? getSystemInfoSync().statusBarHeight ?? 0 : 20

const menuButtonBoundingClientRect = isWeapp
  ? getMenuButtonBoundingClientRect()
  : {
      bottom: 56,
      height: 32,
      left: 278,
      right: 365,
      top: 24,
      width: 87
    }

const stateHeigth = (menuButtonBoundingClientRect.top - statusBarHeight) * 2 + menuButtonBoundingClientRect.height

export const navigationHeight = isWeapp ? stateHeigth + statusBarHeight : stateHeigth

const Component: FC<PropsWithChildren<IMMNavigationProps>> = (props) => {
  const { title, place = true, shadow = false, type = MMNavigationType.Default, renderLeft, beforeNavBack } = props

  const rootStyle = useMemo(
    () => ({
      height: !place ? 0 : `${navigationHeight}px`,
      borderBottom: place ? '0.5px solid transparent' : 'unset',
      ...props.style
    }),
    [place, props.style]
  )

  const className = useMemo(() => {
    return classnames(
      styles.fixed,
      {
        [MMNavigationType.Transparent]: styles.fixed__transparent,
        [MMNavigationType.Primary]: styles.fixed__primary
      }[type],
      props.contentClass
    )
  }, [type, props.contentClass])

  async function hanldeNavBack() {
    // 省略
  }

  const renderGoBack = () => {
    // // 省略
  }

  return (
    <View className={classnames(styles.MMNavigation, props.className)} style={rootStyle}>
      <View
        className={className}
        style={{
          zIndex: 1000,
          boxShadow: shadow ? styles.boxShadow : 'none',
          paddingTop: isWeapp ? `${statusBarHeight}px` : 0,
          ...props.contentStyle
        }}
      >
        <View className={styles.content} style={{ height: `${stateHeigth}px` }}>
          {renderLeft === false ? null : renderLeft ? <View className={styles.leftBox}>{props.renderLeft}</View> : renderGoBack()}

          <View className={styles.title}>{props.children || title}</View>
        </View>
      </View>
    </View>
  )
}

const MMNavigation = memo(Component) as NamedExoticComponent<PropsWithChildren<IMMNavigationProps>> & {
  /**
   * 导航占位高度
   * 当你使用place： false时。你可以通过这个属性拿到原本导航占据的高度
   */
  navigationHeight: number
}

MMNavigation.navigationHeight = navigationHeight
export default MMNavigation

```

#### components\taro-design\src\components\toast

```ts
/* eslint-disable max-nested-callbacks */
import { ReactNode, useEffect, useImperativeHandle, useMemo, useRef, useState } from 'react'
import MMIconFontName from '../icon-font/const'
import { guid } from '../utils'

export interface IToastProps {
  /**
   * 持续时间
   *
   */
  duration?: number

  /**
   * 全局是否有蒙层
   */
  mask?: boolean
}

export enum ToastState {
  new,
  slideIn,
  slideOut,

  fadeIn,
  fadeOut
}

export enum EAnimationType {
  /** 淡入淡出 */
  fade = 'fade',

  /** 滑动滑出 */
  slideup = 'slideup'
}

/** 弹窗位置 */
export enum EToastPosition {
  /** 顶部 */
  top = 'top',
  /** 居中 */
  center = 'center',
  /** 底部 */
  bottom = 'bottom'
}

export interface IToastMessage {
  /** 消息 */
  message: ReactNode

  /** 自定义icon */
  icon?: MMIconFontName

  /** 自定义图片 */
  img?: string

  /**
   * 动画方式
   *
   * @default fade 淡入淡出
   */
  animationType?: EAnimationType | keyof typeof EAnimationType

  /**
   * 弹窗位置
   */
  position?: EToastPosition | keyof typeof EToastPosition

  /**
   * 持续时间
   * 覆盖默认的持续时间
   */
  duration?: number

  /**
   * 是否有蒙层
   */
  mask?: boolean

  /**
   * 是否不显示
   *
   * @description 比如当你需要一个loading态蒙层防止点击事件时。你可以设置此属性
   */
  hidden?: boolean
}

export interface IToastMessageState extends IToastMessage {
  id: string

  /** 当前动画状态 */
  state: ToastState

  /** 退出状态 */
  outState: ToastState

  /** 回调函数 */
  cb?: IToastMessageCallBack
}

export type IToastMessageCallBack = () => void

export type IToastAction = Partial<Omit<IToastMessage, 'icon'>>

export type IToastRef = ReturnType<typeof useToastService>['imperativeHandler'] // Toast ref引用类型

export type IToastInstance = {
  /** 更新message信息 */
  setMessage(message: IToastMessage['message']): void
  /** 隐藏当前弹窗 */
  hide(): void
}

export interface IHideOption {
  /** toast */
  id: string
  /**
   * 延迟时间
   * @default transitionTiming = 500
   */
  time?: number
}

const { slideIn, slideOut, fadeIn, fadeOut } = ToastState
const transitionTiming = 500

/**
 * 组件逻辑hook
 * @param props
 * @param ref
 * @returns
 */
export default function useToastService(props: IToastProps, ref: any) {
  const { duration = 2000, mask = false } = props
  const [messages, setMessages] = useState<IToastMessageState[]>([])
  // const clearSetTimeout = useRef<any>()
  const durationTime = useRef<Record<string, any>>({})

  const showMask = useMemo(() => messages.some(it => it.mask), [messages])

  const loadingRef = useRef('')

  /**
   * 设置隐藏当前弹窗实例
   */
  function setHide({ id, time = transitionTiming }: IHideOption) {
    // 执行推出动画
    if (time > 50) {
      setMessages(pre => pre.map(value => (value.id === id ? { ...value, state: value.outState } : value)))
    }
    setTimeout(() => {
      // 实例推出
      setMessages(pre =>
        pre.filter(value => {
          // 在此执行回调函数函数
          if (value.id === id) {
            setTimeout(() => {
              value.cb?.()
            }, 0)
          }
          return value.id !== id
        })
      )
    }, time)
  }

  function addToast(option: IToastMessage, cb?: IToastMessageCallBack): IToastInstance | undefined {
    const id = guid()

    if ((option.icon as any) === 'loading') {
      if (loadingRef.current) {
        return undefined
      }
      // 如果是loadind 缓存id
      loadingRef.current = id
    }

    // 计算动画类型
    const { animationType = EAnimationType.fade } = option
    const [ain, outState] = {
      [EAnimationType.slideup]: [slideIn, slideOut],
      [EAnimationType.fade]: [fadeIn, fadeOut]
    }[animationType]

    // 推入栈
    setMessages(pre => [...pre, { mask, ...option, id, state: ToastState.new, outState, cb }])

    // 执行进入动画并计算持续时间
    setTimeout(() => {
      setMessages(pre => pre.map(value => (value.id === id ? { ...value, state: ain } : value)))
      setDuration()
    }, 50)

    // 设置持续时间
    function setDuration() {
      if (durationTime.current[id]) {
        clearTimeout(durationTime.current[id])
      }
      // 计算退出时间
      durationTime.current[id] = setTimeout(() => {
        setHide({ id })
      }, (option.duration || duration) - 100)
    }

    return {
      /**
       * 更新弹窗信息
       * 更新内容后会重置弹窗持续时间
       */
      setMessage(message) {
        setMessages(pre => pre.map(me => (me.id === id ? { ...me, message } : me)))
        setDuration()
      },
      /**
       * 隐藏弹窗
       */
      hide: () => setHide({ id, time: 100 })
    }
  }

  function addToastWrapper(message: string | IToastAction, options?: Partial<IToastMessage>, cb?: IToastMessageCallBack) {
    const option = typeof message === 'string' ? { message } : { ...message }
    return addToast({ message: '', ...option, ...options }, cb)
  }

  const imperativeHandler = {
    /**
     * 弹出一个信息提示框
     *
     */
    message: (message: string | IToastMessage, cb?: IToastMessageCallBack) => addToast(typeof message === 'string' ? { message } : { ...message }, cb)!,

    /**
     * 弹出一个失败信息提示框
     *
     */
    fail: (message: string | IToastAction, cb?: IToastMessageCallBack) => addToastWrapper(message, { icon: MMIconFontName.Close }, cb)!,

    /**
     * 弹出一个成功信息提示框
     *
     */
    success: (message: string | IToastAction, cb?: IToastMessageCallBack) => addToastWrapper(message, { icon: MMIconFontName.Check }, cb)!,

    /**
     * 显示一个loading
     *
     * @duration 强制持续时间为10s
     * @description loading设计为单例模式。 在组件级别只存在一个
     * loading状态下mask强制为true.也就是会阻止一切点击事件。
     * 所以你必须手动调用toast.hideLoading()来关闭
     */
    loading: (message?: string | IToastAction, cb?: IToastMessageCallBack) => {
      let option: IToastAction = {
        duration: 10000
      }

      option = typeof message === 'string' ? { ...option, message } : { ...option, ...message }
      return addToastWrapper(option, { icon: 'loading' as any, mask: true }, cb)
    },

    /**
     * 隐藏loading
     */
    hideLoading: () => {
      if (loadingRef.current) {
        setHide({ id: loadingRef.current, time: 100 })
        loadingRef.current = ''
      }
    }
  }

  useImperativeHandle(ref, () => imperativeHandler, [duration])

  useEffect(() => {
    return () => {
      // clearSetTimeout.current && clearTimeout(clearSetTimeout.current)
      Object.keys(durationTime.current).forEach(key => {
        durationTime.current[key] && clearTimeout(durationTime.current[key])
      })
    }
  }, [])

  return {
    messages,
    showMask,
    imperativeHandler
  }
}
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

### 代码模板

提供不同类型页面模板。其中下面列明部分按需使用。如果不需要可以不在代码中编写

- 页面模板主体格式尤其是jsx部分必须保留布局格式
- 文件中有诸如[:=xxx:]的字符。这是模板变量。在生成模板的过程中。你可以通过模板变量来进行更加细致的控制。支持的变量有: CamelCaseName 组件驼峰命名;PascalName 组件帕斯卡命名;KebabCaseName 组件横杠线命名;UnderlineCase 下划线命名;dirname 组件目录名

#### taro-function-page.yml Taro 函数式页面模板

```yml
name: '@wmeimob/taro-function-page'
description: Taro 函数式页面模板
tags: []
tpl:
  index.tsx: >
    import Taro from '@tarojs/taro'
    import { FC, memo, useEffect } from 'react'
    import { View } from '@tarojs/components'
    import styles from './index.module.less'
    import MMNavigation from '@wmeimob/taro-design/src/components/navigation'
    import PageContainer, { useToast, useDialog } from
    '@wmeimob/taro-design/src/layout/pageContainer'
    import { api } from '~/request'

    interface I[:=PascalName:]Props {}

    const Component: FC<I[:=PascalName:]Props> = () => {
      const [toast] = useToast()
      const dialog = useDialog()

      useEffect(() => {
        fetchDetail()
      }, [])

      async function fetchDetail() {
        const res = await api.get['/mock/user']({})
        setUser(res.data)
      }

      // 自定义逻辑
      function customLogic() {
        toast.success('自定义逻辑')
        dialog?.show({title: '哈哈哈', content: '哈哈哈哈' })
      }

      return (
        <PageContainer className={styles.[:=CamelCaseName:]Style}>
          <MMNavigation title="[:=CamelCaseName:]" />

          <View>[:=CamelCaseName:]</View>
        </PageContainer>
      )
    }

    const [:=PascalName:] = memo(Component)
    export default [:=PascalName:]
  index.module.less: |
    .[:=CamelCaseName:]Style {

    }
```

#### taro-function-component.yml 组件模板

```yml
name: '@wmeimob/taro-function-component'
description: Taro 函数组件模板
tags: []
tpl:
  index.tsx: |
    import { memo, FC } from 'react'
    import { View, Text } from '@tarojs/components'
    import styles from './index.module.less'

    interface I[:=PascalName:]Props {}

    const Component: FC<I[:=PascalName:]Props> = (props) => {
      // const {} = props;

      return (
        <View className={styles.[:=CamelCaseName:]Style}>
          <Text>[:=CamelCaseName:]</Text>
        </View>
      )
    }

    const [:=PascalName:] = memo(Component)
    export default [:=PascalName:]
  index.module.less: |
    .[:=CamelCaseName:]Style {

    }

```

### 代码要求

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