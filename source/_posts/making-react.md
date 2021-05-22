---
title: Making a tiny React clone - 1
tags:
- React
- Build-my-own-x
---
***Creating something is a great way to understand it.***
Recently, I created a tiny clone of React which I called [***reflex***](https://github.com/saurabh0402/build-my-own-x/tree/main/react). Though it only has the most basic features that React provide, it was a great exercise to learn the internals of React and in the way, there are a lot of things that I learnt. Through this series of blog posts, I will try to jot down everything I learnt along the way.

Credits where due, I followed this beautiful article [here](https://pomb.us/build-your-own-react/) 👏 and these blogs will be a derivative of it only with my learnings sprinkled along the way. At the end of these, you should have a simple React clone of yourself.

Before we hop onto the coding train and crate our own React, let's first get a few terminologies out of the way:
- **Virtual DOM**: This is the concept of representing the DOM or Document Object Model "virtually" in memory (mostly as objects). When we work with React components we actually work with this abstraction and don't usually work directly with the underlying DOM. As we will see later on, we represent the DOM "virtually" using objects.<br><br>
- **Fibers**: These are internal objects which (we will see) is a superset of React components. These objects are an internal representation of the components and are used to add things to the DOM. We will take a very detailed look at what all data these objects contain and their role in rendering later.

Now let's see what happens when we create a simple app in React
```jsx
  import React from 'react';
  import { render } from 'react-dom';

  const App = (
    <div className="hello">Hello World</div>
  );

  render(App, document.querySelector('#root));
```
So, the HTML embedded in JS is what we call JSX. That is not supported by the browser natively and therefore that will firstly be transpiled into normal JS which is usually done through Babel. Once this is transpiled through babel, we get the following result
```js
  import React from 'react';
  import { render } from 'react-dom';

  const App = React.createElement('div', { className: "hello" }, 'Hello World');

  render(<App />, document.querySelector('#root));
```
`React.createElement` is a function that we will soon be implementing but it takes 3 or more arguments - the type of element to create, the props or properties, and the children and it returns nothing but an object - the object representing our `App` component in the virtual DOM. The object for the `App` in this case would be somewhat like this
```js
  const App = {
    type: 'div',
    props: {
      className: 'hello',
      children: [
        'Hello World',
      ],
    },
  };
```
Our example is pretty simple and therefore the final object formed is simple but we can pass in other components as a child as well, also we can have more than 1 children and therefore this object can become pretty complicated. So, that's our element.

The `render(App, document.querySelector('#root));` is what converts the virtual DOM created in the previous step to the actual DOM. (We will see that this not 100% complete 🙈). We will take a look at this later but first, let's see how we can write this `createElement` function. 

---
A tiny note before we write the `createElement` function. We saw that babel transpiled the JSX to `React.createElement` calls but we can actually tell babel to transpile it to something like `Reflex.createElement` by simply adding a comment like this before the JSX
```jsx
  /** @jsx reflex.createElement */
  const App = (
    <div className="hello">Hello World</div>
  );
```
When this is transpiled, it will get converted to 
```js
  const App = Reflex.createElement('div', { className: "hello" }, 'Hello World');
```
---

Okay, no more delay, let's implement the `createElement` function now. Here's the code that should be put inside the `reflex.js` file.
```js
  function createTextElement(text) {
    return {
      type: 'TEXT',
      props: {
        nodeValue: text,
        children: [],
      },
    };
  }

  function createElement(type, props, ...children) {
    return {
      type,
      props: {
        ...props,
        children: children.map((child) =>
          typeof child === 'object' ? child : createTextElement(child)
        ),
      },
    };
  }
```
So, I promised one but created two functions. Believe me, they are important. Let me explain.

Let's start with `createElement`, it's simple enough. The first argument is `type` and the second object is the `props` object as we saw in the react examples. Then, it collects all other arguments inside the `children` array. Next, it returns the object representing this element which has the `type` property assigned as-is. All the props along with the `children` are assigned to the `props` key (this is as per React behaviour, the `children` is also considered a prop). But the `children` object is changed a bit before saving.

To understand this we need to remember that the children can be two things:
- Other elements, in which case they will be objects generated through nested `createElement` calls. We simply append them to the children array.
- Text, in which case we are calling the `createTextElement` function and append the returned value.

So, what we actually do is that for texts, we create a special element using the `createTextElement` function. We do this to have a somewhat consistent behaviour for text and other types of elements.

---

Let's see this in action. To actually test this, we need to do some setup. Create a folder called `example`.
Inside the `example` folder create a new file called `babel.config.js` with the following content
```json
{
  "presets": [
    ["@babel/env"],
    ["@babel/react"]
  ]
}
```

Next, create a new file `index.jsx` with following content
```jsx
  const reflex = require('../reflex');

  /** @jsx reflex.createElement */
  const App = (
    <div>
      <h1> Hello World </h1>
    </div>
  );

  console.log(JSON.stringify(App, null, 2));
```

Install required dependencies using the following command
```
yarn install -D @babel/core @babel/cli @babel/preset-react @babel/preset-env
```

And now, finally, run the following command
```
npx babel index.jsx --our-file build.js
node build.js
```

You should see an output similar to this
```js
{
  "type": "div",
  "props": {
    "children": [
      {
        "type": "h1",
        "props": {
          "children": [
            {
              "type": "TEXT",
              "props": {
                "nodeValue": " Hello World ",
                "children": []
              }
            }
          ]
        }
      }
    ]
  }
}
```
---
So, one step down. We have created the `createElement` function but that's just a beginning. Next, we will be implementing the `render` function and see how these two fit together.
Meet you on that side, until then tada 👋 👋