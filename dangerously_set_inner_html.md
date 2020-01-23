# Render dangerously content with React
> This blogpost show how React handle one of the most common web vulnerability

## Crosss-site Scripting (XSS Attacks)

Among all the web vunerabilities, one of the most common is [cross-site scripting](https://owasp.org/www-community/attacks/xss/), this type of vunerability allows the attackers to **inject** scripts on the page to get access to any sensitive information the browser and the site are sharing (cookies, tokens, etc...).

This attack occurs when the data entered is comming from an untrusted source or the data that is sent to the user include dynamic content without been validated first.
Although there are limitless varieties of XSS attacks, Javascript XSS attacks seems to be popular among hackers.

----

### Types of XSS Attacks

There are 3 types of XSS attacks:

**Stored XSS Attacks** occurs when the injected script is store on the server, this could be from a database, so each time the user request a resource from the server, this malicious scripts gets attached to the response.

**Reflected XSS Attacks** happens when the malicious script is reflect on the web with the vulnerability, this could be due to a click on an email or any external source.

**DOM Based XSS Attacks** is a vulnerability that occurs on the DOM (Document Object Model) instead of the HTML.
Let's say you have a this snippet of code on your web app:

```html
<script>
   document.write('<h1>My URL:</h1>: '+ document.baseURI);
</script>
```

Now, imagine someone visits your site using the URL `https://www.nicesite.com/index.html#<script>alert('test')</script>`, they will get the alert because the script writes the whatever comes on url to the document using `document.write`.
We can point one of the main differences between this type of XSS attack and the Stored and Reflected: The servers **can't stop** this attack, since the *hash* part of the url is not being sent to the server on the request.

### Mitigate XSS Attacks

For most of the XSS Attacks the solution is simple, just [sanitize](https://medium.com/@abderrahman.hamila/what-sanitize-mean-and-why-sanitize-in-code-data-5c68c9f76164) your input data, even if comes from a ***trusted*** source and use the correct inputs and outputs

Javascript offers us plenty ways to interact with the DOM, so we can works with dynamic content in an easy way, but we need to be carefull on how to use it, since it can make vulnerable our websites.

#### Inputs & Outputs

Here's a tiny list of the most common input and outputs

|         Inputs         |        Outputs      |
|------------------------|---------------------|
|     `document.URL`     |   `document.write`  |
| `document.documentURI` | `element.innerHTML` |
|     `location.href`    |    `element.src`    |

---

## React and cross-site scripting

---

Nowadays all the web apps required some dynamism, from having a multiple step form that shows different question depending on your answers to a simple tables that filter out information based on the logged in user, all of this require depend a lot from Javascript.

Back in the time, when Javascript vainilla was enough to get everything done (which still is, we just *'syntax-sugar'* it), one of the way you could handle insert dynamic content was using `innertHTML` property.
According to the docs:

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

The first 2 lines create a variable that holds a plain string, then using `innerHTML` set the content of an element to be this variable, so far so good, nothing is harmless here.

On the next 2 lines of code, we do the same but this time, the string value is a and html `<script>` tag with an alert on it, what would you think is the output of this??

Well if you though that this will result on an alert prompting to the user that he has been hacked, well you are **wrong**.

HTML5 specifications says that scripts inserted using `innerHTML` [shouldn't execute.](https://www.w3.org/TR/2008/WD-html5-20080610/dom.html#innerhtml0)

----

##### Easy to be safe

React follows the philosophy *easy to be safe*, that's why we as developers should be explicit if we want to go for the *unsafe* path, and this is the case for the `dangerouslySetInnerHTML` prop.

This prop allows you to inject dynamic content to an element, all you need to do is pass and object with a single property: `__html`, that holds the html that you want to render:

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

As you can see, seems a little odd that you have to pass an object, when it could be a simple string but this is done intentionally to remind you that it's dangerous and well you should avoid using it as much as possible.


#### innerHTML vs dangerouslySetInnerHTML

Writing React doesn't mean that you can't use the features that Javascript offer us, that beign said, you can use `innerHTML` to add the dynamic html to a react component and it will work the same (both will update the node with the html), but it can lead to undesire and performan issues.

React uses a **virtual DOM** to compare what's been update and needs a re-render, using `dangerouslySetInnerHTML` tell React to ignore this html, so it doesn't include it on the comparations agains the virtual DOM.

Another issue when using `innerHTML` is that due to this virtual DOM comparation, the dynamic html could be replace on the next re-render.

Since both properties works the same (in fact `dangerouslySetInnerHTML` implements `innerHTML` to set the content) they both share the same vulnerabilities, and they are fixable with the same method: Sanitize your inpur sources.

## Render the danger

Now what happens when you want to use `dangerouslySetInnerHTML` but also need to execute any `script` tag that comes inside the html? That's agains HTML5 specifications, but if we dig a little bit more on what `innerHTML` do to inject the html we can found something interesting:

> The specified value is parsed as HTML or XML (based on the document type), resulting in a **DocumentFragment** object representing the new set of DOM nodes for the new elements.

So, this **DocumentFragment** creates a lightweigh version of the `document` that can hold nodes, the main difference,is that since is a fragment, it doesn't actually a part of the active `document`.

Good for us, we can create a **DocumentFragment** using the [document.Range]() object.

```js
const html = `
  <h1>Fragment</h1>
`
const node = document.createRange().createContextualFragment(html);
```

This snippet of code will create a `DocumentFragment` object and parse the value of the `html` variable, and store the result on a variable called `node`. All we have to do is to render this variable:

```js
element.appenChild(node)
```

If we translate all of this to a React component we end up with something like:

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

[dangerously-set-html-content](https://www.npmjs.com/package/dangerously-set-html-content) ss a tiny (297B Gzipped), no-dependencies, library that allows you to render dynamic html and execute any `scripts` tag within it.

Just add it to your project:

```bash
yarn add dangerously-set-html-content
// or
// npm install dangerously-set-html-content --save
```

and you can start using it;

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

Of course this doesn't prevent any attack (in fact, does the opposite of that), but sometimes you can find the needed of rendering and execute (if it has `script` tags) dynamic html content

## Conclusions

All the web is full with vulnerabilities that can cause you headache if you are not aware on how to prevent them. Mostly of the common frontend libraries already handle a few of these in a way, so you don't have to worry about it
but still is better to known what are we dealing with as frontend developers.

Additionally of what React offers us, there are several techniques that can help you to prevent an attack, so if you are having a problem of this type just head to the docs and you'll probably find the solution.

While 99% of the times all this *magic* behind React works perfect for us, sometimes we can found ourself struggling with it, but at the end is just Javascript so embracing both will help us to find the solution to our problem.

Thanks!


