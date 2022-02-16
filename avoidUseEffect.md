
# Why not to use useEffect (so frequently)

## TLDR;

In my React carieer I found that programmers do very magic code in name of `useEffect`.

Unfortunatelly `useEffect` can be good servant but a bad master because if you overuse useEffect your code starts to be `Reactive` (similar as rxjs). We will talk more about reactivity bellow.

On the other hand useEffect can be really nice colution for more complex problem but I found that in reality it's minor scenario

Every problem in programming can have nice solution so I'll show you my tool to remove 80% of useEffect.
I will show you how usefull hook `useAsyncState` which may handle tricky async states in the React apps because linear sequence of code is much more readable, than event-driven reactive architecture.

Let's start from beginning. Why people use useEffect.

## Where's the snag?

Problem with `useEffect` is much deeper that you may thought. There are two main useCases when you use it.

### 1. componentDidMount for component

this is good and you should continue with using useEffect like this

I Advice you to create custom hook `useComponentDidMount` and it may save you from creating some unwanted useEffect dependencies.

```tsx
const useComponentDidMount = (fn: Parameters<typeof useEffect>[0]) => {
  useEffect(fn, [])
}
```


### 2. waiting to some external event like changing Component properties

This is good usage of `useEffect` in general but you may omit it in many cases just by using it with different architecture of your React state or different API of your functions.

for example:
  - synchronizing parent component state with the local component 
  - wait till some state will be chnaged

### 3. Do Action after React state is changed

This kind of `useEffect` is not good and you should avoid of using it.

This is the point we will be talk about.

for example:
  - Calling sequence of async callbacks.
  - Apply requst to the server after the variable is changed
  - Sequence of changes with async prevState callback

Problem with this kind of usage of useEffect is that it brings high level of code complexity.
Instead of just simple calling linear sequence of functions.
With useEffect you create some sort of reactive event-driven architecture where you define dependencies and call your code every time when some interal/external variables is changed.
In the small codebases it looks really elegant and nice but in the larger and big codebases you start to be scared to change variable
because you have no idea what will going to happen because you don't know 100% of codebase
so you have no idea if someone *registered* some useEffect change listerner into your variable.

This behavior may be handy for really complex use cases in complex architectures but in general I found that 90% of useCases are not good and may be removed from the codebase.


Let's solve all different use cases where people use `useEffect`

## onChange vs useEffect

First of all there is example of basic use case which I found in many programming project

useEffect code


```tsx
// TODO: add async alertValue to demonstrate call to the server
const Component = () => {
  const [couter, setCounter] = useState(0)
  const alertValue = () => alert(`current counter value is: ${counter}`)

  useEffect(() => {
    doSomething()
  }, [counter])

  return (
    <button onClick={() => {
      setCounter(p => p + 1)
      setCounter(p => p + 1)
    }}>
      +1
    </button>
  )
}
```

it will be much more readable if we'll have linear sequence of code like this

```tsx
const Component = () => {
  const [couter, setCounter] = useState(0)
  const alertValue = () => alert(`current counter value is: ${counter}`)

  return (
    <button onClick={() => {
      setCounter(p => p + 1)
      doSomething()
    }}>
      +1
    </button>
  )
}
```

Problem is that this example will not work because change of React state is asynchronouse.

We will handle that issue in a while. First look how does reand

## React change of state is asynchronouse?

the change of React state is asynchronous so if you write code like this.

```tsx
const [counter, setCounter] = useState(0)

/* ... */
setState(counter + 1)
```

Everything is working properly until you call it two times without waiting to the new render cycle.


```tsx
const [counter, setCounter] = useState(0)

/* ... */
setState(counter + 1)
setState(counter + 1)
```

As we may know React state change is asynchronous action so if you call `setState(counter + 1)` it will not mutate `counter` variable.
If you call `setState` synchronously for the second time `counter` variable has still old value.
So the code what you're looking for is bugged.

You may fix it with usage of `prevState` callbacks.

```tsx
const [counter, setCounter] = useState(0)

/* ... */
setState(prevCounterValue => prevCounterValue + 1)
setState(prevCounterValue => prevCounterValue + 1)
```

Now, everything is working correctly but the code is kinda more complicated.

### Why is React state changing asynchronous???
TODO......:::::::

## screenshot of code with useEffect vs with linear sequence of commands
TODO: add screenshot of code organisation with usage of `useEffect`


## useAsyncState

For use cases when you want to have linear async sequence of calls which handle states and you want to avoid useEffect we will create new custom React hook called `useAsycState`

```tsx

const useAsyncState = <T>(defaultState: T) => {
  const [state, __setState] = useState(defaultState);

  const setState = (setStateAction: Parameters<typeof __setState>[0]) =>
    new Promise<T>(res =>
      __setState(prevState => {
        const newState =
          setStateAction instanceof Function ? setStateAction(prevState) : setStateAction;
        res(newState);
        return newState;
      })
    );

  const getPrevState = () => setState(p => p);

  return [state, setState, getPrevState] as const;
};

```


## Render optimisation and batching React changes

TODO: test
