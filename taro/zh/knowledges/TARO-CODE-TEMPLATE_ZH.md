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
    import PageContainer, { useToast } from
    '@wmeimob/taro-design/src/layout/pageContainer'

    interface I[:=PascalName:]Props {}

    const Component: FC<I[:=PascalName:]Props> = () => {
      // const [toast] = useToast()

      useEffect(() => {}, [])

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