---
name: '@wmeimob/react-function-component'
type:
  - react-component
description: 组件模板
---

# 模板说明
组件模板

## 文件内容

### index.tsx
```tsx
import { FC, memo } from 'react'
import styles from './index.module.less'

interface I[:=PascalName:]Props {}

const Component: FC<I[:=PascalName:]Props> = (props) => {
  const {} = props

  return (
    <div className={styles.[:=CamelCaseName:]Style}>[:=CamelCaseName:]</div>
  )
}

const [:=PascalName:] = memo(Component)
export default [:=PascalName:]
```

### index.module.less
```less
.[:=CamelCaseName:]Style {
}
```
