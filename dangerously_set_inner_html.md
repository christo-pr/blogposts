# Render dangerous content with React
> This post explain how React handle cross-site scripting and how we can 'bypass' it (**NOTE: only if you needed**)

## Cross-site Scripting (XSS Attacks)

Among all the web vunerabilities, one of the most common is [cross-site scripting](https://owasp.org/www-community/attacks/xss/), this type of vunerability allows the attackers to **inject** scripts on the page to get access to any sensitive information the browser and the site are sharing (cookies, tokens, etc...).

This attack occurs when the data entered in your web app is comming from an untrusted source or the data that is sent to the user include dynamic content without been validated first.
Although there are limitless varieties of XSS attacks, Javascript XSS attacks seems to be popular among hackers.

### Types of XSS Attacks

There are 3 types of XSS attacks:

**Stored XSS Attacks** occurs when the injected script is store on the server (i.e. store on a database) so each time the user request a something from the server
the malicious script is sent to the client.

**Reflected XSS Attacks** happens when the malicious script is reflect on the web that's vulnerable, this could be due to a click on an email link that's malformed or any other external source.

**DOM Based XSS Attacks** is a vulnerability that occurs on the DOM (Document Object Model) instead of the HTML.

Let's say you have this code on your app:
```html
<script>
   document.write('<h1>My URL:</h1>: '+ document.baseURI);
</script>
```

Now, imagine someone visits your site using the URL `https://www.nicesite.com/index.html#<script>alert('test')</script>`, the script will be executed because the code above writes whatever comes on url to the document using `document.write`.

We can point one of the main differences between this type of XSS attack and the Stored and Reflected: The servers **can't stop** this attack, since the *hash* (#) part of the url is not being sent to the server on the request.

### Prevent XSS Attacks

For most of the XSS sttacks the solution is simple, just [sanitize](https://medium.com/@abderrahman.hamila/what-sanitize-mean-and-why-sanitize-in-code-data-5c68c9f76164) your input data, even if comes from a ***trusted*** source.
Doing this will ensure that no matter what's the input or output, is always secure.

Javascript offers us plenty ways to interact with the DOM, so we can works with dynamic content in an easy way, but we need to be carefull on how to use it, since it can make vulnerable our websites.

#### Inputs & Outputs

Here's a tiny list of the most common input and outputs that can be dangerous to use.

|         Inputs         |        Outputs      |
|------------------------|---------------------|
|     `document.URL`     |   `document.write`  |
| `document.documentURI` | `element.innerHTML` |
|     `location.href`    |    `element.src`    |


## React and cross-site scripting

Nowadays all the web apps required some dynamism, from having a multiple step form that shows different question depending on your answers to a simple tables that filter out information, this is where Javascript enters to the ecuation.

Back in time, when Javascript vainilla was enough to get everything done (which still is, we just *'syntax-sugar'* it), one of the way you could handle insertion of dynamic content, was using `innertHTML` property.

> Element.innerHTML:
>
> The Element property **innerHTML** gets or sets the HTML or XML markup contained within the element

So, you can set the HTML content from an element using this property, but what happend when the content has an `script` inside?

```js
const content = 'Christofer'
el.innerHTML = content


const newContent = "<script>alert('You've been hacked')</script>";
el.innerHTML = newContent
```

The first 2 lines create a variable that holds a plain string, then using `innerHTML` set the content of an element to be this variable, so far so good, nothing harmless here.

On the next 2 lines of code we do the same, but this time, the string value is html with a `<script>` tag within it, so what would you think will be the output?

Well, if you though that this will result on an alert prompting to the user that he has been hacked, well you are **wrong**.

HTML5 specifications says that scripts inserted using `innerHTML` [shouldn't execute.](https://www.w3.org/TR/2008/WD-html5-20080610/dom.html#innerhtml0)

This way HTML5 helps us to prevent XSS.

----

#### Easy to be safe

React follows the philosophy *"easy to be safe"*, that's why we as developers should be explicit if we want to go for the *unsafe* path, and this is the case for the `dangerouslySetInnerHTML` prop.

This prop allows you to inject dynamic html to an element, all you need to do is pass and object with a single property: `__html`, that holds the html (string format) that you want to render:

```js
function App() {
  const html = `
    <div>
      <h1> Injected html</h1>
    </div>
  `

  return (
    <div  dangerouslyInnerHTML={{ __html: html }}/>
  )
}
```

As you can see, seems a little odd that you have to pass an object when it could be a simple string, but this is done intentionally, to remind you that it's dangerous and you should avoid using it as much as possible.


#### innerHTML vs dangerouslySetInnerHTML

Writing React doesn't mean that you can't use the features that Javascript offer us, you can use `innerHTML` to add the dynamic html to a react component and it will work the same (both will update the node with the html), but it can lead to undesire and performant issues.

React uses a **virtual DOM** to compare what's been update and needs a re-render, using `dangerouslySetInnerHTML` tell React to ignore this html, so it doesn't include it on the comparations agains the virtual DOM.
When using `innerHTML` this "ignore" feature goes away, also the content could be replaced on the next re-render

Since both properties works the same (in fact `dangerouslySetInnerHTML` implements `innerHTML` to set the content) they both share the same vulnerabilities, and they are fixable with the same method: sanitize your input sources.

## Render the danger

React just prevent the js execution of `script` tags within the html we are inserting, so we are still expose to an XSS attack:

```js
import React from 'react'

function MaliciousComponent() {
  const html = `<img src="/path/no/exists/" onerror="alert('hacked')" />`

  return (
    <div dangerouslySetInnerHTML={{ __html: html }}/> // will execute the alert
  )
}

```

And how about the `script` tags?

```js
import React from 'react'

function MaliciousComponent() {
  const html = `
  <div>Nice content</div>
  <script>
    alert('testing')
  </script>`

  return (
    <div dangerouslySetInnerHTML={{ __html: html }}/> // will execute it??
  )
}
```

I mention before, `dangerouslySetInnerHTML` implements `innerHTML`, so it wont execute any script tag at all.

So what happens if you need to execute any `script` tag that comes inside the html? That's against HTML5 specifications, but you could use **DocumentFragments:**

> The specified value is parsed as HTML or XML (based on the document type), resulting in a **DocumentFragment** object representing the new set of DOM nodes for the new elements.

This **DocumentFragment** is a lightweight version of the `document` and you can add childs to it, the main difference is that since is a fragment, is not actually a part of the active/main `document`.

We can create a **DocumentFragment** using the [document.Range](https://developer.mozilla.org/en-US/docs/Web/API/Range) API.

```js
const html = `
  <h1>Fragment</h1>
`
const node = document.createRange().createContextualFragment(html);
```

This code snippet will create a `DocumentFragment` object, parse the value of the `html` variable and store the result on a variable called `node`. All we have to do is render this variable:

```js
element.appenChild(node)
```

If we translate all of this to a React component we end up with something like this:

```js
import React, { useEffect, useRef } from 'react'

// InnerHTML component
function InnerHTML(props) {
  const { html } = props
  const divRef = useRef(null)

  useEffect(() => {
    const parsedHTML = document.createRange().createContextualFragment(html)
    divRef.current.appendChild(parsedHTML)
  }, [])


  return (
    <div ref={divRef}></div>
  )
}

// Usage
function App() {
  const html = `
    <h1>Fragment</h1>
  `

  return (
    <InnerHTML html={html} />
  )
}
```

This way we could pass a string with html content that includes `<script>` tags, and those will be executed
> **NOTE:** Works for both variations `<script> .. content .. </script>` or `<script src="file.js" />`


## dangerously-set-html-content

[dangerously-set-html-content](https://www.npmjs.com/package/dangerously-set-html-content) is a tiny (**297B Gzipped**), **no-dependencies**, library that allows you to render dynamic html and execute any `scripts` tag within it.

1) Add it to your project:

```bash
yarn add dangerously-set-html-content
// or
// npm install dangerously-set-html-content --save
```

2) Start using it:

```js
import React from 'react'
import InnerHTML from 'dangerously-set-html-content'

function App() {
  const html = `
    <div>
      <h1>Fragment</h1>
      <script>
        alert('this will be executed');
      </script>
    </div>
  `

  return (
    <InnerHTML html={html} />
  )
}
```

Of course this doesn't prevent any attack (in fact, does the opposite of that), but sometimes this functionally could be what you are looking for.
> I build this beacuse I actually needed this functionality.

## Conclusions

All the web is full with vulnerabilities that can cause you headache if you are not aware on how to prevent them. Mostly of the common frontend libraries already handle a few of these in a way, so you don't have to worry about it
but still is better to known what are we dealing with as frontend developers.

Additionally of what React offers us, there are several techniques that can help you to prevent an attack, so if you are having a problem of this type just head to the docs and you'll probably find the solution.

It's not recommened to bypass any security feature, since it's there for a reason, but in order to get the job done, sometimes you need to get your hand dirty! (XD) and while 99% of the times all this *magic* behind React works perfect for us, sometimes we can found ourself struggling with it, just remember, is Javascript, so embracing the best of both may help you to find the solution to the problem.

Thanks!


