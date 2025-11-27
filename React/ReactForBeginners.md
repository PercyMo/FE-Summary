## React 新手入门

### 一. 一级标题a

### 二. 一级标题b

#### 1. 二级小标题


### 待整理
* props 是否可以直接解构，是否影响响应式？
```js
const BeautySettings: FC<any> = ({sceneItemMeta = {}}) => {
};
```

* useMemo，传个空数组，这种后续还能被监听变动执行吗？
```js
// 监听的数组中也可以传函数？
const params = useMemo(() => (new URL(location.href)).searchParams, []); ;

// getOS() 返回当前系统，应该是不可变的，为什么还要使用useMemo计算生成
const isWindows = useMemo(() => getOS() === OS.Windows, []);
```

* useEffect的用法
1. 在组件初次渲染就会调用一次
```js
useEffect(() => {

    const removeListener = window.electron.ipcRenderer.on('modal:contributionDataChange', data => {
        changeStoreStatus(data);
    });

    // return的函数会在什么时机执行，组件卸载时执行？
    return () => {
        removeListener();
    };

    // 监听的参数变化，传参是useMemo的返回值，是个函数？
}, [changeStoreStatus, params]);
```
2. 使用useEffect模拟组件mounted和直接写在函数里，什么区别
直接，页面点击+1，函数也会被执行？写在useEffect里不会？
函数组件，当有状态变更时，函数会被重新执行。
逻辑如果直接写在函数里，在组件运行期间，可能会被多次执行，浪费性能。

* useCallback的用法


* useImperativeHandle的用法
React中父组件如何调用子组件的方法
useRef()无法使用在函数组件上使用，在函数组件中要先使用 useImperativeHandle 定义要暴露给父组件的实例值，另外要把函数组件传入forwardRef处理后再导出。

* useLayoutEffect的用法

* 自定义Hooks和普通函数的区别
    * 可以调用官方提供的Hooks
    * 规范约定中以use开头