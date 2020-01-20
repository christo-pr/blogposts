<div align="center">
<h1> React - Bootstraping your app</h1>
</div>

> Style guide and best practices to write React based on my experience :D


This repo shows the programming style I use to write React components, all based on my experience working with this library.

Feel free to use it or change it at your own will.

Enjoy!
-----

# Javascript linter

Even though this post is about React, we can't write clean React code, without writing clean Javascript code first

Every time I write Javascript I like to have a linter to check some basic style rules to make code.
I used to use [ESLint](https://eslint.org), which is pretty common among js developers, but then I found
[Standard](https://standardjs.com), my favorite js linter.

It has some special (some would say ugly) rules (like no semi-colon, or the lack of ability to change any rule),
but I found my code cleaner and looks better (at least for me) when using it.

Standard has some features that make it easy to use:

- **You don't need a configuration file to start using the linter.**

    There's a lot of discussions about programming style in js (tabs vs spaces, semi-colons) that's why I like
  standard, because it picks some 'standard' rules and that's all you have, no more, no less therefore no more discussions.

------

- **Automatically fix your problems (just use the `--fix` flag).**

    I always create 2 npm scripts: `npm run lint` and `npm run lint:fix`.
  The first script is what I use more often, that shows all the errors with information about the line and file where they occurr
  The second script is just to automatically fix the common errors, but I still try to manually fix as much as possible.

------

- **Git pre-commmit hook.**

    Sometimes when I'm more strict about the programming style, I create a
  [pre-commit](https://standardjs.com/#is-there-a-git-pre-commit-hook) hook for the project, that can save
  some review time.


# Initializing a react project

The `npm registry` is one of the biggest databases of public and private libraries for javascript. It offers
a command line client to use all the features like, download, upload and do some other things, that allow you to
intereact with serveral javascript packages.

There's one particular package that I use a lot when creating a react app: [npx](https://github.com/zkat/npx#readme).
This lib allows you to **execute** the package binaries files, it executes the `<command>` you passed in, following an specific order:

  - Local `node_modules/.bin` folder
  - A central cache
  - Install it and execute

#### Why `npx` instead of `npm`?

Honestly, I just don't like to struggle with all the versions and deprecations stuff, with `npx` I ensure that the library that I'm using is on the latest stable version.

```bash
npx <command>
```

You can use `npm` to download React and start your project from scratch (configure webpack to do all the fancy stuff
you are used to have 'out-of-the-box') or you can use [create-react-app](https://github.com/facebook/create-react-app).
This library allow you to bootstrap a React project. It all the configurations needed for a rapid development with React (like hot reload, ES6 support, etc...)

One of the good things about CRA, is the abilty to have all the latest patches and features with a single bump
version of your `react-scripts` dependency (which is what CRA uses), so you don't have to worry for this anymore.
CRA also allows you to have your custom setup by **ejecting** your application, this action will give you the
full control of the webpack configuration, so you can twist it and do whatever you want with it.

 #### `npx` with `create-react-app`

So now you know all the benefits of using `npx` and `create-react-app`, you can start figuring out how we can mix up these 2 libraries to simplify the creation of a React application.

Each time I start a new React project I just run:

```bash
npx create-react-app awesomeApp
```

That command will download (the latest-stable version) CRA and executes it, that's why we need to pass the name of the project we want to use (*awesomeApp*).

# Organize app structure

CRA offers a very basic folder structure for your app:

```
 awesomeApp/
  |__public/
  |__src/
    |__App.css
    |__App.js
    |__App.test.js
    |__index.css
    |__index.js
    |__logo.svg
    |__serviceWorker.js
    |__setupTests.js
  |__.gitignore
  |__package.json
  |__README.md
```
> Some apps that I have in production are so simple that this folder structure is enough to work.

When I know that a project will be a little more complicated than that, I change the folder structure so that it's easy for me and any other developer jump into the project.
I split my components in two types:

 - **Components**
 - **Cointainers**

Following this idea, the folder structure that I use, looks like pretty much like this:

```
 awesomeApp/
  |__public/
  |__src/
    |__components/
      |__ui/
    |__containers/
    |__utils/
    |__App.css
    |__App.js
    |__App.test.js
    |__index.css
    |__index.js
    |__logo.svg
    |__serviceWorker.js
    |__setupTests.js
  |__.gitignore
  |__package.json
  |__README.md
```

##### Components

This is where I put all my UI components, that means, components that doesn't do to much of a logic, they are just there to present some information to the user and depend a little bit from the props
that we passed in.

The `ui/` folder holds all the components that are related to the User Interface (i.e. custom component for commons elements like <CustomInput /> or <CustomImg />)

##### Containers

This is where I put the *smart* components. A smart component is the one that controls the state of a particular part of the app. I use this type of components to wrap the most of the base markdown of the pages.

Also I create a new folder called `utils/`, which I use for all the utility functions that I'd use across the app.
> **Note:** When I work with React Native, I just rename the `containers/` folder to `pages/`, seems better name for me.

# Organize Code

The `index.js` file contains all the code that register the service working and also render your app. So this file is basically your entry point, I suggest not to touch nothing on this file
unless you really have to.

Then we have the `App.js` file, which is the React component that's been rendered on the `index.js` file. I use this file as my main React file, and I try to keep it as simple as possible.
Most of my `App.js` file look like this:


```jsx
import React from 'react'

import MainContainer from './containers/MainContainer'

function App() {
  return <MainContainer />
}

export default App
```

We can point up some things here:

- 1) It's a functional component instead of a class component.
- 2) It does nothing but render a main container component
- 3) Export a defult function which is the actual component


##### 1) It's a functional component instead of a class component:
  I used to use class components all the time, so that I could have an state and control everything with the lifecycles of React but
  since hooks came out, all my components started to shrink a lot, and I liked it, so I haven't need to use a class component again since then.


##### 2) It does nothing but render a main container component:
  I always try to keep this component clean, unless I need some initial data that's coming from outside (i.e. API calls). So this will only return the main container, which
  will have all the bussines logic.
  I often use this function to wrap my app with a High Order Component (HOC), like react router or any css theme, so that's available for any chidlren component.


##### 3) Export a defult function which is the actual component
  Each time I jump to an existing project and try to figure out all the imports a single file is doing, is really annoying to be searching if there's any export in a particular
  line, or if they default export a function that's been declare in line 128, that's why I prefer to have all my exports at the end of the file, so each time I wanted to see what's
  been exported, I just go the end of the file.


### Props and State

I used to use class components for my **containers/pages** and functional components for all other component, this way I could separate the concenrs for each type of component.

Now since hooks are live, I found myself writting cleaner components using functional components and hooks.

##### Class components

A simple class component of my own looks like this:

```js
import React from 'react'

class HomeContainer extends React.Component {

  state = {}

  componentDidMount() {
    // Initialization of component's state
  }

  customMethod = () => {
    // 'this' is safe
  }

  render() {
    const { prop1, prop2 } = this.props

    // Render anything
  }
}

export default HomeContainer
```

First, I import React, some people use destructuring to import `Component` also, I use the React variable since `Component` is available as a property of the default export of react.

I also [don't use constructor](https://hackernoon.com/the-constructor-is-dead-long-live-the-constructor-c10871bea599), I prefer to use [class properties](https://github.com/tc39/proposal-class-fields)
to define the state, and use the lifecycles of react to fetch initial data or to update state on renders.

I've always though that the use of `this` in javascript is really hardcore, but I liked though, seems to me like if you had all the javascript wisdom just beacuse `.bind` is in your code.
I change all of that when working with React, (even tough I still think that using `this` is cool if that solves your problem) instead of the regular method declaration of the classes
I use an arrow function assignment, so `this` keyword works as expected and looks cleaner.

The first line of the `render()` method is always the desctructuring of all the props of the component, so next time I come across this component,
I know easily which props I'm using without having to dig into all the jsx code (which supposed to be clean).

And last but not least, I export the component at the end of the file.

##### Functional components

For functional components I follow kinda the same rules:

```js
import React, { useEffect } from 'react'

function Test(props) {
  const { prop1, prop2 } = props

  useEffect(() => {
    // Initialization of component's 'state'

  }, []) // '[]' == no dependecies == only time execution

  return (
    // All the render
  )
}

export default Test
```

So I still use the same destructuring-first technique for the props.

When I need to do some state initialization of my functional components (i.e using `useState` hook) I use the `useEffect` hook, which is the replacement for the lifecycles on the class components.

Finally I export my component at the end of file.

##### Handle JSX

JSX is the syntax extension for javascript, it looks like html tags, and allows you to manipulate the UI of your components.

There are some rules when using JSX though, one of the most known rule is the use of `className` instead of `class` for the html tag property, this is because the special keyword `class` in Javascript
represents a class declaration and it's reserved.

Another speacial rule for jsx is that it doesn't allow multiple elements to be rendered, something like this:

```js
import React from 'react'

function CheckBox(props) {

  return (
    <label>
      Checkbox
    </label>
    <input type="checkbox" value="1" />
  )
}
```

This component is not jsx-valid, since you can't render multiple elements from a React component, instead you have to wrap all the content within a parent element. Most people use a `div`

```js
import React from 'react'

function CheckBox(props) {

  return (
    <div>
      <label>
        Checkbox
      </label>
      <input type="checkbox" value="1" />
    </div>
  )
}
```

This works perfectly mostly of the time, but there are some special cases where this could be an issue (i.e. inside of a table row, you can't have a `div` element as a child), so for those cases, the React team
build `Fragment`.

With `Fragment` you can safely return multiple elements without worrying about the semantic of the html

```js
import React from 'react'

function CheckBox(props) {

  return (
    <React.Fragment> // <>
      <label>
        Checkbox
      </label>
      <input type="checkbox" value="1" />
    </React.Fragment> // </>
  )
}
```

There's a shortcut for `Fragment` that you can use instead: `<> ... </>` but you have to choose when to use it, since this shortcut **doesn't accept any prop**
while the `Fragment` component let you use the `key` prop, which is helpfull when creating elements within a loop.

# Organize your dependencies

When I started to work with javascript, I love how the community helps to solve any kind of issue. Almost anything you would need when creating an app is likely to have it's own library/sdk than can help you with that.
At first glance, that's good, but it can lead you to a ***lazyness*** development, where you are used to find a library for nearly any feature you'd need, so when you don't find the library, you start thinking that
it might be hard to do (at least that was what I though :sad:).

In order to remove that bad habit of **depend** a lot of my **dependencies** (that's what the name stands for ??), I started to take a look to the code that I included on my projects, and that's how I realize that some of it
it's really simple that it might wont worth to be included, and can be just a new file in the `utils/` folder.

I also try to think twice before install a dependencie that's kinda small (I used to include [momentjs](https://momentjs.com) on each project that I needed to present a simple formatted date)
so my `node_modules/` folder doesn't grow up too much.

##### Versioning on your dependencies

**Versionin** is a huge problem on the Javascript environment (I supposed that all languages have this problem). You install the version 9.x of a dependecy, and it works perfectly on your React 16.3, but then after a few
months (or event weeks) in production a new version of that libary came out, and you just deploy normally to production, then `npm install` do it's job. Next, you have a white screen (no matter how many times you reload the page) presented to your users, ugh!!

With npm, you can use [Range versions](https://github.com/npm/node-semver#ranges) to control the version of your dependecies, by default it's configure to use the **caret range**, that means patch and minor updates are allowed

```
^1.0.0 => 1.x.x
~1.0.0 => 1.x.0
```

So when you install a package, your package json looks pretty much like this:

```js
"dependecies": {
  "react": "^16.3.1",
  "momentjs": "^4.3.1",
  // ...
}
```

Even though that `minor` and `patch` updates should not break your code, not everyone follow that rule, and sometimes you can struggle with that for a long time without notice that it's because of the library version.
That's why I **lock** the version of my dependecies (I just remove the caret or the tilde), so whenever I do a npm install again, the same version of the dependecy will be downloaded.

Of course, doing this will require that you keep up-to-date with the dependecies that are likely to be updated often.


# Wire up

One of the reasons why Javascript is well adopted, is the flexibility on how to write code, it doesn't have an explicit convention on how to do it, but that can lead to technical debt when doing it, that's why I stick to some rules
when working with Javascript, and also why you should do it too, the future yourself (or any developer you work with), will thank you for that.

I presented you a small style guide that I follow when working with React, you can use it, or twisted at your convinence, whatever makes you feel happy when programming!

Thanks for reading and happy coding!
