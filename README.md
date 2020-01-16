<div align="center">
<h1> React - The good parts </h1>
</div>

> Style guide and best practices to write React based on my experience :D


This repo shows the programming style I use to write React components, all based on my experience working with this library.
Feel free to use it or change it at your own will.

Enjoy! 
-----

# Javascript linter

Every time I write Javascript I like to have a linter to check some basic style rules to make code.
I used to use [ESLint](https://eslint.org), which is pretty common among js developers, but then I found
[Standard](https://standardjs.com) and became my main linter.

It has some special (some would say ugly) rules (like no semi-colon, or the lack of ability to change any rule), 
but I found my code cleaner and looks better (at least for me).

Standard has some features that I like and make it easy to use:

- **You don't need a configuration file to start using the linter.**
  
    There's a lot of discussions about programming style in js (tabs vs spaces, semi-colons) that's why I like 
  standard, because it picks some 'standard' rules and that all you have, no more, no less
  
------

- **Automatically fix your problems (just use the `--fix` flag).**
  
    I like to have 2 npm scripts: `npm run lint` and `npm run lint:fix`, but I always try to fix the problems
  manually.
  
------

- **Git pre-commmit hook.**
  
    Sometimes I really like to be strict about the programming style, so I create a 
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

Honestly, I just don't like to struggle with all the versions and deprecations stuff, with `npx` I ensure that the library that I'm using is on the latest stable version,

```js
npx <command>
```

You can use `npm` to download React and start your project from scratch (configure webpack to do all the fancy stuff
you are used to have 'out-of-the-box') or you can use [create-react-app](https://github.com/facebook/create-react-app).
I like to keep it simple (when is possible), so I use it to bootstrap mostly of my React projects.

One of the things I like about CRA, is the abilty to have all the latest patches and features with a single bump 
version of your `react-scripts` dependency (which is what CRA uses), so you don't have to worry for this anymore.
CRA also allows you to have your custom setup by **ejecting** your application, this action will give you the 
full control of the webpack configuration, so you can twist it and do whatever you want with it.

 #### `npx` with `create-react-app`
 
So now you know all the benefits of using `npx` and `create-react-app`, you can start figuring out how we can mix up these 2 libraries to simplify how we create a React application.

Each time I start a new React project I just run:

```js
npx create-react-app my-react-app
```

That command will download (the latest version) CRA and executes it, that's why we already are passing the name of the project we want to use.


