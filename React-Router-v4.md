---
title: React Router v4
comments: true
date: 2018-05-15 21:21:05
type: categories
categories: Full stack
tags: React-Router
---


### I. Three types of Components：
**1. router components**
>create a history object, 提供“前进”、“后退”

- **`<BrowserRouter>`**: for server that responds to requests
- **`<HashRouter>`**: for a static file server.

**2. route matching components**
>comparing a <Route>'s path prop with the current location’s pathname  
>只要路由匹配，Route所在位置就会render相应的Component

- **`<Route>` ：包容性路由**
    - 3个props： 
        - path：匹配路径
        - component：对应的组件
        - render：没有可用组件时，直接render
  - 1个state：
        - match
        - match包含4个field：`isExact`,`params`,`path`,`url`
  - 1个context：
    - router obj
    - router 包含 `history`,`routes`

- **`<Switch>`：排他性路由**

    ```javascript
    <Switch>
        <Route exact path='/' component=	{Home}/>
        <Route path='/about' component={About}/>
        <Route path='/contact' component={Contact}/>
        <Route component={NoMatch}/>
    </Switch>
    
    // when none of the above match, <NoMatch> will be rendered 
    ```

  - How ？  
  iterate over all of its children <Route> elements and only render the first one that matches the current location.
  - Props: `component`, `render`, and `children`

**3. navigation components**

- **`<Link> `**
- **`<NavLink> `** :a special type of <Link> that can style itself as “active” when its ‘to’ prop matches the current location.
- **`<Redirect> `**: navigate using its ‘to’ prop
  - 注意：不是 “函数”，而是“组件”，需要放在render中return
  - 没有路由解析时候，可以直接使用redirect，重定向到默认页面

### II. 3种方式renders Component

- `<Route component>`
- `<Route render>`
- `<Route children>`

### III. Route在render component同时，传入3个量，作为component的props（不论3中方式的哪一种）
- match
- history
- location

比如render:func( )方式render conponent
 
```javascript
// eg1: 
  <Route path="/home" render={() => <div>Home</div>}/>
  
  // eg2: wrapping/composing
  const FadingRoute = ({ component: Component, ...rest }) => (
    <Route {...rest} render={props => (
      <FadeIn>
        <Component {...props}/>
      </FadeIn>
    )}/>
  )
  
  <FadingRoute path="/cool" component={Something}/>"
  - render props: path、children／render／component、
  - render state：match
  - component props(route所render): history、location、match
    "不论是哪种方式render，都会拿到这三个props"
  - 因此，eg2中render:func方式，直接将props作为function component的参数传入
    "render={ props => (
        <FadeIn>
          <Component {...props}/>
        </FadeIn>
      )}"
```

### IV. Route vs Component如何关联
  - Router 将路径 match上之后，会调用相应的 Route， 由Route 将相关的参数：match、location、history等传入到 Component中
  - Route 的 state: match、location 传入到 Component的 props: match、location
  - The Route will render <Dashboard {...props}/> , where props are some router specific things that look like { match, location, history }. 
  - If the user is not at /dashboard then the Route will render null. That’s pretty much all there is to it.

### V. `match.url` vs `match.path`
>A match object contains information about how a <Route path> matched the URL.

  - match：是 `route`的`state`；是`component`的`props`
  - Eg: consider the route `/users/:userId`
      - `match.path` - (string) 用于匹配路径模式。用于构建嵌套的 <Route>; `match.path` would be `/users/:userId`
      - `match.url` - (string) URL 匹配的部分。 用于构建嵌套的 <Link>; `match.url` would have the :userId value filled in, e.g. "users/5"."
    
### VI. `match.params` 表示 url 匹配上的变量值

```javascript 
<Route path="/:id" component={Child} />

{match.params.id}
   
```
  
URL 后半部分由变量 id 兜住，如果进来的url为`/abc`，那么 `match.params.id = abc`

### VII. Eg: Custom Link
>Route实际上是 Component 的买办，为Component办事！  
>Route放在哪个位置，那里就有可能render出它携带的component(只要url match)

```javascript
<Route path="/abc" component={Child_1} />
<Route path="/abc" component={Child_2} />
<Route path="/abc" component={Child_3} />
```

### VIII. `<Prompt>`：防止中途跳转
>适用情况：用户在填表过程中，点击了“后退”或“跳转链接”，需弹窗询问  
>`<Prompt>`是保留tag，跟`<Link>`一样

- 创建一个wrapper，包含`<form>`, `<Prompt>`
- wrapper中设置`state.inMiddle`，来决定是否render`<Prompt>`
- 改变`state.inMiddle`的根源：`form`中的`input`输入框；
    + 只要有任何输入值，就将`state.inMiddle`置为`true`
    + 提交form后，将form reset()之后，需要将`state.inMiddle`置为`false`
- `Prompt`的props:
    + `when`：检查`state.inMiddle`
    + `message `: 决定弹窗的显示信息

```javascript
class Form extends React.Component {
  state = {
    isBlocking: false
  };

  render() {
    const { isBlocking } = this.state;

    return (
        <form
        onSubmit={event => {
          event.preventDefault();
          event.target.reset();
          this.setState({
            isBlocking: false
          });
        }}
        >
        
        <Prompt
          when={isBlocking}
          message={location =>
            `Are you sure you want to go to ${location.pathname}`
          }
        />

        <input
        placeholder="type something"
        onChange={event => {
          this.setState({
            isBlocking: event.target.value.length > 0
          });
        }}
        />
        
        
        <button>Submit</button>

      </form>
    );
  }
}
```


### Reference

- Official API: [https://reacttraining.com/react-router/web/guides/philosophy](https://reacttraining.com/react-router/web/guides/philosophy)
- 中文版API：[https://www.jianshu.com/p/e3adc9b5f75c](https://www.jianshu.com/p/e3adc9b5f75c)
