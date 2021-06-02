# React useReducer 和 Context 对ts更友好的使用方式
先上代码
## 创建 Context
```ts 
// ./src/context/GlobalContext.tsx
/**
 * 定义要储存的类型接口
 */
export interface GlobalFace {
  userName: string
  age: number
}
/**
 * 初始值
 */
export const globalDataInit: GlobalFace = {
  userName: "张三",
  age: 0,
}
/**
 * GlobalReducer 接口
 */
export interface GlobalReducerProps {
  globalState: GlobalFace
  setGlobalState: Dispatch<Partial<GlobalFace>>
}
export const GlobalContext = createContext<GlobalReducerProps>({
  globalState: globalDataInit,
  setGlobalState: () => {
    throw new Error("GlobalContext 未定义")
  }
})
export const GlobalReducer = (prevState: GlobalFace, updatedProperty: Partial<GlobalFace>): GlobalFace => ({
  ...prevState,
  ...updatedProperty
})

```

## 当前层级组件使用
### 在当前组件使用的正确方式
```tsx
// ./src/App.tsx
const App: FC = () => {
  const [globalState, setGlobalState] = useReducer(GlobalReducer, globalDataInit)
  return (
    <GlobalContext.Provider value={{ globalState, setGlobalState }}>
      <div>
        <p>在当前组件正确使用</p>
        <p>姓名:{globalState.userName}</p>
        <p>年龄:{globalState.age}</p>
        <button onClick={() => {
          setGlobalState({ userName: '李四' })
        }}
        >设置姓名为李四
        </button>
      </div>
      <hr />
      <HomePage />
    </GlobalContext.Provider>
  )
}
export default App

```
### 在当前组件使用的错误方式

```tsx
// bad
const AppV2: FC = () => {
  const [globalState, setGlobalState] = useReducer(GlobalReducer, globalDataInit)
  const globalContext = useContext(GlobalContext)
  return (
    <GlobalContext.Provider value={{ globalState, setGlobalState }}>
      <div>
        <p>在当前组件错误的使用方式</p>
        <p>姓名:{globalContext.globalState.userName}</p>
        <p>年龄:{globalContext.globalState.age}</p>
        <button onClick={() => {
          globalContext.setGlobalState({ userName: '李四' })
        }}
        >设置姓名为李四
        </button>
      </div>
    </GlobalContext.Provider>
  )
}
export default AppV2
```
这里也体现了为什么我们在创建createContext，setGlobalState的默认值为抛出错误， 默认值仅在组件的树中没有匹配的 Provider 时被使用，正常情况下，默认值都会被覆盖，没被覆盖是不符合我们预期的，抛出错误有助于我们排查bug。

ps：现实中很多程序员非常不愿意抛出异常，这不是一个好习惯，Error也是代码的一部分，适当的时机抛出异常，有助于我们更容易的发现问题。

## 子组建使用
```tsx
// ./src/page/HomePage.tsx
const HomePage: FC = () => {
  const { globalState, setGlobalState } = useContext(GlobalContext)
  return (
    <div>
      <p>在子组件中使用</p>
      <p>{globalState.userName}</p>
      <p>年龄:{globalState.age}</p>
      <div>
        <button onClick={() => {
          setGlobalState({
            userName: '王五'
          })
        }}
        >设置姓名为王五
        </button>
        <button onClick={() => {
          setGlobalState({
            age: 18
          })
        }}
        >设置年龄为18
        </button>
      </div>
    </div>
  )
}
export default HomePage

```


## ts友好的useReducer方法
```ts
export const GlobalReducer = (prevState: GlobalFace, updatedProperty: Partial<GlobalFace>): GlobalFace => ({
  ...prevState,
  ...updatedProperty
})
const [globalState, setGlobalState] = useReducer(GlobalReducer, globalDataInit)
```
`Partial<T>` 是ts自身提供一种类型，作用是将传入的接口属性全部变为可选的

这个方法很好理解,利用`...`分解新老状态，然后用新状态覆盖老状态，并且返回
### 使用
```ts
const { setGlobalState } = useContext(GlobalContext)

setGlobalState({
  age:12
})
```
### 与官网demo方法对比
```ts
interface StateType {
  count: number
}
interface actionType {
  type: 'increment'|'decrement'|'customize'
  data: any
}
function reducer (state: StateType, action: actionType): StateType {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 }
    case 'decrement':
      return { count: state.count - 1 }
    case 'customize':
      return { count: action.data }
    default:
      throw new Error()
  }
}
```

相比之下，定义一个type字段，就算我们预先定义好type允许的值，但是当StateType包含属性类型很多data字段还要设为any类型，放弃了ts的类型检查是很不优雅的。


## 储存的状态非常多时怎么办

当要储存的状态非常多时，useReducer + Context方案也会很麻烦。
    
答:能问出这个问题的大多都是些滥用redux的程序员，俗话说redux是个筐，啥也往里装。大家应该能发现，我们往redux里储存的状态绝大多数都是从服务端获取，如果移除这些状态，实际我们要存的状态是非常少的。

服务端状态管理有个非常好用的库推荐[React Query](https://react-query.tanstack.com/),提供开箱即用的数据获取，缓存，更新，滚动回复等功能。

使用`React Query`做接口的状态管理，useReducer + Context 做本地状态管理。 双剑合璧，让你用少量的代码，简单的逻辑，做强健的状态管理。
