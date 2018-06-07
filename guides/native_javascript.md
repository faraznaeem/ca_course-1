In this section we'll demonstrate how to accomplish some of the most common DOM manipulation tasks with plain JavaScript, namely:

- querying and modifying the DOM,
- modifying classes and attributes,
- listening to events, and
- animation.

### DOM Manipulation: Querying the DOM

*In the usage examples, you may encounter methods we haven’t introduced explicitly. In this case just refer to the excellent [Mozilla Developer Network](https://developer.mozilla.org/en-US/) for details.*

The DOM can be queried using the `.querySelector()` method, which takes an arbitrary CSS selector as an argument. 

Given we have this html markup on our page: 

```html
<div id="foo">
    <div class="bar">
        Text
    </div>
</div>
```

We can qury the DOM like this:

```js
const myElement = document.querySelector('div.bar')
```

We can check if an element matches a selector.

```js
myElement.matches('div.bar') === true
```

And, concequently, the following would evaluate to false:

```js
myElement.matches('div.foo') === true
```

This will return the first match it encounters. However, if we try this ...

```js
const myElement = document.getElementsByClassName('bar')
```

... we will get a collection of elements.

```js
HTMLCollection [div.bar]
    0: div.bar
    length: 1
    __proto__: HTMLCollection
```

We can also use `.querySelectorAll()` if we want to get all occurrences, we can use:

```js
const myElements = document.querySelectorAll('.bar')
```

That will return:

```js
NodeList [div.bar]
    0: div.bar
    length: 1
    __proto__: NodeList
```



If we already have a reference to a parent element, we can just query that element’s children instead of the whole document. By narrowing down the context like this, we can simplify selectors and increase performance.

```js
const myChildElemet = myElement.querySelector('input[type="submit"]')
```

Instead of:

```js 
document.querySelector('#foo &gt; div.bar input[type="submit"]')
```

Then why use those other, less convenient methods like  `.getElementsByTagName()` at all? Well, one important difference is that the result of `.querySelector()` is *not live*, so when we dynamically add an element that matches a selector, the collection won’t update.

```js
const elements1 = document.querySelectorAll('div')
const elements2 = document.getElementsByTagName('div')
const newElement = document.createElement('div')

document.body.appendChild(newElement)
elements1.length === elements2.length // false
```

Another consideration is that such a live collection doesn’t need to have all of the information up front, whereas `.querySelectorAll()` immediately gathers everything in a static list, making it [less performant](https://www.nczonline.net/blog/2010/09/28/why-is-getelementsbytagname-faster-that-queryselectorall/).

#### Working with Nodelists

**There are two common pitfalls regarding `.querySelectorAll()`.** The first one is that we can’t call Node methods on the result and propagate them to its elements. Rather, we have to explicitly iterate over those elements. 

The other thing to look out for is that the returned value is a `NodeList`, not an `Array`. This means the usual Array methods are not available directly. There are a few corresponding `NodeList` implementations such as `.forEach`, which however are still not supported by any version of Internet Explorer. We have to convert the list to an array first, or “borrow” those methods from the Array prototype.

Using `Array.from()`:

```js
Array.from(myElements).forEach(doSomethingWithEachElement)

// Or prior to ES6
Array.prototype.forEach.call(myElements, doSomethingWithEachElement)

// Shorthand:
[].forEach.call(myElements, doSomethingWithEachElement)
```

Each element also has a couple of rather self-explanatory read-only properties referencing the “family”, all of which are live:

```js
myElement.children
myElement.firstElementChild
myElement.lastElementChild
myElement.previousElementSibling
myElement.nextElementSibling
```

As the [Element](https://developer.mozilla.org/en-US/docs/Web/API/element) interface inherits from the [Node](https://developer.mozilla.org/en-US/docs/Web/API/Node) interface, the following properties are also available:

```js
myElement.childNodes
myElement.firstChild
myElement.lastChild
myElement.previousSibling
myElement.nextSibling
myElement.parentNode
myElement.parentElement
```

Where the former only reference elements, the latter (except for `.parentElement`) can be any kind of node, e.g. text nodes. We can then check the [type](https://developer.mozilla.org/en-US/docs/Web/API/Node/nodeType) of a given node like e.g. 

```js
const myElement = document.querySelector('#foo div.bar')
myElement.firstChild.nodeType === 3
// this would be a text node
```

As with any object, we can check a node’s prototype chain using the `instanceof` operator:


```js
myElement.firstChild instanceof Text
```

### Modifying Classes and Attributes

Modifying classes of elements is as easy as:

```js
myElement.classList.add('foo')
myElement.classList.remove('bar')
myElement.classList.toggle('baz')
```

Element properties can be accessed like any other object’s properties:

```js
// Get an attribute value
const value = myElement.value

// Set an attribute as an element property
myElement.value = 'foo'

// Set multiple properties using Object.assign()
Object.assign(myElement, {
  value:'foo',
  id:'bar'})
  
// Remove an attribute
myElement.value = null
```

Note that there are also the methods `.getAttibute()`, `.setAttribute()` and `.removeAttribute()`. These methods directly modify the *HTML attributes* (as opposed to the DOM properties) of an element, thus causing a browser redraw (you can observe the changes by inspecting the element with your browser’s dev tools). Not only is such a browser redraw more expensive than just setting DOM properties, but these methods also can have [unexpected results](https://developer.mozilla.org/en-US/docs/Web/API/Element/setAttribute#Notes).

As a rule of thumb, only use them for attributes that don’t have a corresponding DOM property (such as colspan), or if you really want to “persist” those changes to the HTML (e.g. to keep them when cloning an element or modifying its parent’s  `.innerHTML`.

#### Adding CSS styles

CSS rules can be applied like any other property; note though that the properties are `camelCased` in JavaScript:

```js
myElement.style.marginLeft = '2em'
```

If we want certain values, we can obtain these via the `.style` property. However, this will only give us styles that have been explicitly applied. To get the computed values, we can use, `.window.getComputedStyle()`. It takes the element and returns a [CSSStyleDeclaration](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration) containing all styles from the element itself as well as those inherited from its parents:

```js
window.getComputedStyle(myElement).getPropertyValue('margin-left')
```

### Modifying the DOM

We can move elements around like this:

```js
// Append element1 as the last child of element2
element1.appendChild(element2)// Insert element2 as child of element 1, right before element3
element1.insertBefore(element2, element3)
```

If we don’t want to move the element, but insert a copy, we can clone it like this:

```js
// Create a cloneconst myElementClone = myElement.cloneNode()
myParentElement.appendChild(myElementClone)
```

The `.cloneNode()` method optionally takes a boolean as an argument if set to true, a *deep_copy*  will be created, meaning its children are also cloned.

Of course, we can just as well create entirely new elements or text nodes:

```js
const myNewElement = document.createElement('div')
const myNewTextNode = document.createTextNode('some text')
```

These can then be inserted as shown above. 

If we want to remove an element, we can’t do so directly, but we can remove children from a parent element, like so:

```js
myParentElement.removeChild(myElement)
```

This gives us a nice little work around, meaning can actually remove an element indirectly, by referencing its parent element:

```js
myElement.parentNode.removeChild(myElement)
```

#### Element properties

Every element also has the properties `.innerHTML`  and  `.textContent` (but also `.innerText`, which is similar to `.textContent`, but has some [important differences](http://perfectionkills.com/the-poor-misunderstood-innerText/)). 

These hold the HTML and plain text content respectively. They are writable properties, meaning we can modify elements and their contents directly:

// Replace the inner HTML

```html+javascript

myElement.innerHTML = '<a href="www.craftacademy.se">Craft Academy</a>'

```

Appending markup to the HTML, as shown above, is usually a bad idea though, as we’d lose any previously made property changes on the affected elements and bound event listeners. Setting 

the 

.innerHTML

 is good for completely throwing away markup and replacing it with something else, e.g. server-rendered markup. So appending elements would better be done like so:

const link = document.createElement('a')const text = document.createTextNode('continue reading...')const hr = document.createElement('hr')

link.href ='foo.html'
link.appendChild(text)
myElement.appendChild(link)
myElement.appendChild(hr)

With this approach, however, we’d cause two browser redraws — one for each appended element — whereas changing 

the 

.innerHTML

 only causes one. As a way around this performance issue we can first assemble all nodes in a [DocumentFragment](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment), and then just append that single fragment:

const fragment = document.createDocumentFragment()

fragment.appendChild(text)
fragment.appendChild(hr)
myElement.appendChild(fragment)

### Listening to events

This is possibly the best-known way to bind an event listener:

myElement.onclick =function onclick (event){
  console.log(event.type +' got fired')}

But this should generally be avoided. Here, 

.onclick is a property of the element, meaning that you can change it, but you cannot use it to add additional listeners — by reassigning a new function you’ll overwrite the reference to the old one.

Instead, we can use the much 

mightier 

.addEventListener()

 method to add as many events of as many types as we like. It takes three arguments: the event type (such as 

click), a function that gets called whenever the event occurs on the element (this function gets passed an event object), and an optional config object which will be explained further below.

myElement.addEventListener('click',function(event){
  console.log(event.type +' got fired')})

myElement.addEventListener('click',function(event){
  console.log(event.type +' got fired again')})

Within the listener function, 

event.target refers to the element on which the event was triggered (as 

does 

this

, unless of course, we’re using an [arrow function](https://www.sitepoint.com/es6-arrow-functions-new-fat-concise-syntax-javascript/)). Thus you can easily access its properties like so:

// The `forms` property of the document is an array holding// references to all formsconst myForm = document.forms[0]const myInputElements = myForm.querySelectorAll('input')

Array.from(myInputElements).forEach(el =&gt;{
  el.addEventListener('change',function(event){
    console.log(event.target.value)})})

#### Preventing default actions

Note 

that 

event

 is always available within the listener function, but it is good practice to explicitly pass it in 

anyway when needed (and we can name it as we like then, of course). Without elaborating on the [Event](https://developer.mozilla.org/en-US/docs/Web/API/Event) interface itself, one particularly noteworthy method 

is 

.preventDefault()

, which will, well, prevent the browser’s default behavior, such as following a link. Another common use-case would be to conditionally prevent the submission of a form if the client-side form-validation fails.

myForm.addEventListener('submit',function(event){const name =this.querySelector('#name')if(name.value ==='Donald Duck'){alert('You gotta be kidding!')
    event.preventDefault()}})

Another important event method 

is 

.stopPropagation()

, which will prevent the event from bubbling 

up the DOM. This means that if we have a propagation-stopping click listener (say) on an element, and another click listener on one of its parents, a click event that gets triggered on the child element won’t get triggered on the parent — otherwise, it would get triggered on both.

Now 

.addEventListener() takes an optional config object as a 3rd argument, which can have any of the following boolean properties (all of which default to 

false):

- 

capture: The event will be triggered on the element before any other element beneath it in the DOM (event capturing and bubbling is an article in its own right, for more 

details have a look [here](http://javascript.info/tutorial/bubbling-and-capturing))
- 

once: As you might guess, this indicates that the event will get triggered only once
- 

passive: This means that 

event.preventDefault() will be ignored (and usually yield a warning in the console)

The most common option 

is 

.capture

; in fact, it is so common that there’s a shorthand for this: instead of specifying it in the config object, you can just pass in a boolean here:

myElement.addEventListener(type, listener,true)

Event listeners can be removed 

using 

.removeEventListener()

, which takes the event type and a reference to the callback function to be removed; for example, the 

once option could also be implemented like

myElement.addEventListener('change',function listener (event){
  console.log(event.type +' got triggered on '+this)this.removeEventListener('change', listener)})

#### Event delegation

Another useful pattern is _event delegation_: say we have a form and want to add 

a 

change

 event listener to all of 

its 

input

 children. One way to do so would be iterating over them 

using 

myForm.querySelectorAll('input')

 as shown above. However, this is unnecessary when we can just as well add it to the form itself and check the contents 

of 

event.target

.

myForm.addEventListener('change',function(event){const target = event.target
  if(target.matches('input')){
    console.log(target.value)}})

Another advantage of this pattern is that it automatically accounts for dynamically inserted children as well, without having to bind new listeners to each.

### Animation

Usually, the cleanest way to perform animations is to apply CSS classes with a 

transition property, or use 

CSS 

@keyframes

. But if you need more flexibility (e.g. for a game), this can be done with JavaScript as well.

The naive approach would be to have a 

window.setTimeout() function call itself until the desired animation is completed. However, this inefficiently forces rapid document reflows; and this _layout thrashing_ can quickly lead to stuttering, 

expecially on mobile devices. 

Intead, we can sync the updates 

using 

window.requestAnimationFrame()

 to schedule all current changes to the next browser repaint frame. It takes a callback as 

argument which receives the current (high res) timestamp:

const start = window.performance.now()const duration =2000

window.requestAnimationFrame(function fadeIn (now)){const progress = now - start
  myElement.style.opacity = progress / duration

  if(progress &lt; duration){
    window.requestAnimationFrame(fadeIn)}}

This way we can achieve very smooth animations