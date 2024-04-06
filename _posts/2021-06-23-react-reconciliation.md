---
title: "Post: React Reconciliation"
last_modified_at: 2021-06-16T12:58:39+0900
categories:
  - Blog
featuredPost: true
tags:
  - React
  - Reconciliation
  - Virtual DOM
---

# Overview

Although React is often introduced as a UI framework, the reason React has become a dominant framework recently is not
merely due to its UI capabilities. React differentiates itself from other frameworks through its efficient[^1] support
for reactive, declaratory programming via the Virtual DOM. Understanding the process called Reconciliation is essential
to understand how React supports efficient declarative programming.

Before we begin, I’d like to clarify that this post will primarily focus on class components. Hooks will be covered
later.

# Reactive, Declarative programming

First, let's briefly explain what declarative programming is. Suppose you're developing a UI in the traditional
imperative programming paradigm. Initially, this page would start by drawing an empty space. After the basic UI is
drawn, it would call an API to fetch whether the weather is clear or cloudy and write that single word in the previously
empty space.

The UI development with the imperative methodology would proceed as follows:

1. Load the page.
2. Call the API.
3. Upon receiving the response from the API, find the element where the weather value needs to be displayed.
4. Update the element with the weather value.

On the other hand, UI development in declarative programming is as follows:

1. Here's an empty space, and this space will contain the value of a variable called weather.
2. When the response from the API arrives, put the received weather value into weather.
3. ???
4. Profit!!

Even with this simple example, no application is this simple in reality. Developers have a lot on their plate, and
considering the exceptions and sequence in the UI controlling process is quite bothersome. React, which applies
declarative programming, can significantly simplify these processes. It allows developers to focus solely on the flow
and changes of data, rather than on how to display data on the screen. Declarative programming is the opposite of the
imperative programming paradigm, and reactivity is a way to apply this declarative programming paradigm (in my opinion).
It re-renders the part of the rendering result that changes due to the change in value.

React constructs the UI in units called components. Components have a dictionary called props, which corresponds to
attributes in traditional DOM elements, and another dictionary called state, which can be changed within each component.
Props are modified by the parent component, while state is modified through a method called setState within the
component.
When either props or state changes in a component, and thus the rendering result of that component instance differs,
React updates only the DOM that was being drawn by that specific component, thereby performing efficient and declarative
UI rendering. The process doesn't immediately update the rendering result on the DOM but rather represents it virtually
in memory and only updates the changed parts based on the comparison. This pattern is known as the virtual DOM.

# So how does React render?

As mentioned before, React re-renders the relevant part automatically when developers update props or state, and the
rendering result changes. However, as everyone knows, there is no magic in development. What exactly does React do to
render so efficiently?
First, let's understand what happens in the render method. In the render method, we write the UI using jsx[^2] syntax.
Famously, this jsx code is transpiled into JS code that hierarchically lists React.createElement. This
React.createElement takes in the React element you want to create, the props, and the children.

```javascript
React.createElement('div', {class: 'container-div'}, 'Hello ${someVar}');
```

Unlike document.createElement, React.createElement doesn't return a DOM but a React element. Note that it returns not a
component but a React element. React.createElement can receive a tag name as a string or a component type or fragment
type as an argument. React elements can contain child elements, therefore can be considered tree-shaped.
The friend that renders this to the web page is ReactDOM. In other words, render is a method that creates a tree
composed of React elements, and the created tree is rendered to the designated DOM using ReactDOM.render(). If you think
about this structure, can you roughly imagine how React was ported to other platforms like React Native?

Anyway, this React element is an immutable object. Once created, its children or properties cannot be changed through a
specific method. The official documentation describes it as a snapshot of a specific moment in the UI changing process,
like frames in a movie.

The secret to React being able to redraw only the changed parts is here. It compares the snapshot tree at time A and the
snapshot tree at time B to identify and update the changed specific sections. This process is called Reconciliation.

# Reconciliation

So, we've learned that React diligently compares trees for us whenever props or state change. According to the official
documentation, even with very efficient tree comparison, transforming a tree with n elements has a complexity of O(n^3).

Therefore, when implementing Reconciliation, React uses a heuristic algorithm to achieve an algorithm with a complexity
of O(n).
Two prerequisites are required for applying this algorithm:

1. Two elements of different types will produce different trees.
2. Developers can indicate which child elements should not change using the key prop.

These assumptions seem reasonable. The first is mostly true, and the second is often related to a common mistake made
when adding components iteratively using React - the key prop issue. To put it conversely, React applications that do
not adhere to 1 and 2 are considered to follow an anti-pattern.

To summarize, the Reconciliation algorithm of React is as follows:

1. Comparisons start simultaneously from the root element.
2. If the element types differ? Through the assumption 1, it can be inferred that there has been a change. The former
   DOM nodes are discarded, and new DOM nodes are inserted.
3. If the DOM element types are the same, compare the element attributes and update the changed attributes. Then
   recursively reconcile the children.
4. If the component element types are the same. It gets a bit more complicated here. The component is updated but
   retains the instance. That means, the state remains, and after updating the props, it calls
   UNSAFE_componentWillReceiveProps(), UNSAFE_componentWillUpdate(), componentDidUpdate() callback methods in sequence.
   And by calling the render() method, it creates a new tree and compares again.

If recursively reconciling children and encountering elements with sibling nodes,

```html

<ul>
    <li>first</li>
    <li>second</li>
</ul>

<ul>
    <li>first</li>
    <li>second</li>
    <li>third</li>
</ul>
```

In the case where the tree changes as above, React would normally handle the change well after comparing up to "second"
and encountering the previously non-existent "third". However, in the example case...

```html

<ul>
    <li>Duke</li>
    <li>Villanova</li>
</ul>

<ul>
    <li>Connecticut</li>
    <li>Duke</li>
    <li>Villanova</li>
</ul>
```

Although we intuitively know that only the first element was added to the original tree, because React goes through each
child node sequentially, it perceives that all child nodes have changed. This is why setting the key prop is necessary.

```html

<ul>
    <li key="2015">Duke</li>
    <li key="2016">Villanova</li>
</ul>

<ul>
    <li key="2014">Connecticut</li>
    <li key="2015">Duke</li>
    <li key="2016">Villanova</li>
</ul>
```

In cases like this, where the key prop is set, by comparing elements with the same key sequentially, React realizes that
elements with keys 2015 and 2016 just need to be moved.

Although the algorithm is understandable, one of React's common anti-patterns, the key prop error, can lead to serious
performance degradation in React. Therefore, setting the key prop correctly is more important than anything. Suppose
we're developing an application with React and calling a user search API. Let's say the search API takes a sorting key
as a parameter and includes an index for the sorting order in the results. Carelessly using the sorting order index as a
key can lead to re-rendering of all child nodes when changing the sorting by calling the same search conditions.

The best practice is to consider whether it can be designated as a primary key when designing the DB. In the
aforementioned example, using a user ID that does not allow duplicates as the key prop would be ideal. If that's not
feasible, a method of combining multiple data into a hash, much like creating a composite key, is also possible.

Furthermore, upon a detailed examination of this methodology, one can realize that there are ways to induce inefficient
rendering. The official documentation introduces anti-pattern cases as below:

1. When an element moves not between siblings but to a completely different element
2. When rendering results are very similar but change to a different component type

# Fiber

The Reconciliation process discussed above is quite abstract for something claimed to lack magic. This process is an
algorithm, not an implementation. It's more of a Principal, in a sense.

The actual implementation, the reconciler, was redeveloped in React 16, and this new Reconciler is called React Fiber.

The previous implementation used the call stack, meaning the recursive call of the render method for the updated tree
occurred all at once. What it means to have a long call stack in JavaScript has been discussed in a previous post.
Therefore, higher-priority operations like animations, gestures[^3], etc., could be delayed, resulting in dropped
frames. Conversely, areas off-screen that could be rendered lazily were pushed back due to important higher-priority
tasks.[^4] React 16 introduced a new Reconciler, React Fiber.

To implement priorities and lazy rendering, the following must be possible:

* Must be able to suspend work and return to it later.
* Must be able to assign priorities to different types of work.
* Must be able to reuse results from previously completed work.
* Must be able to abort work if it is no longer needed.

In [React Fiber](https://github.com/acdlite/react-fiber-architecture), a single Fiber refers to a unit of work. In other words, it corresponds to a stack frame in the call stack and can be thought of as a virtual stack frame. To be more precise, it is a JavaScript object that holds components, input values, and output values. Even though it corresponds to a stack frame, it also pertains to an instance of a component. Fiber does not use a stack and is structured as a tree-shaped linked list.

Before starting, let's remember that in React, the UI is based on the design concept that it always has the same output for the same input values [^5]. That is, the premise is that if you render with the same props and state, you will always get the same value.

Among the values that fiber holds, some are as follows:

1. type, key
2. child, sibling
3. return
4. pendingProps, memoizedProps
5. pendingWorkPriority
6. alternate
7. output

## type, key

As explained, corresponds to the type and key of a React element.

## child, sibling

Fields that describe the recursive tree structure of fiber. It can be thought of as the return value of the render
method. When returning one it's a child, and if multiple, they’re siblings.

## return

The return fiber refers to the fiber that should be returned to after processing the current fiber. In terms of the call
stack, it refers to the caller.

## pendingProps, memoizedProps

Represent the props before and after the execution of fiber. If they are the same, it could mean the output is the same
as before and that the previous result could be reused.

## pendingWorkPriority

A number indicating the priority of the work represented as a fiber. Excluding NoWork, represented by 0, a larger number
indicates a lower priority.

## alternate

Every component instance can have at most two fibers: current, flushed, work-in-progress. Flush means flushing out the
fiber's output to render it on the screen, and work-in-progress corresponds to an unfinished stack frame in the stack.
Current is the current screen-composing fiber tree, while work-in-progress is a draft reflecting new props and state
changes. Alternate is like a counterpart between these two trees. To create a tree with the newly applied information,
it creates alternate nodes for each node from the current and renders it to make the work-in-progress tree become the
new current.
The process of creating alternate nodes is done lazily using a function called cloneFiber, and it minimizes allocation
operations by reusing the existing alternate rather than always creating new objects.
Therefore, if the current work-in-progress tree has finished rendering and becomes the current tree, and props or state
change again without adding new nodes, the original friends, which were initially the work-in-progress and now the
current tree, do not get reallocated as they are still the alternates of the current nodes.

## output

In React's fiber tree, the leaf nodes may vary depending on the rendering environment, but for web, it would be DOM
elements. These are referred to as host components.
The output in fiber would be the return value of the render function. If one follows the work-in-progress tree to the
end, the host component will emerge, and there’s no need to recreate the results. If the host component stores the
outcome in the output property, the parent fiber can reflect this in its output.
Of course, React's design separates UI technology and rendering environments, so the output's actual results are
entirely the renderer's responsibility.

## deep dive

Still feeling a bit abstract? In such cases, I like to simply put a breakpoint in the source code and explore.
Let's take a slight variation of this example and look at a transpiled example of a class component here.

```javascript
    class LikeButton extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            liked: false
        };
        this.handleClick = this.handleClick.bind(this);
    }

    handleClick() {
        this.setState({
            liked: !this.state.liked
        });
    }

    render() {
        let text = this.state.liked ? 'like' : 'haven\'t liked';
        return React.createElement("p", {
            onClick: this.handleClick
        }, "You ", text, " this. Click to toggle.");
    }
}

ReactDOM.render(React.createElement(LikeButton, null), document.getElementById('example'));
```

Place a breakpoint inside the render and travel up the call stack once. You'll encounter a function named
finishClassComponent.

```javascript
    function finishClassComponent(current, workInProgress, Component, shouldUpdate, hasContext, renderLanes) {
    // Refs should update even if shouldComponentUpdate returns false
    markRef(current, workInProgress);
    var didCaptureError = (workInProgress.flags & DidCapture) !== NoFlags;
    if (!shouldUpdate && !didCaptureError) {
        // Context providers should defer to sCU for rendering
        if (hasContext) {
            invalidateContextProvider(workInProgress, Component, false);
        }
        return bailoutOnAlreadyFinishedWork(current, workInProgress, renderLanes);
    }
    var instance = workInProgress.stateNode; // Rerender
    ReactCurrentOwner$1.current = workInProgress;
    var nextChildren;
    if (didCaptureError && typeof Component.getDerivedStateFromError !== 'function') {
        // If we captured an error, but getDerivedStateFromError is not defined,
        // unmount all the children. componentDidCatch will schedule an update to
        // re-render a fallback. This is temporary until we migrate everyone to
        // the new API.
        // TODO: Warn in a future release.
        nextChildren = null;
        {
            stopProfilerTimerIfRunning();
        }
    } else {
        {
            setIsRendering(true);
            nextChildren = instance.render();
            if (workInProgress.mode & StrictMode) {
                disableLogs();
                try {
                    instance.render();
                } finally {
                    reenableLogs();
                }
            }
            setIsRendering(false);
        }
    } // React DevTools reads this flag.
    workInProgress.flags |= PerformedWork;
    if (current !== null && didCaptureError) {
        // If we're recovering from an error, reconcile without reusing any of
        // the existing children. Conceptually, the normal children and the children
        // that are shown on error are two different sets, so we shouldn't reuse
        // normal children even if their identities match.
        forceUnmountCurrentAndReconcile(current, workInProgress, nextChildren, renderLanes);
    } else {
        reconcileChildren(current, workInProgress, nextChildren, renderLanes);
    } // Memoize state using the values we just used to render.
    // TODO: Restructure so we never read values from the instance.
    workInProgress.memoizedState = instance.state; // The context might have changed so we need to recalculate it.
    if (hasContext) {
        invalidateContextProvider(workInProgress, Component, true);
    }
    return workInProgress.child;
}
```

Let's stop poring over the entire thing and focus from the middle part:

```javascript
        setIsRendering(true);
nextChildren = instance.render();
if (workInProgress.mode & StrictMode) {
    disableLogs();
    try {
        instance.render();
    } finally {
        reenableLogs();
    }
}
setIsRendering(false);
...
if (current !== null && didCaptureError) {
    // If we're recovering from an error, reconcile without reusing any of
    // the existing children. Conceptually, the normal children and the children
    // that are shown on error are two different sets, so we shouldn't reuse
    // normal children even if their identities match.
    forceUnmountCurrentAndReconcile(current, workInProgress, nextChildren, renderLanes);
} else {
    reconcileChildren(current, workInProgress, nextChildren, renderLanes);
} // Memoize state using the values we just used to render.
...
return workInProgress.child;
```

The flag `isRendering` (probably acting as a lock) is raised, and the current instance is rendered. This is temporarily
saved as a variable called `nextChildren`, and if no error has occurred, the `reconcileChildren` function is called,
after which the child of the `workInProgress` fiber is returned.

```javascript
  function reconcileChildren(current, workInProgress, nextChildren, renderLanes) {
    if (current === null) {
        // If this is a fresh new component that hasn't been rendered yet, we
        // won't update its child set by applying minimal side-effects. Instead,
        // we will add them all to the child before it gets rendered. That means
        // we can optimize this reconciliation pass by not tracking side-effects.
        workInProgress.child = mountChildFibers(workInProgress, null, nextChildren, renderLanes);
    } else {
        // If the current child is the same as the work in progress, it means that
        // we haven't yet started any work on these children. Therefore, we use
        // the clone algorithm to create a copy of all the current children.
        // If we had any progressed work already, that is invalid at this point so
        // let's throw it out.
        workInProgress.child = reconcileChildFibers(workInProgress, current.child, nextChildren, renderLanes);
    }
}
```

Very clearly, in the case where the current tree is not present (initialization status), we `mount`, and in the case
where it is present, we go with a function called `reconcileChildFibers`.

```javascript
    function reconcileChildFibers(returnFiber, currentFirstChild, newChild, lanes) {
    // This function is not recursive.
    // If the top level item is an array, we treat it as a set of children,
    // not as a fragment. Nested arrays on the other hand will be treated as
    // fragment nodes. Recursion happens at the normal flow.
    // Handle top level unkeyed fragments as if they were arrays.
    // This leads to an ambiguity between <>{[...]}</> and <>...</>.
    // We treat the ambiguous cases above the same.
    var isUnkeyedTopLevelFragment = typeof newChild === 'object' && newChild !== null && newChild.type === REACT_FRAGMENT_TYPE && newChild.key === null;
    if (isUnkeyedTopLevelFragment) {
        newChild = newChild.props.children;
    } // Handle object types
    var isObject = typeof newChild === 'object' && newChild !== null;
    if (isObject) {
        switch (newChild.$$typeof) {
            case REACT_ELEMENT_TYPE:
                return placeSingleChild(reconcileSingleElement(returnFiber, currentFirstChild, newChild, lanes));
            case REACT_PORTAL_TYPE:
                return placeSingleChild(reconcileSinglePortal(returnFiber, currentFirstChild, newChild, lanes));
        }
    }
    if (typeof newChild === 'string' || typeof newChild === 'number') {
        return placeSingleChild(reconcileSingleTextNode(returnFiber, currentFirstChild, '' + newChild, lanes));
    }
    if (isArray$1(newChild)) {
        return reconcileChildrenArray(returnFiber, currentFirstChild, newChild, lanes);
    }
    if (getIteratorFn(newChild)) {
        return reconcileChildrenIterator(returnFiber, currentFirstChild, newChild, lanes);
    }
    if (isObject) {
        throwOnInvalidObjectType(returnFiber, newChild);
    }
    {
        if (typeof newChild === 'function') {
            warnOnFunctionType(returnFiber);
        }
    }
    if (typeof newChild === 'undefined' && !isUnkeyedTopLevelFragment) {
        // If the new child is undefined, and the return fiber is a composite
        // component, throw an error. If Fiber return types are disabled,
        // we already threw above.
        switch (returnFiber.tag) {
            case ClassComponent: {
                {
                    var instance = returnFiber.stateNode;
                    if (instance.render._isMockFunction) {
                        // We allow auto-mocks to proceed as if they're returning null.
                        break;
                    }
                }
            }
            // Intentionally fall through to the next case, which handles both
            // functions and classes
            // eslint-disable-next-lined no-fallthrough
            case Block:
            case FunctionComponent:
            case ForwardRef:
            case SimpleMemoComponent: {
                {
                    {
                        throw Error((getComponentName(returnFiber.type) || 'Component') + "(...): Nothing was returned from render. This usually means a return statement is missing. Or, to render nothing, return null.");
                    }
                }
            }
        }
    } // Remaining cases are all treated as empty.
    return deleteRemainingChildren(returnFiber, currentFirstChild);
}

return reconcileChildFibers;
}
```

Here, when `$$typeof` is a React element, we put the result of `reconcileSingleElement` into `placeSingleChild` and
return it. `placeSingleChild` is a simple function that initializes in cases where there’s no alternate (new fiber).
In `reconcileSingleElement`, as described in the document, we use props to fetch a reusable fiber or create a new fiber
and return it.

Tracing back from `finishClassComponent`, this eventually leads to the `performUnitOfWork` function.

```
next = beginWork$1(current, unitOfWork, subtreeRenderLanes);
```

This `next` becomes the next fiber, and finally, if there is no `next`, it ends as below.

```javascript
    if (next === null) {
    // If this doesn't spawn new work, complete the current work.
    completeUnitOfWork(unitOfWork);
} else {
    workInProgress = next;
}
```

In `completeUnitOfWork`, the `setCurrentFiber` function is used to replace the current with a completed work-in-progress
tree.

# Conclusion

Thus, we have looked into React's reconciliation process to some extent. While I don't fully understand every deep
content, I hope this post helps in understanding.

[^1]: The term efficient here means rendering efficiency.
[^2]: In fact, jsx should be considered as syntactic sugar provided to alleviate the inconvenience of UI creation using `React.createElement`. The essence is ultimately in the React API!
[^3]: One might wonder why animations are among the operations with the highest priority. It's important to remember that the most crucial thing in the frontend is ensuring the user does not experience any jitters, emphasizing UX.
[^4]: Such priority issues occur not only in React but also in general JavaScript applications, and the Web API created to supplement this is [window.requestIdleCallback()](https://developer.mozilla.org/ko/docs/Web/API/Window/requestIdleCallback), [window.requestAnimationFrame()](https://developer.mozilla.org/ko/docs/Web/API/Window/requestAnimationFrame).
[^5]: If we pull this into a concept of functional programming, it's called a pure function.