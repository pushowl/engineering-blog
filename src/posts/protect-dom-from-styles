---
title: Protecting your DOM from website styles!
date: '2021-01-11'
tags:
  - blog
  - CSS
author:
  name: Kushagra Gour
  link: https://twitter.com/chinchang457
---

Many times we need to build elements on a 3rd party website. The issue is, that the CSS on that 3rd party website can affect your DOM. What options do we have in this case to protect out DOM from the 3rd party CSS?

## A trivial start - inline styles

Inline styles take precedence over styles in CSS files. So if we write all our styles as inline styles, we should be good. Not really! There are 2 issues here:

1. There may still be a property being set in the 3rd party CSS on a generic selector like `div` which we don't have in our inline styles. That would affect us.
2. Even our inline styles could be getting overridden because the 3rd party CSS has those properties set through `!important`

Fix to problem #2 could be to write all our inline styles with `!important` too. 

And for problem #1, we could (just a possibility) put each possible property in our inline styles. Clearly this isn't a very scalable and reliable solution as it will bloat our CSS and also, we can't be sure that we have all properties listed as new properties keep coming.

Note: This doesn't need to be really inline, it could be in-page as well i.e. `<style>` tag injected in the page because 3rd party styles cannot be inline on our DOM, it can be at max be in-page.

## CSS file with stronger selectors

As mentioned above, we don't really need to inline our styles as we don't need to override any inline styles. We could also go a better route of having our CSS in CSS files. But then what about website CSS overriding our rules?

We can do 2 things:

1. We can use selectors with very high specificity. Eg. `#form .wrap h1`. Use of ID and multiple classes gives our selector high specificity and reduces chance of someone overriding us.
2. Plus we can always put `!important` in our property declarations.

This approach gives us better developer experience, file caching but still suffers from same issues as in previous approach.

## The iframe abstraction

One typical way of solving this is using an `iframe`. We could put all our DOM inside an iframe and that ways the 3rd party CSS cannot affect our DOM. That does solve the style overriding but brings in additional problems:

1. **We loose on Accessibility**. Since now our DOM elements inside the iframe won't be acessible through the keyboard with Tab focus.
2. **Communication becomes difficult**. This actually depends - if we choose to host all the JavaScript inside the iframe as well, then we won't be able to communicate with the parent window normally. A different approach here is to have all the JavaScript in the parent frame itself and just create the DOM in an iframe with an empty URL.
3. **DOM Event** - This is somewhat related to communication. We can't listen for the DOM events firing inside the iframe by being outside the DOM. That means if we want to know about these events outside the iframe, we'll have to use something like `postMessage` for communication.

## Shadow DOM

[Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM) came as part of the Web Components API. It offers an encapsulation for being able to keep the markup structure, style, and behavior hidden and separate from other code on the page so that different parts do not clash, and the code can be kept nice and clean. Just what we need here!

Moreover, its well supported in all modern browsers now. So we can easily use it without worrying about compatibility.

That is it for this post, until next time!
