# JavaScript Handlebars Templates

## Objectives

1. Explain how to use Handlebars templates
2. Describe Handlebars' built-in helpers (`if` and `each`)
3. Practice writing a template using Handlebars
4. Write a custom Handlebars helper

## Introduction

Templates are a powerful way to dynamically create HTML content without
having to hand-build and concatenate strings of HTML and data values within your JavaScript code.

A good template engine will allow you to create HTML templates and
insert placeholders for dynamic data in the least obtrusive way
possible, separating the definition of the template from the consumption
of it so that our code stays readable, maintainable, and well-organized.

Today we're going to work with [Handlebars](http://handlebarsjs.com),
which is a powerful template engine. We'll see how to render simple
templates and also learn how Handlebars makes tasks like working with
collections of data easier.

## Basic Handlebars

Let's jump right in and create a Handlebars template and render it to
the page. Similar to some other template engines, we can define a
Handlebars template inside of a `<script>` tag with a combination of
regular HTML and Handlebars-specific delimiters.

In our `index.html` let's define a basic template to render a github
issue.

```html
<!-- <body>... -->
<main id="main">
  <a href="#" onclick="loadIssue();">Load Github Issue</a>
</main>
<script id="issue-template" type="text/x-handlebars-template">
  <article>
    <header><h3>Issue #{{number}} ({{state}})</h3></header>
    <p>{{body}}</p>
    <footer><a href="{{url}}">created {{created_at}}</a></footer>
  </article>
</script>
```

We have a `script` tag with an `id` and a `type` of
`text/x-handlebars-template`. We need that type to be set for Handlebars
to know what to do with this template, and we need to give it an `id` so
that we can easily select it and get its `innerHTML` when we render it.

Inside the `script` tag, we have some pretty basic HTML for the most
part, and then some variables  marked by double curly brace delimiters,
like `{{state}}`. What we're saying there is that, when we render this
template, we're going to have a variable called `state` that we want to
go in that spot.

**//Flat-fact:** Handlebars uses the curly brace as a delimiter because if you turn it
sideways it looks like a mustache, and Handlebars.js is an extension of
the [Mustache](http://mustache.github.io/) template language. One need
not grow a mustache to be an effective user of Handlebars, but if you're
also hoping to be a legendary relief pitcher, it couldn't hurt.

![rollie](http://i.giphy.com/26tOW7KBU86Vkc4rC.gif)

Finally, we included a simple link to fire a `loadIssue` function that
we'll use to render our template, so let's get into our `index.js` and
create that function.

```js
//index.js
function loadIssue() {
  var issue = {
    state: "closed",
    number: 5,
    created_at: "2016-03-31 16:23:13 UTC",
    body: "Instructions say GET /team and POST /newteam. Rspec wants GET/newteam and POST/team."
  }

  var template = Handlebars.compile(document.getElementById("issue-template").innerHTML);
  var result = template(issue);
  document.getElementsByTagName("main")[0].innerHTML += result;
}
```

We started by creating a simple `issue` object to hold our Github issue
data. This will be used as the `context`, or argument of our Handlebars
template. Note that the property names match the names of the variables
in our template. This is crucial. If your context object properties
don't match the template variables, you'll get unexpected results.

Next, we make a call to `Handlebars.compile` with the `innerHTML` of our
template. This will "compile" the markup and delimiters as part of a
function that we can call with our context to output a rendered HTML
string, which is what's happening in `var result = template(issue)`. If
we had another `issue` object with different data, we could call
`template()` again and get the same template rendered with the new data.

Finally, we're taking our rendered template and putting it back into our
page so we can see it. If you load up your `index.html` page, you should
be able to see this in action!

Already, Handlebars is giving us a powerful way to render templates with
our data, but it offers a few more things that make it stand out even
more.

## Handlebars Helpers

Handlebars offers several built-in functions, or *helpers*, that provide
easy ways to work with code inside our templates. We'll explore two of
these helpers: `if` and `each`.

### each Helper

In the example above, we rendered a single issue to the page, but what
if we have a whole collection of issues? Take a look inside `issues.js`
and you'll see an array of about a thousand Github issues just waiting
to be displayed on our issue-tracking page.

Now we could take this data and render it using a `for` loop. We already
know that our `template` function can be reused with a different context
object, so, if we modify `index.js` a little, we can render all our
issues.

```js
function loadIssue() {
  var template = Handlebars.compile(document.getElementById("issue-template").innerHTML);
  for(var i=0;i<issues.length;i++) {
    var result = template(issues[i]);
    document.getElementsByTagName("main")[0].innerHTML += result;
  }
}
```

Here, we're still compiling `template` the one time, then using it
inside our loop to output to the DOM. If we reload index.html, it works
(albeit a little slowly, perhaps).

There's a couple of problems with this approach. The first is that it
was very slow. We're reallocating this `result` variable every time,
then querying the DOM every time and adding to it every time through the
loop.

The second problem is that we're doing the heavy-lifting of iterating
over the collection ourselves, instead of letting our template do it for
us. With just a few modifications, we can take advantage of the built-in
`each` helper and let Handlebars worry about building out the
collection.

First, let's alter the `loadIssue` function.

```js
function loadIssues() {
  var template = Handlebars.compile(document.getElementById("issue-template").innerHTML);
  var result = template(issues);
  document.getElementsByTagName("main")[0].innerHTML += result;
}
```

Here we're just passing the whole `issues` array to `template` and
rendering it all at once. Also note that we changed the function name to
`loadIssues`. Since we're dealing with a collection of issues, it's a
good practice to make the name plural. Don't forget to change it in
`index.html`.

Now let's alter our template to handle the collection.

```html
<main id="main">
  <a href="#" onclick="loadIssues();">Load Github Issue</a>
</main>
<script id="issue-template" type="text/x-handlebars-template">
  {{#each this}}
  <article>
    <header><h3>Issue #{{number}} ({{state}})</h3></header>
    <p>{{body}}</p>
    <footer><a href="{{url}}">created {{created_at}}</a></footer>
  </article>
  {{/each}}
</script>
```

Did you change `loadIssues`?

Note the two new things in our template: `{{#each this}}` and
`{{/each}}`. The `#` and `/` are needed because `each` is a
*block-level* helper, so we have to start (`#`) and end (`/`) the block
similar to open/close braces for a JavaScript block or open/close tags
for HTML.

The `each` helper needs something to iterate on. In this case, we'll use
`this`, because `this` is set to whatever the context object is when we
call our template function. Here, it's an array of objects, so we're
saying "`this` is an array, apply template to `each` item".

Nothing else to do here, since each item of the array still has the
properties we used before. Let's reload `index.html` and see what
happens.

Much faster, right? Taking advantage of the built-in `each` helper not
only sped things up, but helped keep our code cleaner and easier to
read.

### if Helper

Handlebars gives us another block helper for making decisions in our
template - the `if` helper.

Looking at our `issues.js`, it seems like some of our issues have
comments, and some do not. Let's conditionally display the number of
comments if there are any. We'll use the `if` block helper.

```html
<script id="issue-template" type="text/x-handlebars-template">
  {{#each this}}
  <article>
    <header><h3>Issue #{{number}} ({{state}})</h3></header>
    <p>{{body}}</p>
    {{#if comments_count}}
    <p>{{comments_count}} Comments</p>
    {{else}}
    <p>No Comments Yet</p>
    {{/if}}
    <footer><a href="{{url}}">created {{created_at}}</a></footer>
  </article>
  {{/each}}
</script>
```

We added a block helper for `{{#if comments_count}}`. You'll notice this
isn't a full expression like you might use in a regular `if` statement.
You can only give `if` a value from the context object, and if that
value evaluates *falsy* (`false`, `undefined`, `null`, `0`, `""`, or
`[]`) then Handlebars won't render the block. In this case, since
`comments_count` is an integer, we can rely on a zero-comments object
not rendering this block.

The next thing to notice is that we provided an `{{else}}` to default to
if there are no comments. We don't use the `#` for `else`, it only goes
at the very beginning of the block.

For simple manipulation of the template based on truthy/falsy values,
the `if` helper is your friend!

## Custom Helpers

We've seen some built-in Handlebars helpers that give us easy ways to
iterate collections and make simple display decisions, but what if we
want to encapsulate some more complex display logic into a function to
use in our template?

Handlebars allows us to create and register our own custom helpers for
exactly this purpose.

Let's say we want to apply a different style to the issue body if the
issue is closed versus open.

We could probably use that built-in `if` helper to draw out two
different template styles, the more we litter our templates with
decision-making logic, the less decoupled our presentation logic is from our
data.

Also they just get ugly with a bunch of `if` statements all over the
place.

Instead, we want to encapsulate that logic into a function that will
allow us to use a helper in our template that will do the work for us.

We can use `Handlebars.registerHelper` to register our own helper for
use in our template. Let's add one to our `index.js` above our
`loadIssues` function.

```js
Handlebars.registerHelper('comment_body', function() {
  if(this.state === "closed") {
    return new Handlebars.SafeString(this.body)
  } else {
    return new Handlebars.SafeString("<strong>" + this.body + "</strong>")
  }
})

function loadIssues() {
  //...
}
```

We are telling Handlebars to register a helper called `comment_body` for
our use. Inside, we have access to the same `this` as the template that
calls the helper, in this case, it will be our issue context object.

**Top-tip:** You can also register a helper and pass a value to the
function, if you aren't 100% sure you can safely use the context object.

We use `Handlebars.SafeString` to return a string that contains HTML so
that HTML won't get escaped.

Then it's a simple matter of just wrapping `this.body` inside `strong`
tags for unclosed issues, and we've created our first Handlebars helper!

Now to use it in our template:

```html
<script id="issue-template" type="text/x-handlebars-template">
  {{#each this}}
  <article>
    <header><h3>Issue #{{number}} ({{state}})</h3></header>
    <p>{{comment_body}}</p>
    {{#if comments_count}}
    <p>{{comments_count}} Comments</p>
    {{else}}
    <p>No Comments Yet</p>
    {{/if}}
    <footer><a href="{{url}}">created {{created_at}}</a></footer>
  </article>
  {{/each}}
</script>
```

Here we've simply replaced `{{body}}` with our helper, `{{comment_body}}`. Now if we reload our `index.html` and click our link, we should see closed issues with regular font and open issues with bold!

## Summary

We've learned how to use Handlebars templates to render object, and
collections of objects with the built-in `each` helper. We've made decisions on how to display things
using the built-in `if` helper, and implemented and registered our own
custom helper to move logic out of the template.

## Resources

- [Handlebars](http://handlebarsjs.com/)

<p class='util--hide'>View <a href='https://learn.co/lessons/javascript-handlebars-templates'>Handlebars Templates</a> on Learn.co and start learning to code for free.</p>
