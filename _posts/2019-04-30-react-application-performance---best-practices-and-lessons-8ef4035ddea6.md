---
layout:	"post"
categories:	"blog"
title:	"React application performance - Best practices and lessons"
date:	2019-04-30
thumbnail:	/img/1*B81H8GW7Vk3bUrfe-LZd3A.png
author:	
---

* * *

![](/img/1*B81H8GW7Vk3bUrfe-LZd3A.png)

> Original article posted at <https://botsplash.com/blog/react-application-
performance-best-practices-and-lessons-8ef4035ddea6.html>

[React](https://reactjs.org/) is one of the most widely used and popular
libraries that has seen great adoption from developers in comparison to other
libraries. It supports many out-of-the-box techniques to minimize the number
of costly DOM operations required to update the UI.

Though it automatically handles many of the heavy lifting under the hood, that
does not mean everything is done efficiently. We might be writing inefficient
code that is making React do unnecessary things and slowing things down
significantly.

If you are like us at [Botsplash](https://botsplash.com), who likes to develop
applications at rapid speed with great quality, you are likely to see few or
no performance issues. However, there are times, it is inevitable to self
reflect and audit the performance plan. Let's look at few lessons learned that
will help to get the best out of React and improve the application's
performance.

### 1\. Check for redundant component renderings

The render method inside a React component is called automatically when its
parent component's render is called or setState function is called inside the
component. Every time the render method is called, the JSX inside it is
evaluated along with its state and props.

There are cases when the component's state and props haven't changed but is
still forced to re-render. Let's take the following component for instance.

This is a really simple component which has an input and a button to set the
new message. There is child component that takes in the message as prop and
displays it. I've also added a variable called `childComponentRenderCount` to
keep track of the render count for the child component.

The problem arises when you start typing the new message in the text box. As
you can see, the render count keeps increasing on every keystroke. The main
component with the text input needs to be updated but there is no need for the
child component to update. It only needs to update when the submit button is
clicked.

This becomes a problem in the real world application where the component is
complicated and the child component also becomes nested and large. It is
therefore good to identify such unwanted renderings and cut down the unwanted
work React has to do. One way is to implement shouldComponentUpdate lifecycle
hook to cut down unnecessary renderings of the child component.

    
    
    ...  
    // inside ChildComponent
    
    
    shouldComponentUpdate(nextProps) {  
      return this.props.message !== nextProps.message;  
    }  
    ...

The lifecycle hook shouldComponentUpdate returns true by default and hence the
component is updated every time. But we can use it to add our own logic to
control when we want to re-render the component. We only want to re-render the
Child component when the message prop changes. Let's look at the difference
with shouldComponentUpdate implemented.

You can also use [PureComponent](https://reactjs.org/docs/react-
api.html#reactpurecomponent) which always shallowly compares the change in the
props and state and re-renders the component only when the change is detected.
We do not need to implement shouldComponentUpdate with PureComponent. Using
PureComponent is exactly the same like writing React.Component the following
way.

    
    
    class MyComponent extends React.Component {  
      shouldComponentUpdate(nextProps, nextState) {  
        return !shallowEqual(this.props, nextProps) ||  
              !shallowEqual(this.state, nextState);  
      }  
      ...  
    }

PureComponent should only be used when the component has simple props and
state as complex data structures could produce false-negatives as it only
shallowly compares the objects.

There are couple of ways we can identify these unwanted renderings in existing
application. If you are using React v15, there is an excellent tool called
[react-addons-perf](https://reactjs.org/docs/perf.html). It shows exactly how
many times a component rendered unnecessarily and how much time it wasted on
it. You can also use the [Performance](https://reactjs.org/docs/optimizing-
performance.html#profiling-components-with-the-chrome-performance-tab) tab in
[Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/) or
the [React Profiler](https://reactjs.org/blog/2018/09/10/introducing-the-
react-profiler.html) that is available from React v16.5. By looking at the
results, we can implement shouldComponentUpdate to remove those renderings.

### 2\. Prevent function binding or new object as props inside render

While passing objects and functions as props to child components, it might
seem convenient to just declare these inline or inside the render function
like the following code.

However, this is a bad practice. Since all the props above are declared
inline, every time the render function is called, new objects (for propA and
propB) and a new function (for actionA) is created even though the values they
hold are same. This is also true when using bind inside the render function.
(Using ES6 arrow function automatically binds the function)

Creating new object and function every time on render causes even bigger issue
when we have implemented shouldComponentUpdate to shallowly compare the props
or used PureComponent as discussed in previous point. Since new reference of
the actionA is passed as prop every time, the shallow compare of the props
fail leading to re-rendering of the component.

The above component can be re-written as:

We have now assigned arrow function to actionA and also declared propA and
propB at the time when the component is created.

Note: If you prefer to use bind over the arrow function, you should bind it
early inside the constructor

    
    
    ...  
    constructor(props) {  
      super(props);  
      this.handleActionA = this.actionA.bind(this);  
    }
    
    
    ...

There might be cases when we need to create arrow functions inside the render
method when using list of items. One possible solution is to create a new
component and pass down the id and the function as props as mentioned
[here](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules
/jsx-no-bind.md#protips). You can also opt to use
[memoize](https://lodash.com/docs/4.17.11#memoize) function from lodash
without having to create a new React Component.

### 3\. Index as key

We require keys whenever we are dynamically rendering list of items in loop.
Keys are required by React to identify which items have changed, are added, or
are removed. These keys need to be unique for each element and sometimes we do
end up using index as key for the element.

    
    
    ...  
    {items.map((item, index) =>  
      <Item  
        key={index}  
      />  
    )}  
    ...

Now this code is absolutely fine in most cases until we need to update, sort
or filter the list. In that case, the key for each item also changes and this
can cause the application to behave strange. If you look at the ReactJS
documentation on
[Reconciliation](https://reactjs.org/docs/reconciliation.html), it says:

> Component instances are updated and reused based on their key. If the key is
an index, moving an item changes it. As a result, component state for things
like uncontrolled inputs can get mixed up and updated in unexpected ways.

If you have a list of 1000 elements and you have used index as key, when you
remove the first element, the indexes for all the subsequent items are updated
and since React compares the DOM based on the key, it thinks all the items
have been updated and thus causes performance degradation.

It is hence, advised to use unique, stable and predictable key. You can use
`item.id` in above case given that it has a unique id. If not, you can use
libraries like [shortid](https://www.npmjs.com/package/shortid) to generate an
id for you. You should also prevent using functions like Math.random to
generate keys. You should proceed to use index as key only when you are
certain that the list doesn't update in any sort of form.

### 4\. Production build and code splitting

When working with React application, there are lots of tools that are only
used while development. There might be tools to display different warning and
errors that assist us in debugging issues. These packages come at a cost and
increases the bundle size of our application. When used in the production
environment, it will significantly slow down our application. Thus, while
deploying the React application to the production, you should make sure you
are using the build that is generated specifically to be used in production
environment.

If you are using create-react-app, you can simply generate it using `npm run
build` or `yarn run build` command. If you are using webpack you can also set
flag (`webpack --mode=production`) to instruct it to generate production
build. This way, the bundle size will be significantly lower since
development-only codes are removed. The code will also be uglified and
minified with create-react-app though you might need to setup plugins for
webpack if you aren't using create-react-app. In addition to it, there are
other different ways to reduce size of the production build.

 **Code Splitting  
** Using a single production build is okay for small to medium sized
application but as the application grows, so does the production build size.
Sometimes, due to large production build, the application might take long time
to load. In that case, we can make use of webpack's code splitting feature.

If we have code like this:

    
    
    import someLibrary from './someLibrary';
    
    
    someLibrary.someFunction();

We can split the "someLibrary" file to separate bundle and only load it when
it is required by the application.

    
    
    import('./someLibrary').then(someLibrary => {  
      someLibrary.someFunction();  
    });

This can reduce the initial bundle size and initial page load time
significantly when used correctly. This is however only supported in webpack.
Also, the dynamic import() syntax is still not supported in ECMAScript. You
will need to use [babel-plugin-syntax-dynamic-
import](https://yarnpkg.com/en/package/babel-plugin-syntax-dynamic-import) to
support the syntax.

React also supports [React.lazy](https://reactjs.org/docs/code-
splitting.html#reactlazy) which allows us to render a dynamic import as a
regular component.

    
    
    const OtherComponent = React.lazy(() => import('./OtherComponent'));  
      
    function MyComponent() {  
      return (  
        <div>  
          <OtherComponent />  
        </div>  
      );  
    }

The OtherComponent is bundled into separate bundle and loaded only when
MyComponent is loaded. This makes it very easy to implement code-splitting
based on route. We can easily create separate bundles for routes that have
large components and load it when that route is active.

Suspense in the above example is another component provided by React which
gives us the control to display fallback content while the component is being
loaded. The above example has been taken from React's code-splitting
documentation. You can find more detailed information
[here](https://reactjs.org/docs/code-splitting.html).

### 5\. Clearing timers and event listeners

Timers and event listeners might not be React specific but I've decided to
include it here anyway since we use timers and event listeners frequently in
our applications and can cause memory leaks and performance impacts when not
handled properly.

The component above has setInterval to change the value of countdown every 60
second and also has an event listener for scroll attached when the component
is mounted. Now suppose that this page is rendered by react-router and you
change the route to mount different component. The event and timer will still
continue to exist in the memory.

If there are multiple of such timers and event listeners, it will
significantly impact the page's performance over the time if the page hasn't
been refreshed. Even worse, if the App component is mounted again, new timer
is started and a new event is attached on top of the existing.

This is the reason you always need to clear these intervals and event
listeners before the component unmounts.

setInterval returns an integer which needs to be passed to clearInterval
function to kill the timer. Similarly, we need to pass the same callback
function that was used in addEventListener to remove the event listener.

### Conclusion

So these are few of the ways you can speed up your application. There still
are several other ways like SSR or ServiceWorkers which you can implement but
that really depends on the level of optimization you are looking for. Having
said that, you need to correctly identify where and why your application is
slowing down first before you start to implement the solutions. As I mentioned
earlier, [Chrome DevTools](https://developers.google.com/web/tools/chrome-
devtools/) or the [React Profiler](https://reactjs.org/blog/2018/09/10
/introducing-the-react-profiler.html) are great ways to visualize and
benchmark your application.

Thanks for following through. I hope you liked the article. Do share your
thoughts, opinions or questions in the comments sections. I would love to read
them and get back. If you want to read more of Botsplash team contributions,
check out articles [**here**](https://medium.com/botsplash-engineering).

* * *

For more articles on Live Chat, Automated Bots, SMS Messaging and
Conversational Voice solutions, explore our
[**blog**](https://blogs.botsplash.com/).

Want to learn more about [Botsplash](https://www.botsplash.com) platform?
Write to us [here](https://botsplash.com/support/).

