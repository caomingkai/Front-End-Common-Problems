---
title: Flux & Redux
comments: true
date: 2018-10-04 21:04:19
type: categories
categories: Full stack
tags: 
- Flux
- Redux
- ReactJS
---

## I. Flux

1. **4 parts of Flux architecture:**
   - View
   - Actions
   - Dispatcher
   - Store

2. **Relationship**

   - **Store:** model object managing all data; has updating methods as well
     - provide data to <u>View</u>
     - provide updating methods to <u>Actions</u>
   - **Dispatcher:  **extends `Event Emitter object`
     - used by component to <u>register event & callbacks</u>
     - used by component to <u>emit actions to call store updating methods</u>
   - **View:** receive data from store
   - **Actions: **used by <u>*component*</u> to let <u>*dispather*</u> <u>emit actions</u>
   - Great article: http://www.ruanyifeng.com/blog/2016/01/flux.html


## II. Redux

1. **`store`** 

   1. `getState()`: source of data
   2. `dispath()`: emit actions, execute <u>reducer</u> to update state
   3. `subscribe()`: component subscribe to receive updated state

2. **reducer**

   - like`emitter.on`, used to register all the <u>updating functions</u> for related <u>actions</u>
   - pure function
   - `combineReducers()`: used to make logic clean

3. **Store implementation**

   1. How to use the `store` object

   2. ```js
      import { createStore } from 'redux';
      let { subscribe, dispatch, getState } = createStore(reducer);
      // or 
      let store = createStore(todoApp, window.STATE_FROM_SERVER)
      store.getState()
      store.dispatch()
      store.subscribe()
      ```

   3. redux provide an interface `createStore()` to create the `store` object

   4. ```js
      const createStore = (reducer) => {
        let state;
        let listeners = [];
      
        const getState = () => state;
      
        const dispatch = (action) => {
          state = reducer(state, action);
          listeners.forEach(listener => listener());
        };
      
        const subscribe = (listener) => {
          listeners.push(listener);
          return () => {
            listeners = listeners.filter(l => l !== listener);
          }
        };
      
        dispatch({});
      
        return { getState, dispatch, subscribe };
      };
      ```

4. **An example using Redux**

5. ```js
   const Counter = ({ value, onIncrement, onDecrement }) => (
     <div>
     <h1>{value}</h1>
     <button onClick={onIncrement}>+</button>
     <button onClick={onDecrement}>-</button>
     </div>
   );
   
   const reducer = (state = 0, action) => {
     switch (action.type) {
       case 'INCREMENT': return state + 1;
       case 'DECREMENT': return state - 1;
       default: return state;
     }
   };
   
   const store = createStore(reducer);
   
   const render = () => {
     ReactDOM.render(
       <Counter
         value={store.getState()}
         onIncrement={() => store.dispatch({type: 'INCREMENT'})}
         onDecrement={() => store.dispatch({type: 'DECREMENT'})}
       />,
       document.getElementById('root')
     );
   };
   
   render();
   store.subscribe(render);
   ```


## III. React-Redux package

1. `connect()`: 
   - connect <u>UI component</u> with it outer <u>Controller component</u>
   - usage: `const Contrainer = connect(mapStateToprops, mapDispatchToProps)(UIComponent)`
2. `mapStateToProps()`
   - map `state` in Redux store to UI 
   - like `eventEmitter.subscribe()`, will always get the updated state from Redux
3. `mapDispatchToProps`
   - map `actions dispatch()` in Redux to UI
   - like `eventEmitter.emit()`, will trigger actions when UI event is fired, to execute reducer to update the state

4. `<Provider>` component
   - an outer container, used to pass the `state` to inner components
   - using `context` attribute

