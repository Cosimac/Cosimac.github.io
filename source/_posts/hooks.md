---
title: hooks简介
date: 2024-07-24 13:58:30
tags: js
---

# [React](https://so.csdn.net/so/search?q=React&spm=1001.2101.3001.7020) Hooks

## Hooks简介

> 介绍Hooks之前，首先要说一下React的组件创建方式，一种是类组件，一种是纯函数组件，并且React团队希望，组件不要变成复杂的容器，最好只是数据流的管道。开发者根据需要，组合管道即可。也就是说组件的最佳写法应该是函数，而不是类。

但是，在以往开发中类组件和纯函数组件的区别是很大的，纯函数组件有着类组件不具备的多种特点：

- 纯函数组件没有状态
- 纯函数组件没有生命周期
- 纯函数组件没有this

这就注定，我们所推崇的函数组件，只能做UI展示的功能，涉及到状态的管理与切换，不得不用类组件或者redux，但类组件也是有缺点的，比如，遇到简单的页面，代码会显得很重，并且每创建一个类组件，都要去继承一个React实例

**React Hooks：** 就是用函数的形式代替原来的继承类的形式，并且使用预函数的形式管理state，有Hooks可以不再使用类的形式定义组件了。

这时候认知也要发生变化了，原来把组件分为有状态组件和无状态组件，有状态组件用类的形式声明，无状态组件用函数的形式声明。那现在所有的组件都可以用函数来声明了。

**使用Hooks的优点：**

- 告别难以理解的Class( this 和 生命周期 的痛点)
- 解决业务逻辑难以拆分的问题
- 使状态逻辑复用变得简单可行
- 函数组件从设计思想上来看更加契合React的理念

**Hooks并非万能：**

- Hooks暂时还不能完全的为函数组件补齐类组件地能力（如生命周期的getSnapshotBeforeUpdate、componentDidCatch方法暂时还未实现）
- 将类组件的复杂变成函数组件的轻量，可能使用者并不能很好地消化这种复杂
- Hooks在使用层面有着严格地规则约束

**例如：类组件实现计数器：**

```javascript
import React, {Component} from "react";
class AddCount extends Component {
    constructor(props) {
        super(props);
        this.state = {
            count : 0
        }
    }
    addcount = () => {
        let newCount = this.state.count;
        this.setState({
            count: newCount += 1
        })
    }
    render() {
        return (
            <>
                <p>{ this.state.count }</p>
                <button onClick={ this.addcount }>count++</button>
            </>
        )
    }
}
export default AddCount;
```

可以看出来，上面的代码确实很重。因此React队设计了React Hooks。React Hooks就是加强版的函数组件，可以完全不使用 class，就能写出一个全功能的组件

React Hooks 使得组件尽量写成纯函数，如果需要外部功能和副作用，就用钩子把外部代码"钩"进来。而React Hooks 就是所说的“钩子”。

那么Hooks要怎么用呢？“你需要写什么功能，就用什么钩子”。对于常见的功能，React为我们提供了一些常用的钩子，当然有特殊需要，我们也可以写自己的钩子。下面是React Hooks为我们提供的默认的四种最常用钩子：

- useState()
- useContext()
- useEffect()
- useReducer()

不同的钩子为函数引入不同的外部功能，上面四种钩子都带有use前缀，React Hooks约定，钩子一律使用use前缀命名。所以，自己定义的钩子都要命名为useXXX。

## Hook函数（9种）

---

**1、State Hook**

useState()：状态钩子。纯函数组件没有状态，**用于为函数组件引入state状态, 并进行状态数据的读写操作**

**语法、参数及返回值说明:：**

```javascript
const [xxx, setXxx] = React.useState(initValue) 
```

- **参数:** 第一次初始化指定的值在内部作缓存
- **返回值:** 包含2个元素的数组，第1个为内部当前状态值，第2个为更新状态值的函数

**setXxx()2种写法:**

- **setXxx(newValue)**: 参数为非函数值，直接指定新的状态值，内部用其覆盖原来的状态值
- **setXxx(value => newValue)**: 参数为函数，接收原本的状态值，返回新的状态值，内部用其覆盖原来的状态值

**例如：React Hooks 实现计数器：**

```javascript
import React,{ useState } from "react";

const NewCount = ()=> {
    const [ count,setCount ] = useState(0)
    addCount = ()=> {
        let newCount = count;
        setCount(newCount +=1)
    }
   return (
       <>
           <p> { count }</p>
           <button onClick={ addCount }>Count++</button>
       </>
   )
}
export default NewCount;
```

代码看起来更加的轻便简洁，没有了继承，没有了渲染逻辑，没有了[生命周期](https://so.csdn.net/so/search?q=%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F&spm=1001.2101.3001.7020)等

---

**2、Context Hook**

useContext()：共享状态钩子。**作用就是可以做状态的分发**，在React16.X以后支持，避免了react逐层通过Props传递数据。

> Context：一种组件间通信方式, 常用于【祖组件】与【后代组件】间通信

**使用语法和说明：**

1. 创建Context容器对象：
    
    ```javascript
    const XxxContext = React.createContext()  
    ```
    
2. 渲染子组件时，外面包裹xxxContext.Provider， 通过value属性给后代组件传递数据：
    
    ```javascript
    <xxxContext.Provider value={数据}>
    	<子组件/>
    </xxxContext.Provider>
    ```
    
3. 后代组件读取数据：
    
    ```javascript
    const {} = useContext(XxxContext)
    ```
    

**例如：A组件和B组件需要共享一个状态:**

```javascript
import React, { useContext } from "react";
const HookTest = ()=> {
    const AppContext = React.createContext();
    const A = ()=> {
        const { name } = useContext(AppContext)
        return (
            <p>
                我是A组件,我的名字是：{ name }；
                <span>我是A的子标签：{ name }</span>
            </p>
        )
    }
    const B= ()=> {
        const { name } = useContext(AppContext);
        return (
            <p>我是B组件,名字是： { name }</p>
        )
    }
    return (
        <AppContext.Provider value={{ name: '张三'}}>
            <A />
            <B />
        </AppContext.Provider>
    )
}
export default HookTest;
```

---

**3、Effect Hook**

useEffect()：副作用钩子。**用来更好的执行副作用操作**(用于模拟类组件中的生命周期钩子)，如异步请求等，在类组件中会把请求放在componentDidMount里面，在函数组件中可以使用useEffect()

**语法和说明:**

```javascript
useEffect(() => { 
      // 在此可以执行任何带副作用操作
      return () => { // 在组件卸载前执行
        // 在此做一些收尾工作, 比如清除定时器/取消订阅等
      }
}, [stateValue]) // 如果指定的是[], 回调函数只会在第一次render()后执行
```

**参数、返回值说明：**

- useEffect()接受两个参数，**第一个参数是要进行的异步操作**，**第二个参数是一个数组**，用来给出Effect的**依赖项**，只要这个数组发生变化，useEffect()就会执行。
- 当第二项省略不填时。useEffect()会在每次组件渲染时都会执行useEffect，只要更新就会执行。
- 当第二项传 **空数组\[ \]** 时，只会在组件挂载后运行一次。
- useEffect()返回值可以是一个函数，在组件销毁的时候会被调用。清理这些副作用可以进行如取消订阅、清除定时器操作，类似于componentWillUnmount。

**React中的副作用操作:**

- 发ajax请求数据获取
- 设置订阅 / 启动定时器
- 手动更改真实DOM

**useEffect两个注意点：**

- React首次渲染和之后的每次渲染都会调用一遍useEffect函数，而之前要用两个生命周期函数分别表示 **首次渲染(componentDidMonut)** 和**更新导致的重新渲染(componentDidUpdate)**
- useEffect中定义的函数的执行不会阻碍浏览器更新视图，也就是说这些函数时异步执行的，而componentDidMonut和componentDidUpdate中的代码都是同步执行的。

**总结：** 可以把 useEffect Hook 看做如下三个函数的组合 ：**componentDidMount()、componentDidUpdate()、componentWillUnmount()**

1、类似于componentDidMount的useEffect

```javascript
import { useEffect } from 'react'

const Demo = () => {
  useEffcet(() => {
	console.log('类似于componentDidMount，通常在此处调用api获取数据')
  }, [])
}

export default Demo
```

2、类似于componentWillUnmount的useEffect

```javascript
import { useEffect } from 'react'

const Demo = () => {
  useEffcet(() => {
	return () => {
	  console.log('类似于componentWillUnmount，通常用于清除副作用');
	}
  }, [])
}

export default Demo
```

3、类似于componentDidUpdate的useEffect

```javascript
import { useState,useEffect } from 'react'

const Demo = () => {
  const [count,setCount] = React.useState(0)
	
  useEffcet(() => {
	console.log('当count发生改变时，执行当前区域的代码')
  }, [count])
}
```

---

**4、Reducer Hook**

useReducer()：Action钩子。在使用React的过程中，如遇到状态管理，一般会用到Redux。而React本身是不提供状态管理的。而useReducer() **提供了状态管理**。

首先，关于redux我们都知道，其原理是**通过用户在页面中发起action，从而通过reducer方法来改变state，从而实现页面和状态的通信。**

而Reducer的形式是(state, action) => newstate。hooks的形式如下：

**语法格式：**

```javascript
const [state, dispatch] = useReducer(reducer, initialState)
```

**参数、返回值说明：**

它接受 **reducer函数** 和 **状态的初始值** 作为参数，返回一个**数组**，其中第一项为**当前的状态值**，第二项为**发送action的dispatch函数**。

**例如：使用useReducer()实现一个计数器**

```javascript
import  { useReducer } from "react";
const HookReducer = ()=> {
    const reducer = (state,action)=> {
        if (action.type === 'add') {
            return {
                ...state,
                count: state.count + 1
            }
        }else {
            return state
        }
    }
    const addCount = ()=> {
        dispatch({
            type: 'add'
        })
    }
    const [state,dispatch ] = useReducer(reducer,{count: 0})
    return (
        <>
            <p>{state.count}</p>
            <button onClick={ addCount }>useReducer</button>
        </>
    )
}
export default HookReducer;
```

通过代码可以看到，使用useReducer()代替了Redux的功能，但useReducer无法提供中间件等功能，假如有这些需求，还是需要用到redux。

---

**5、Ref Hook**

userRefef()：Ref Hook可以**在函数组件中存储、查找组件内的标签或任意其它数据**

**语法和参数说明：**

```javascript
const refContainer = useRef()
```

useRef返回一个可变的ref对象，useRef接受一个参数绑定在返回的ref对象的current属性上，返回的ref对象在整个生命周期中保持不变。

作用：保存标签对象，功能与React.createRef()一样

**例子：input上绑定一个ref，使得input在渲染后自动焦点聚焦**

```javascript
import{ useRef,useEffect} from "react";
const RefComponent = () => {
    let inputRef = useRef(null);
    useEffect(() => {
        inputRef.current.focus();
    })
    return (
        <input type="text" ref={inputRef}/>
    ) 
}
```

---

**6、Memo Hook**

useMemo()： 主要**用来解决使用React hooks产生的无用渲染的性能问题**。

**语法和参数说明：**

```javascript
const cacheSomething = useMemo(create,deps)
```

- `create`：第一个参数为一个函数，函数的返回值作为缓存值
- `deps`： 第二个参数为一个数组，存放当前 useMemo 的依赖项，在函数组件下一次执行的时候，会对比 deps 依赖项里面的状态，是否有改变，如果有改变重新执行 create ，得到新的缓存值。
- `cacheSomething`：返回值，执行 create 的返回值。如果 deps 中有依赖项改变，返回的重新执行 create 产生的值，否则取上一次缓存

使用function的形式来声明组件，失去了shouldCompnentUpdate（在组件更新之前）这个生命周期，也就是说没有办法通过组件更新前条件来决定组件是否更新。

而且在函数组件中，也不再区分mount和update两个状态，这意味着函数组件的每一次调用都会执行内部的所有逻辑，就带来了非常大的性能损耗。

**useMemo原理：**

useMemo 会记录上一次执行 create 的返回值，并把它绑定在函数组件对应的 fiber 对象上，只要组件不销毁，缓存值就一直存在，但是 deps 中如果有一项改变，就会重新执行 create ，返回值作为新的值记录到 fiber 对象上。

**useMemo应用场景：**

- 可以缓存 element 对象，从而达到按条件渲染组件，优化性能的作用。
- 如果组件中不期望每次 render 都重新计算一些值，可以利用 useMemo 把它缓存起来。
- 可以把函数和属性缓存起来，作为 PureComponent 的绑定方法，或者配合其他Hooks一起使用

---

**7、Callback Hook**

useCallback()： 主要**是为了性能的优化**

**useCallback(fn, deps) 相当于 useMemo(() => fn, deps)**

可以认为是对依赖项的监听，接受一个**回调函数**和**依赖项数组**。

- useCallback会返回一个函数的memoized(记忆的)值。
- 该回调函数仅在某个依赖项改变时才会
- 在依赖不变的情况下，多次定义的时候，返回的值是相同的

```javascript
import {useState,useCallback} from "react";

const CallbackComponent = () => {
    let [count, setCount] = useState(1);
    let [num, setNum] = useState(1);

    const memoized = useCallback(() => {
        return num;
    }, [count])
    console.log("记忆：", memoized());
    console.log("原始：", num);
   return (
        <>
            <button onClick={() => {setCount(count + 1)}}> count+</button>
            <button onClick={() => {setNum(num + 1)}}> num+</button>
        </>
    )
}
export default CallbackComponent
```

如果没有传入依赖项数组，那么记忆函数在每次渲染的时候都会更新。

---

**8、LayoutEffect Hook**

useLayoutEffect() ：和useEffect相同，**都是用来执行副作用，但是它会在所有的DOM变更之后同步调用effect**。useLayoutEffect和useEffect最大的区别就是一个是同步，一个是异步。

从这个Hook的名字上也可以看出，它主要用来读取DOM布局并触发同步渲染，在浏览器执行绘制之前，useLayoutEffect 内部的更新计划将被同步刷新。

官网建议还是尽可能的是使用标准的useEffec以避免阻塞视觉更新。

---

**9、ImperativeHandle Hook**

useImperativeHandle()： 可以**在使用 ref 时自定义暴露给父组件的实例值。**

就是说：当使用父组件把ref传递给子组件的时候，这个Hook允许在子组件中把自定义实例附加到父组件传过来的ref上，有利于父组件控制子组件。

**使用方式：**

```javascript
import {useEffect,useRef,useImperativeHandle} from "react";
import {forwardRef} from "react";

function FancyInput(props, ref) {
    const inputRef = useRef();
    useImperativeHandle(ref, () => ({
        focus: () => {
            inputRef.current.value="Hello";
        }
    }));
    return <input ref={inputRef} />;
}
FancyInput = forwardRef(FancyInput);

const ImperativeHandleTest=() => {
    let ref = useRef(null);
    useEffect(() => {
        console.log(ref);
        ref.current.focus();
    })
    return (
        <>
            <FancyInput ref={ref}/>
        </>
    )
}
export default ImperativeHandleTest
```

## 自定义Hooks

> 有时候我们需要创建自己想要的Hooks，来满足更便捷的开发，就是根据业务场景对其它Hooks进行组装，从而得到满足自己需求的钩子。

**自定义 Hooks：是一个函数，其名称以 “use” 开头，函数内部可以调用其他的 Hook**

**自定义Hooks：可以封装状态，能够更好的实现状态共享**

自定义hooks可以说成是一种约定而不是功能。当一个函数以use开头并且在函数内部调用其他hooks，那么这个函数就可以成为自定义hooks

**例如：**

```javascript
import { useState,useEffect } from "react";
const usePerson = ({name}) => {
    const [loading, setLoading] = useState(true)
    const [person, setPerson] = useState({})

    useEffect(() => {
        setLoading(true)
        setTimeout(()=> {
            setLoading(false)
            setPerson({name})
        },2000)
    },[name])
    return [loading,person]
}
const AsyncPage = (name)=> {
    const [loading,person] = usePerson(name)
    return (
        <>
            {loading?<p>Loading...</p>:<p>{ person.name }</p>}
        </>
    )
}

const PersonPage = ()=> {
    const [state,setState] = useState('')
    const changeName = (name)=> {
        setState(name)
    }
    return (
        <>
            <AsyncPage name={ state } />
            <button onClick={ ()=> { changeName('郭靖')}}>郭靖</button>
            <button onClick={ ()=> { changeName('黄蓉')}}>黄蓉</button>
        </>
    )
}
export default PersonPage;

```

上面代码中，封装成了自己的Hooks，便于共享。其中，usePerson()为自定义Hooks它接受一个字符串，返回一个数组，数组中包括两个数据的状态，之后在使用usePerson()时，会根据传入的参数不同而返回不同的状态，然后很简便的应用于我们的页面中。

这是一种非常简单的自定义Hook。如果项目大的话使用自定义Hook会抽离可以抽离公共代码，极大的减少我们的代码量，提高开发效率。