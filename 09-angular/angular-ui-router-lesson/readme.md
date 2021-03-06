# Front-end Routing with UI-Router

### Objectives
- Build a SPA with multiple pages
- Describe the when to consider server-side routing and when to consider front-end routing

### Preparation

- Build a basic Angular app
- Interact with an API
- Download the [starter code](https://github.com/wdi-hk-11/lesson-angular-ui-router)

## Intro (5 mins)

Routing, as you've seen in multiple frameworks and languages, is adding in the ability to render different pages in a application – but in a single-page app, how can we have multiple pages? In Angular, it comes down to storing all our views on our main page and turning them on and off as we need.

But what's the benefit? Why even make it single page? Why add that complexity? The main use case for front-end frameworks is added speed – by loading everything upfront, and just switching sections on and off, our page will seem wonderfully speedy because we'll be skipping quite a few steps that a more traditional framework has to run through.

Now, Angular comes with a basic routing mechanism, ``ngRoute``, which you can read about here if you want to – https://docs.angularjs.org/api/ngRoute/service/$route

But today we're looking at an even more beefed up router: a third-party plugin called ``ui-router`` – https://github.com/angular-ui/ui-router

**Our ultimate goal is to build out two pages – a main Todo list and an Archive page for all the Todos we've completed.**

Let's walk through it.

## Six Steps to UI-Router - Codealong (40 mins)

Because of the nature of what we're building today - our URL will be telling our application what particular views to render - we can't just open our file with ``file://`` anymore. We'll have to load up a quick server to render our initial HTML file.

We could do that a number of different ways, but for today, let's use a simple little node library, [http-server](https://github.com/nodeapps/http-server).

In your Terminal, run a quick command to install. The `-g` installs it globally on our computer remember, so we can use it in any directory.

```bash
$ npm install http-server -g
```

When that's done navigate to our starter app folder and run this:

```bash
$ http-server .
```

From now on, rather than opening the HTML file directly, we can navigate to ``http://0.0.0.0:8080`` or ``http://localhost:8080``.

### Step One: Get ui-router

We'll need the UI-Router source. It's not an official, core library, and it's not hosted on Google's site. CDNJS has it (https://cdnjs.com/libraries/angular-ui-router), or you can download it from GitHub and include it yourself. But then you can also use `bower` too:

```bash
bower install angular --save
bower install angular-ui-router --save
```

Then we need to include both files plus our code in our HTML:

```html
		<script src="bower_components/angular/angular.js"></script>
		<script src="bower_components/angular-ui-router/release/angular-ui-router.js"></script>
		<script src="js/app.js"></script>
		<script src="js/TodosController.js"></script>
		<link rel="stylesheet" href="css/style.css">
```

### Step Two: Adding a Dependency

Because we're adding in a new library, it'll be a dependency – we'll need to make sure Angular knows about our library, so we can use it. If you haven't used any external libraries yet, rejoice in that we're finally going to put _something_ in those empty brackets in our ``app.js``.


```javascript
// in app.js
angular
  .module('TodosApp', ['ui.router']);
```

``'ui.router'`` just happens to be what the library is called in it's source. Most libraries will tell you what to write here in their documentation, and if you need more than one, just list them like any array.

#### Wait, what? Routing at the frontend?

We have previously looked at routing in the Ruby on Rails lessons. Rails has a `router` component which intercepts user requests and decides which controller method should be invoked to handle a request. Here we are looking at a `router` for AngularJS. So does it mean we have routes in both frontend and backend? What is a route exactly?

A route, in general, is just the path you take to get somewhere. That's not specific to backend web development, but it's one of those words we've latched on because it's a good description – when you're changing URL, when that location bar changes, you're on a new route.

Our router just sets up which routes we want to exist and points our code where to make it happen.

This means our Angular app can simulate having multiple pages, which gives us the ability to make more complex applications...which is awesome!

Let's open up our ``app.js`` and add some routes.

### Step Three: Add Some Configuration

In ``app.js``, we had this:

```javascript
// in app.js
angular
  .module('TodosApp', ['ui.router']);
```

Let's add on to it:

```javascript
// in app.js
angular
  .module('TodosApp', ['ui.router'])
  .config(MainRouter);
```

Of course, now we need a ``MainRouter()`` function, so let's build one:

```javascript
function MainRouter($stateProvider, $urlRouterProvider) {
  // ROUTE
}
```

Those arguments are necessary parts for our router to do its work. We'll see what they do with this next chunk.

### Step Four: Add Some Routes

Because in Angular we're not really changing locations (single-page apps, here), lets, instead of calling them _routes_, call them **states**. Same idea as routes, but we're just trying to be more descriptive. We're changing the current _state_ of the app, as in a snapshot of the stuff we're looking at and working with, at a particular moment.

```javascript
function MainRouter($stateProvider, $urlRouterProvider) {
  $urlRouterProvider.otherwise('/');  // if no matching path, always route to the root

  $stateProvider
    .state('home', {
      url: "/",
      templateUrl: "home.html",
    });
}
```

That weird ``$stateProvider`` argument comes from our library, and it allows us to add a state to our application.

We define a **name** for the state. This is important because it's how we can refer to it later.

We also define a **relative url** for each state to tell the browser how to simulate navigating different pages. A ``/`` here says it'll be the default homepage, basically.

And finally, we add a **templateURL**, which is sort of a partial HTML file. We'll fill a partial with _just_ the code we'd need to change on the page, here.  Remember, it's just a part of a larger HTML page with parts that we can hide.

Now, before our route can work, we've got to extract some of our view into that partial. Let's do that.

### Step Five: Building Partials

Go over to our ``index.html``, and we'll see we've already got a container for our main list.

```html
<div class="wrapper" ng-controller="TodosController"><!-- a bunch of list-related view stuff --></div>
```

Grab everything inside that div and make a _new_ file. You can call it whatever you like but make it obvious. For this exercise, we'll call it ``home.html``

```bash
$ touch home.html
```

And paste all that view code inside. Now you've got a partial, and all we have left to do is tell our ``index.html`` where we want to put it.

In that ``div.wrapper``, on our ``index.html``, we'll add a new directive: ``ui-view``.

```html
<div class="wrapper" ui-view ng-controller="TodosController"><!-- a bunch of list-related view stuff --></div>
```

And since our route is a default route at ``/``, and our ``templateUrl`` is already ``home.html``, it should actually work!

Now you can visit your application again, but this time, please go to this URL instead: ```http://localhost:8080/#/```

### Step Six: One More State!

Of course, that's exactly what we were looking at before, but _now_, we have the ability to switch out that view with different partials, depending on our _state_.

So let's make things interesting and add another state in here. Let's make a state for when we're looking at an archived list.

```javascript
// in app.js again
function MainRouter($stateProvider, $urlRouterProvider) {
  $stateProvider
    .state('home', {
      url: "/",
      templateUrl: "home.html",
    })
    .state('archive', {
      url: "/archive",
      templateUrl: "archive.html",
    });

}
```

We'll need another partial for ``archive.html`` and for that one, instead of listing all our todos, let's just list the completed ones.

Our new partial will be almost exactly the same as our last so duplicate that file. Inside, find our ``ng-repeat``:

```html
<li ng-repeat="todo in remainingTodos()">
```

and change that expression inside `ng-repeat` to:

```html
<li ng-repeat="todo in completedTodos()">
```

You can now check out the new page at: ```http://localhost:8080/#/archive```

We're 10 seconds away from seeing something awesome. We need one more thing.

### Step Seven: A Navbar!

In order to jump between one view and the other, we need _links_! But not normal links because we're not changing pages. Luckily, `ui.router` gives us a custom directive. Open up both your new partials and beneath the head, let's add a `nav` with a few `a`'s

```html
<header><!-- stuff --></header>
<nav class='tabs'>
  <a ui-sref="home">My List</a>
  <a ui-sref="archive">Archives</a>
</nav>
```

That custom directive, ``ui-sref`` is like ``href``, but referencing _states_ instead. That came with our library, and **the text we're putting in there is just the names of the states we defined**.

You already have a little CSS in your ``style.css`` to make it look nice, something like:

```css
nav.tabs {
  background: #4d5d70;
  max-width: 70%;
  margin: 0 auto;
}
nav.tabs a {
  display: inline-block;
  background: rgba(255,255,255,0.7);
  color: black;
  padding: 10px 20px;
  margin-right: 1px;
}
```

Check it out. Click through and jump from page to page. Super awesome, yeah?

### Helpful Extra - Which state am I on?

``ui.router`` actually gives us another really useful custom directive. Throw it on whichever links are using ``ui-sref``:

```html
<nav class='tabs'>
  <a ui-sref-active="active" ui-sref="home">List</a>
  <a ui-sref-active="active" ui-sref="archive">Archive</a>
</nav>
```
This is a really nice helper that will apply the class of "active" (or whatever you put in quotes) to the link that's currently active, depending on what state you're looking at.

Now you could add some css, like:

```css
nav.tabs a.active { background: rgba(255,255,255,1);}
```

And suddenly, your interface makes a ton more sense. Super helpful.

## Independent Practice (20 minutes)

Having multiple states is really useful, obviously – we can start making a much more complex Angular application.

**What other states would be good to add to your app?** Try adding an about page to start, or even crazier, maybe adding an extra state to be able to _edit_ a todo. Take the next 20 minutes to try, and we'll be here to help if you need it.

## Conclusion (5 mins)
- What's a router? What's it for?
- How do we add routes to our Angular application?
