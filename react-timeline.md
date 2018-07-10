# React timeline

**How react nested component to be rendered**

1. We all know the well-known react lifecyle [Link image here]

2. The question is how the nested component to be rendered, for example a component `MyComponent` which the `render` function return:

```jsx
<MyParent>
  <MyChild />
</MyParent>
```

What is the order of `MyParent` and `MyChild` lifecycle?

The easiest way to know that is place the dummy `console.log` inside the declaration of parent and child components:
**The parent component:**

```jsx
class MyParent extends Component {
  componentWillMount() {
    console.log('parent, will mount');
  }

  componentDidMount() {
    console.log('parent, did mount');
  }
  
  componentWillUnMount() {
    console.log('parent, will unmount');
  }

  render() {
    console.log('parent, render');
    ...
  }
}
```
**The child component:**

```jsx
class MyChild extends Component {
  componentWillMount() {
    console.log('child, will mount');
  }

  componentDidMount() {
    console.log('child, did mount');
  }
  
  componentWillUnMount() {
    console.log('child, will unmount');
  }

  render() {
    console.log('child, render');
    ...
  }
}
```

**The represent component:**
```jsx
<MyParent>
  <MyChild />
</MyParent>
```

When we invoke the `MyComponent`, we found the log like that:
```
parent, constructor
parent, will mount
parent, render

child, constructor
child, will mount
child, render

child, did mount
parent, did mount
```

Does this result surprise you? I have placed the line break to make it clearer.
From the result log we found that the parent component starts it 3 beginning phase: `constructor`, `willMount`, `render`, then waiting the child mounting process before finishing the parent mounting process:

`parent DOM preparation` - `child DOM preparation` - `child DOM mounted` - `parent DOM mounted`


How we can explain this result? Think about function calling another function inside it.

When we render the `MyComponent`, first it meets the `MyParent` invocation, which will go through `constructor`, `componentWillMount`, and `render` function. In the `render` function of the `MyParent` component, it meets the invocation of `MyChild`, go through the `MyChild` mounting process until it meets the legacy component (`div` for example). The mounting process endes when the `MyParent` finishes mounting and calling `componentDidMount` callback.

In the unmounting process, the result is very similar: First, it calls the `componentWillUnmount` of the parent component and then the `componentWillUnmount` of the child component. The unmouting process ends after the child then the parent finish unmounting.

3. The meaning to know the order lifecycle of the nested component:
* Understanding the `ref` callback:
The `ref` callback is invoked before the `componentDidMount` and call with `null` in the `componentWillUnMount`.
* Subcribe parent callback safely: When you want to invoke the parent component `setState` intervally which the timer is declared inside the child component. The callback may lead to call the `setState` function after calling the `componentWillUnmount` function because:
  - The parent's `componentWillUnmount` is called before the child's `componentWillUnmount`.
  - The timer is declared in the child component and removed in the child's `componentWillUnmount`, which means it is still alive, and the callback parent `setState` maybe called after the parent's `componentWillUnmount` invoked.