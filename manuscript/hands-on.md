# Hands-on
In the following sections we will add some models to our app, define the interactions between them, and create an interface to add friends and the articles they borrow from us.

## Adding a friend resource
The main model of our application will be called **Friend**. It represents the people who will borrow articles from us.

Let's add it with the **resource** generator.

{title="", lang="bash"}
~~~~~~~~
$ ember generate resource friends firstName:string lastName:string  \
       email:string twitter:string totalArticles:number
installing model
  create app/models/friend.js
installing model-test
  create tests/unit/models/friend-test.js
installing route
  create app/routes/friends.js
  create app/templates/friends.hbs
updating router
  add route friends
installing route-test
  create tests/unit/routes/friends-test.js
~~~~~~~~

If we open **app/models/friend.js** or **app/routes/friends.js**, we
will see that they have a similar structure.

{title="Object Structure", lang="JavaScript"}
~~~~~~~~
import  Foo from 'foo';

export default Foo.extend({
});
~~~~~~~~

What is that? **ES6 Modules**!  As mentioned previously, **ember CLI** expects us
to write our code using ES6 Modules. `import Foo from 'foo'` consumes the
default export from the package `foo` and assigns it to the variable `Foo`.
We use `export default Foo.extend...` to define what our module will
expose. In this case we will export a single value, which will be a subclass
of `Foo`.

T> For a better understanding of ES6 modules, visit [ http://jsmodules.io/](http://jsmodules.io).

Now let's look at the model and route.

{title="app/models/friend.js", lang="JavaScript"}
~~~~~~~~
// We import the default value from ember-data into the variable DS.
//
// ember data exports by default a namespace (known as DS) that exposes all the
// classes and functions defined in http://emberjs.com/api/data.

import DS from 'ember-data';

// Define the default export for this model, which will be a subclass
// of DS.Model.
//
// After this class has been defined, we can import this subclass doing:
// import Friend from 'borrowers/models/friend'
//
// We can also use relative imports. So if we were in another model, we
// could have written
// import Friend from './friend';

export default DS.Model.extend({

  // DS.attr is the standard way to define attributes with ember data
  firstName: DS.attr('string'),


  // Defines an attribute called lastName of type **string**
  lastName: DS.attr('string'),


  // ember data expects the attribute **email** on the friend's payload
  email: DS.attr('string'),

  twitter: DS.attr('string'),
  totalArticles: DS.attr('number')
});
~~~~~~~~


{title="app/routes/friends.js", lang="JavaScript"}
~~~~~~~~
// Assigns the default export from **ember** into the variable Ember.
//
// The default export for the ember package is a namespace that
// contains all the classes and functions for Ember that are specified in
// http://emberjs.com/api/

import Ember from 'ember';

// Defines the default export for this module. For now we will not
// add anything extra, but if we want to use a Route **hook** or
// **actions** this would be the place.

export default Ember.Route.extend({
});
~~~~~~~~

In a future version of **ember** we might be able to be more explicit
about the things we want to use from every module. Instead of writing
**import Ember from 'ember'**, we could have **import { Route } from
'ember/route'** or **import { Model } from 'ember-data/model'**. This
is currently possible in **ES6** using [Named Imports and
Exports](http://jsmodules.io).

T> The following RFC [https://github.com/emberjs/rfcs/pull/68](https://github.com/emberjs/rfcs/pull/68)
T> offers more context on the move to ES6 modules.

What about tests? If we open the test files, we'll see that they are
also written in ES6. We'll talk about that in a later chapter. Now
let's connect to a backend and display some data.

## Connecting with a Backend
We need to consume and store our data from somewhere. In this case, we
created a public API under **http://api.ember-cli-101.com** with
**Ruby on Rails**. The following are the API end-points.


|Verb   | URI Pattern          |
|-------|----------------------|
|GET    | /api/articles        |
|POST   | /api/articles        |
|GET    | /api/articles/:id    |
|PATCH  | /api/articles/:id    |
|PUT    | /api/articles/:id    |
|DELETE | /api/articles/:id    |
|GET    | /api/friends         |
|POST   | /api/friends         |
|GET    | /api/friends/:id     |
|PATCH  | /api/friends/:id     |
|PUT    | /api/friends/:id     |
|DELETE | /api/friends/:id     |

If we do a **GET** request to **/api/friends**, we will get a list of all our friends.

~~~~~~~~
# The following output might be different for every run since the data
# in the API is changing constantly.
#
$ curl http://api.ember-cli-101.com/api/friends.json | python -m json.tool
{
    "friends": [
        {
            "email": "test@gmail.com",
            "first_name": "jon",
            "id": 1,
            "last_name": "snow",
            "twitter": "foo"
        }
    ]
}
~~~~~~~~

T> Piping JSON data to **python -m json.tool** is an easy way to pretty
print JSON data in our console using python's JSON library. It's very
useful if we want to quickly debug JSON data.

When returning a list, **ember data active-model-adapter** expects the root name of the JSON
payload to match the name of the model but pluralized (**friends**) and followed by
an array of objects. This payload will help us to populate
**ember data** store.

If we want to run the server by ourselves or create our own instance
on **Heroku**, we can use the **Heroku Button** added to the repository
[borrowers-backend](https://github.com/abuiles/borrowers-backend).

Once we have created our own instance on **Heroku**, we need to install
[Heroku Toolbelt](https://toolbelt.heroku.com/) and check our
application's log with `heroku logs -t --app my-app-name`.

## A word on Adapters

By default, ember data uses the **DS.JSONAPIAdapter**[^jsonAdapter],
which expects your API to follow
[http://jsonapi.org/](http://jsonapi.org/), a specification
for building APIs in JSON. In our example, however, we will work with an
API written in **Ruby on Rails** using the active model serializer,
which uses a different convention for keys and naming. Everything is
in **snake_case**.

To do this we need to use a different adapter which is called **ember
data active model adapter**, which is modeled after [active_model_serializers](https://github.com/rails-api/active_model_serializers).

This is widely used in the **Ruby on Rails** world and basically helps
build the **JSON** that the API will return.

There are a bunch of different adapters for different projects and
frameworks.

Some of them are:

- [ember-data-django-rest-adapter](https://github.com/toranb/ember-data-django-rest-adapter)
- [ember-data-tastypie-adapter](https://github.com/escalant3/ember-data-tastypie-adapter)
- [emberfire: FireBase adapter](https://github.com/firebase/emberfire)

We can find a longer list of adapters if we search GitHub for [ember-data adapters](https://github.com/search?q=ember-data+adapter&ref=opensearch).

[^jsonAdapter]: We recommend going through the documentation to get more insights on this adapter [DS.JSONAPIAdapter](http://emberjs.com/api/data/classes/DS.JSONAPIAdapter.html).
[^activeModelAdapter]: Repository for [DS.ActiveModelAdapter.html](https://github.com/ember-data/active-model-adapter).

#### Specifying our own adapter

As mentioned in the previous chapter, if we are using **ember data**
it will **resolve** to the **DS.JSONAPIAdapter** unless we specify
something else.

To see it in action, let's play with the console and examine how **ember** tries
to **resolve** things.

First we need to go to  `config/environment.js` and uncomment `ENV.APP.LOG_RESOLVER`[^uncomment-resolver]. It should look like:

{title="config/environment.js", lang="JavaScript"}
~~~~~~~~
  if (environment === 'development') {
    ENV.APP.LOG_RESOLVER = true;
    ENV.APP.LOG_ACTIVE_GENERATION = true;
    // ENV.APP.LOG_TRANSITIONS = true;
    // ENV.APP.LOG_TRANSITIONS_INTERNAL = true;
    ENV.APP.LOG_VIEW_LOOKUPS = true;
  }
~~~~~~~~

That line will log whatever **ember** tries to "find" to the browser's
console. If we go to [http://localhost:4200](http://localhost:4200)
and open the console, we'll see something like the output below:

{title=""}
~~~~~~~~
[ ] router:main .............. borrowers/main/router
[ ] router:main .............. borrowers/router
[✓] router:main .............. borrowers/router
[ ] application:main ......... borrowers/main/application
[ ] application:main ......... undefined
[ ] application:main ......... borrowers/application
[ ] application:main ......... borrowers/applications/main
[ ] application:main ......... undefined
~~~~~~~~

That's the **ember** resolver trying to find things. We don't need to
worry about understanding all of it right now.

Coming back to the **adapter**, if we open the **ember-inspector** and
grab the instance of the **application** route

![ember-inspector](images/ember-inspector-1.png)

T> We can grab almost any instance of a Route, Controller, View or Model with
T> the **ember-inspector** and then reference it in the console with the
T> `$E` variable. This variable is reset every time the browser gets refreshed.

With the **application route** instance at hand, let's have some fun.

Let's examine what happens if we try to find all our **friends**:

{title=""}
~~~~~~~~
$E.store.findAll('friend')
[ ] model:friend   .............borrowers/friend/model
[ ] model:friend   .............borrowers/models/friend
[✓] model:friend   .............borrowers/models/friend
[✓] model:friend   .............borrowers/models/friend
[✓] model:friend   .............borrowers/models/friend
[ ] adapter:friend .............borrowers/friend/adapter
[ ] adapter:friend .............undefined
[ ] adapter:friend .............borrowers/adapters/friend
[ ] adapter:friend .............undefined
[ ] adapter:application ........borrowers/application/adapter
[ ] adapter:application ........undefined
[ ] adapter:application ........borrowers/adapters/application
[ ] adapter:application ........undefined
~~~~~~~~

First, the **resolver** tries to find an adapter at the model level:

{title=""}
~~~~~~~~
[ ] adapter:friend .............borrowers/friend/adapter
[ ] adapter:friend .............undefined
[ ] adapter:friend .............borrowers/adapters/friend
[ ] adapter:friend .............undefined
~~~~~~~~

We can use this if we want to change the default behavior of **ember data**. For example, changing the way an URL is generated for a
resource.

Second, if no adapter is specified for the model, then the
**resolver** checks if we specified an **application** adapter. As we
can see, it returns **undefined**, which means we didn't specify one:

{title=""}
~~~~~~~~
[ ] adapter:application ........borrowers/application/adapter
[ ] adapter:application ........undefined
[ ] adapter:application ........borrowers/adapters/application
[ ] adapter:application ........undefined
~~~~~~~~

Third, if no model or application adapter is found, then **ember data**
falls back to the default adapter, the **JSONAPIAdapter**. We can
check the implementation for this directly in the
[adapterFor](https://github.com/emberjs/data/blob/131119/packages/ember-data/lib/system/store.js#L1552)
function in **ember data**.

{#pods-adapter}
I>We can see that there is a look up for the friend and application
I>adapter in two places **borrowers/friend/adapter**,
I>**borrowers/adapters/friend**, **borrowers/application/adapter** and
I>**borrowers/adapters/application**. ember CLI allows us to group
I>things that are logically related under a single directory. This
I>structure is known as PODS. We'll work with the normal structure first, and
I>at the end of the book we'll rewrite a part of our code to be structured
I>under PODS.

Since we want to work with a different adapter, we need to tell
**ember** to do so. In this case we want the **ember data active model
adapter** as our application adapter. First we need to install the
adapter and then user **ember CLI** generator for adapters.

We can install active model adapter running the following command:

{title="Installing ember data Active Model Adapter", lang="bash"}
~~~~~~~~
$ ember install active-model-adapter
~~~~~~~~

And then run `ember g adapter application` to create an application adapter:

{title="", lang="bash"}
~~~~~~~~
$ ember g adapter application
installing adapter
  create app/adapters/application.js
installing adapter-test
  create tests/unit/adapters/application-test.js
~~~~~~~~

T> **ember g** is a short version of **ember generator**. We'll use both interchangeably to get used to the syntax.

It will create a file like the following:

{title="app/adapters/application.js", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';

export default DS.RESTAdapter.extend({
});
~~~~~~~~

But we don't want to use the **DS.RESTAdapter** so let's change that
file to import the active model adapter:

{title="app/adapters/application.js", lang="JavaScript"}
~~~~~~~~
//
// Import the default export from active-model-adapter
//
import ActiveModelAdapter from 'active-model-adapter';

export default ActiveModelAdapter.extend({
  namespace: 'api'
});
~~~~~~~~

We now specify our **adapter** and also pass a property **namespace**. The **namespace** option tells **ember data** to namespace all our **API** requests under `api`. So if we ask for the collection `friend`, **ember data** will make a request to `/api/friends`. If we don't have that, then it will be just `/friends`.

Let's restart the ember CLI server and then go to back to the browser's console, grab the **application route** instance again from the **ember-inspector**, and ask the store for our friends.

{title=""}
~~~~~~~~
$E.store.findAll('friend')
[ ] adapter:friend ............. borrowers/friend/adapter
[ ] adapter:friend ............. undefined
[ ] adapter:friend ............. borrowers/adapters/friend
[ ] adapter:friend ............. undefined
[ ] adapter:application ........ borrowers/application/adapter
[ ] adapter:application ........ borrowers/adapters/application
[✓] adapter:application ........ borrowers/adapters/application
[✓] adapter:application ........ borrowers/adapters/application
[✓] adapter:application ........ borrowers/adapters/application
GET http://localhost:4200/api/friends 404 (Not Found)
~~~~~~~~

This time, when the **resolver** tries to find an adapter, it works because we have one specified under **applications/adapters**. We also see a failed **GET** request to **api/friends**. It fails because we are not connected to any backend yet.

We need to stop the **ember server** and start again, but this time let's specify that we want all our **API** requests to be proxy to **http://api.ember-cli-101.com**. To do so we use the option **--proxy**:

{title="Running ember server", lang="bash"}
~~~~~~~~
$ ember server --proxy http://api.ember-cli-101.com
Proxying to http://api.ember-cli-101.com
Livereload server on port 35729
Serving on http://0.0.0.0:4200
~~~~~~~~

Let's go back to the console and load all our friends, but this time
logging something with the response:

~~~~~~~~
$E.store.findAll('friend').then(function(friends) {
  friends.forEach(function(friend) {
    console.log('Hi from ' + friend.get('firstName'));
  });
});

XHR finished loading: GET "http://localhost:4200/api/friends".
Hi from jon
~~~~~~~~

If we see 'Hi from' followed by a name, we have successfully specified
our application adapter and connected to the backend. The output might
be different every time we run it since the API's data is changing.

T> We use the name of our model in singular form. This is important.
T> We always reference the models in their singular form.


[^uncomment-resolver]: [Enable ENV.APP.LOG_RESOLVER](https://github.com/abuiles/borrowers/commit/76022770935e9d0d9ab37e2cb0ff943fec47721a).

## Listing our friends

Now that we have successfully specified our own **adapter** and made a
request to our **API**, let's display our friends.

By convention, the entering point for rendering a list of any kind of
resource in web applications is called the **Index**. This normally
matches to the **Root** URL of our resource. With our friends example,
we do so on the backend through the following end-point
[http://api.ember-cli-101.com/api/friends.json](http://api.ember-cli-101.com/api/friends).
If we visit that URL, we will see a **JSON** list with all our friends.

T> If we are using Firefox or Chrome, we can use JSONView to have a readable version of **JSON** in our browser.
T> [Firefox Version](http://jsonview.com) or [Chrome Version](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc).

In our ember application, we need to specify somehow that every
time we go to URL **/friends**, then all our users should be loaded and
displayed in the browser. To do this we need to specify a **Route**.

[Routes](http://emberjs.com/api/classes/Ember.Route.html) are one of
the main parts of **ember**. They are in charge of everything
related to setting up state, bootstrapping objects, specifying which
template to render, etc. In our case, we need a **Route** that will
load all our friends from the **API** and then make them available to
be rendered in the browser.

### Creating our first Route.

First, if we go to **app/router.js**, we will notice that the **resource** generator added **this.route('friends');**.

{title="app/router.js", lang="JavaScript"}
~~~~~~~~
// ...

Router.map(function() {
  this.route('friends');
});

// ...
~~~~~~~~

We specify the **URLs** we want in our application inside the function
passed to **Router.map**. There, we can declare new routes calling **this.route**.

Let's check the **Routes** that we have currently defined. To do so,
open the **ember-inspector** and click on **Routes**.

![ember-inspector](images/routes-1.png)

By default, **ember** creates 4 routes:

- ApplicationRoute
- IndexRoute
- LoadingRoute
- ErrorRoute

We also see that the friends route was added with
**this.route('friends')**. if we pass a function as second
or third argument, **ember** will create an **Index**,
**Loading**, and **Error** **Route**.

We can modify our route passing an empty function

{title="app/router.js", lang="JavaScript"}
```
this.route('friends', function() {
});
```

And then if we check the inspector, the children routes are automatically
generated for the friends route.

I> When we define a route leaving out the empty function, then the
I> children routes are not generated.

Since we just added a friends index route, visiting
[http://localhost:4200/friends](http://localhost:4200/friends) should
be enough to list all our friends. But if we actually go there, the
only thing we will see is a message with **Welcome to Ember**.

Let's go to **app/templates/friends.hbs** and change it to look like the following:

{title="app/templates/friends.hbs", lang="handlebars"}
~~~~~~~~
<h1>Friends Route</h1>
{{outlet}}
~~~~~~~~

For people familiar with Ruby on Rails, **{{outlet}}** is very similar
to the word **yield** in templates. Basically it allows us to put content
into it. If we check the application templates
(**app/templates/application.hbs**), we'll find the following:

{title="app/templates/application.hbs", lang="handlebars"}
~~~~~~~~
<h2 id='title'>Welcome to Ember</h2>

{{outlet}}
~~~~~~~~

When ember starts, it will render the **application template** as the
main template. Inside **{{outlet}}**, it will render the template
associated with the **Route** we are visiting. Then, inside those
templates, we can have more **{{outlet}}** to keep rendering content.

In our friends scenario, **app/templates/friends.hbs** will get
rendered into the application's template **{{outlet}}**, and then
it will render the **friends index** template into
**app/templates/friends.hbs** **{{outlet}}**.

To connect everything, let's create an index template and list all our
friends. Let's run the route generator **ember g route friends/index**
and put the following content inside
**app/templates/friends/index.hbs**:

{title="app/templates/friends/index.hbs", lang="handlebars"}
~~~~~~~~
<h1>Friends Index</h1>

<ul>
  {{#each model as |friend|}}
    <li>{{friend.firstName}} {{friend.lastName}}</li>
  {{/each}}
</ul>
~~~~~~~~

T> We remove **{{outlet}}** from **app/templates/friends/index.hbs**
T> since the **friends index route** won't have any nested route.

Next, we need to specify in the **friends index route** the data we
want to load in this route. The part in charge of loading the data
related to a route is called the model hook. Let's add one to
**app/routes/friends/index.js** as follows:

{title="app/routes/friends/index.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  //
  // Here we are using ES6 syntax for functions!
  // We can use this out of the box with ember-cli
  // thanks to the addon ember-cli-babel
  //
  // To learn more about ES6, check http://s.abuiles.com/bwWo
  //
  model() {
    return this.store.findAll('friend');
  }
});
~~~~~~~~

I> Remember that the **Route** is responsible for everything related
I> to setting up the application state.

If we visit
[http://localhost:4200/friends](http://localhost:4200/friends) we will
see something like the following along with a list of our friends:

![outlets](images/outlet.png)

We played previously with **store.findAll** to load all our friends from
the **API** and that's what we are doing in the model hook. **ember**
waits for this call to be completed. When the data is loaded, it
automatically creates a friends index controller (or we can define a
controller explicitly) and sets the property **model** with the content
returned from the **API**.

I>
I> Controller will be deprecated in next versions of ember
I> so we'll explore how to work without them in upcoming chapters.
I>

We can also use `store.findRecord` or `store.queryRecord` if we want
to load a record by a given id or appending query parameters to the
request URL, such as **this.store.findRecord('friend', 1)** or
**this.store.queryRecord('friend', {active: true})**, which creates
the following requests to the API **/api/friends/1** or
**/api/friends?active=true**.

When we do **{{#each model as |friend|}}**, ember takes
every element of the collection and set it as **friend**, the
collection which is what the model hook returned is referenced as
**model**.

If we want to display the total number of friends and the **id** for
every friend, then we just need to reference **model.length** in the
template and inside the each use **friend.id**:

{title="app/templates/friends/index.hbs", lang="handlebars"}
~~~~~~~~
<h1>Friends Index</h1>
{{! The context here is the controller}}
<h2>Total friends: {{model.length}}</h2>

<ul>
  {{#each model as |friend|}}
    <li>{{friend.id}} - {{friend.firstName}} {{friend.lastName}}</li>
  {{/each}}
</ul>
~~~~~~~~

Again, because our model is a collection and it has the property
**length**, we can just reference it in the template as
**model.length**.

## Adding a new friend

We are now able to see which friends have borrowed things from us,
but we don't have a way to add new friends. The next step is to
build support for adding a new friend.

To do this we'll need a **friends new route** under the route friends,
which will handle the URL **http://localhost:4200/friends/new**.

T> By convention, the URL for adding a new resource is
T> **/resource_name/new**. For editing a resource, use
T> **/resource_name/:resource_id/edit** and for showing a resource, use
T> **/resource/:resource_id**.

To add the new route, let's run the route generator with the
parameters **friends/new**:

~~~~~~~~
$ ember g route friends/new
installing route
  create app/routes/friends/new.js
  create app/templates/friends/new.hbs
updating router
  add route friends/new
installing route-test
  create tests/unit/routes/friends/new-test.js
~~~~~~~~

If we go to **app/router.js** we'll see that the **new** route was nested
under the route **friends**:

{title="app/router.js", lang="JavaScript"}
~~~~~~~~
this.route('friends', function(){
  this.route('new');
});
~~~~~~~~

Let's add the following content on the new template:

{title="app/templates/friends/new.hbs", lang="handlebars"}
~~~~~~~~
<h1>Add a New Friend</h1>
~~~~~~~~

And then navigate to http://localhost:4200/friends/new:

![friends new route](images/friends-new-route.png)

Notice how the **friends new route** got rendered in the
**{{outlet}}** inside **app/templates/friends.hbs**.


We got our **route** and **template** wired up, but we can't add
friends yet. We need to set a new friend instance as the model of the
**friends new route**, and then create a form that will bind to the
friend's attributes, and save the new friend in our backend.

Following the logic we used in the **friends index route**, we need to
return the model that will be the context of the **friends new route**.

We need to edit **app/routes/friends/new.js** and add the following
model hook:

{title="app/routes/friends/new.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.store.createRecord('friend');
  }
});
~~~~~~~~

We have been using the **this.store** without knowing what it is. The
[Store](http://emberjs.com/api/data/classes/DS.Store.html) is an **ember data** class in charge of managing everything related to our model's data. It knows about all the records we currently have loaded in our application and it has some functions that will help us to find, create, update, and delete records. During the whole application life cycle there is a unique instance of the **Store**, and it is injected as a property into every **Route**, **Controller**, **Serializer**, and **adapter** under the key **store**. That's why we have been calling **.store** in our **Routes** and **Controllers**.

T> The following shows how the store is injected in every instance: [store_injections](https://github.com/emberjs/data/blob/v1.13.5/packages/ember-data/lib/initializers/store-injections.js).


The method we are using on the model hook **store.createRecord**
creates a new record in our application **store**, but it doesn't save it to
the backend. What we will do with this record is set it as the
**model** of our **friends new route**. Then, once we have filled the
first and last names, we can save it to our backend calling the method
`#save()` in the model.

Since we will be using the same form for adding a new friend and
editing, let's create an
[Ember partial](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_partial)
we can generate the template for the partial with template generator,
`ember g template friends/-form` and add the following content:

{title="app/templates/friends/-form.hbs", lang="handlebars"}
~~~~~~~~
<form {{action "save" on="submit"}}>
  <p>
    <label>First Name:
      {{input value=model.firstName}}
    </label>
  </p>
  <p>
    <label>Last Name:
      {{input value=model.lastName }}
    </label>
  </p>
  <p>
    <label>Email:
      {{input value=model.email}}
    </label>
  </p>
  <p>
    <label>Twitter:
      {{input value=model.twitter}}
    </label>
  </p>
  <input type="submit" value="Save"/>
  <button {{action "cancel"}}>Cancel</button>
</form>
~~~~~~~~

T> As we mentioned in conventions, we should always use kebab-case
T> when naming our files. This applies the same way to partials. In ember CLI,
T> they should start with a dash followed by the partial name (**-form.hbs**).

Then we should modify the template **app/templates/friends/new.hbs**
to include the partial:

{title="app/templates/friends/new.hbs", lang="handlebars"}
~~~~~~~~
<h1>Adding New Friend</h1>
{{partial "friends/form"}}
~~~~~~~~

Now if we visit **http://localhost:4200/friends/new**, the form should be displayed.

There are some new concepts in what we just did. Let's talk about them.

### Partials

In **app/templates/friends/new.hbs** we used

{title="Using partials in app/templates/friends/new.hbs", lang="handlebars"}
~~~~~~~~
{{partial "friends/form"}}
~~~~~~~~

The **partial** method is part of the
[Ember.Handlebars.helpers](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_partial)
class. It is used to render other templates in the context of the
current template. In our example, the friend form is a perfect
candidate for a partial since we will be using the same form to
create and edit a new friend.

### {{action}}

The **{{action}}** helper is one of the most useful features in
ember. It allows us to bind an action in the template to an action
in the template's **Controller** or **Route**. By default it is bound to
the click action, but it can be bound to other actions.

The following button will call the action **cancel** when we click it.

~~~~~~~~
<button {{action "cancel"}}>Cancel</button>
~~~~~~~~

And **<form {{action "save" on="submit"}}>** will call the action
**save** when the **onsubmit** event is fired; that is, when we click **Save**.

T> We could have written the save action as part of the submit button,
T> but for demonstration purposes we put it in the form's **on="submit"**
T> event.

If we go to the browser **http://localhost:4200/friends/new**, open the
console, and click **Save** and **Cancel**, we'll see two errors. The first says
**Nothing handled the action 'save'** and the second **Nothing handled the action
'cancel'**.

Ember expects us to define our action handlers inside the property
**actions** in the **controller** or **route**. When the action is called,
ember first looks for a definition in the **controller**. If there is none, it
goes to the **route** and keeps bubbling until **application route**.
If any of the actions returns **false**, then it stops bubbling.

Let's create a controller for the **friends new route** and add the
actions **save** and **cancel**.

To generate the **friends new controller**, we'll run `ember g controller friends/new` and then edit **app/controllers/friends/new.js** to add
the property **actions**.

{title="app/controllers/friends/new.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Controller.extend({
  actions: {
    save() {
      console.log('+- save action in friends new controller');

      return true;
    },
    cancel() {
      console.log('+- cancel action in friends new controller');

      return true;

    }
  }
});
~~~~~~~~

If we go to **http://localhost:4200/friends/new** and click save, we'll see
in the browser's console **"save action controller"**.

Let's check next how returning **true** from the action makes it bubble.
Go to **app/routes/friends/new.js** and add:

{title="app/routes/friends/new.js", lang="JavaScript"}
~~~~~~~~
actions: {
  save() {
    console.log('+-- save action bubbled up to friends new route');

    return true;
  },
  cancel() {
    console.log('+-- cancel action bubbled up to friends new route');

    return true;
  }
}
~~~~~~~~

Add in **app/routes/friends.js**:

{title="app/routes/friends.js", lang="JavaScript"}
~~~~~~~~
actions: {
  save() {
    console.log('+--- save action bubbled up to friends route');

    return true;
  },
  cancel() {
    console.log('+--- cancel action bubbled up to friends route');

    return true;
  }
}
~~~~~~~~

And then create the file **app/routes/application.js**  with:

{title="app/routes/application.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  actions: {
    save() {
      console.log('+---- save action bubbled up to application route');

      return true;
    },
    cancel() {
      console.log('+---- cancel action bubbled up to application route');

      return true;
    }
  }
});
~~~~~~~~

After adding actions in all those routes, if we click **save** or
**cancel** we'll see the action bubbling through every route currently
active.

~~~~~~~~
+- save action in friends new controller
+-- save action bubbled up to friends new route
+--- save action bubbled up to friends route
+---- save action bubbled up to application route
~~~~~~~~

Again, it is bubbling because we are returning true from every child
**actions**. If we want the action to stop bubbling, let's say in the
**friends route**, we just need to return **false** in the actions specified in **app/routes/friends.js** and we'll get:

~~~~~~~~
+- save action in friends new controller
+-- save action bubbled up to friends new route
+--- save action bubbled up to friends route
~~~~~~~~

As we can see, the action didn't bubble up to the **application route**.

Whenever we have trouble understanding how our actions are going to
bubble, we can go to the **ember-inspector**, click Routes, and then
select **Current Route only**:

![Actions Bubbling](images/actions-bubbling.png)

As we can see, the action will bubble in the following order:

    1. controller friends/new
    2. route: friends/new
    3. route: friends
    4. route: application

How is this related to creating a new friend in our API? We'll
discover that after we cover the next helper. On the
**save** action, we'll validate our model, call **.save()**, which
saves it to the API, and finally transition to a route where we can add new articles.

### The input helper

Last we have the [input helper](http://emberjs.com/api/classes/Ember.Templates.helpers.html#method_input). It allows us to automatically bind a
html input field to a property in our model. With the following **{{input
value=firstName}}**, changing the value changes the property **firstName**.

If we add the following before the input buttons in **app/templates/friends/-form.hbs**

{title="app/templates/friends/-form.hbs", lang="handlebars"}
~~~~~~~~
<div>
  <h2>Friend details</h2>
  <p>{{model.firstName}}</p>
  <p>{{model.lastName}}</p>
</div>
~~~~~~~~

And then go to the browser, we'll see that every time we change the
first or last name field, this will change the description in **Friend details**.

We can also use the input helper to render other types of input such as a
[checkbox](http://emberjs.com/api/classes/Ember.Checkbox.html).
To do so, simply specify **type='checkbox'**.

~~~~~~~~
{{input type="checkbox" name=trusted}}
~~~~~~~~

If we click the checkbox, the attribute trusted will be
true. Otherwise, it will be false.

### Save it!

We learned about actions, **{{partial}}**, and **{{input}}**. Now let's
save our friend to the backend.

To do so, we are going to validate the presence of all the required
fields. If they are present, call **.save()** on the model.
Otherwise, we'll see an error message on the form.

First we'll modify **app/templates/friends/-form.hbs** to include a field **{{errorMessage}}**.

{title="app/templates/friends/-form.hbs", lang="handlebars"}
~~~~~~~~
<form {{action "save" on="submit"}}>
  <h2>{{errorMessage}}</h2>
~~~~~~~~

We will see the error every time we try to save a record without first
filling in all the fields.


Then we'll implement a naive validation in
**app/controllers/friends/new.js** by adding a computed property
called **isValid**:

{title="app/controllers/friends/new.js", lang="JavaScript"}
~~~~~~~~
export default Ember.Controller.extend({
  isValid: Ember.computed(
    'model.email',
    'model.firstName',
    'model.lastName',
    'model.twitter',
    {
      get() {
        return !Ember.isEmpty(this.get('model.email')) &&
          !Ember.isEmpty(this.get('model.firstName')) &&
          !Ember.isEmpty(this.get('model.lastName')) &&
          !Ember.isEmpty(this.get('model.twitter'));
      }
    }
  ),
  actions: {
   ....
  }
});
~~~~~~~~

**Ember.computed**? That's new! ember allows us to create functions
that will be treated as properties. These are called computed
properties. In our example, **isValid** is a **computed property**
that depends on the properties **model.email**, **model.firstName**,
**model.lastName**, and **model.twitter**.  When any of those
properties changes, the function that we passed-in is called and the
value of our property is updated with the returned value.

In our example, we are manually checking that all the fields are not
empty by using the
[isEmpty](http://emberjs.com/api/classes/Ember.html#method_isEmpty)
helper.

With our naive validation in place, we can now modify our save and
cancel actions:

{title="actions in app/controllers/friends/new.js", lang="JavaScript"}
~~~~~~~~
actions: {
  save() {
    if (this.get('isValid')) {
      this.get('model').save().then((friend) => {
        this.transitionToRoute('friends.show', friend);
      });
    } else {
      this.set('errorMessage', 'You have to fill all the fields');
    }

    return false;
  },
  cancel() {
    this.transitionToRoute('friends');

    return false;
  }
}
~~~~~~~~

I>Notice how we are using the ES6 arrow `=>` in the `then`.  It is a new
I>feature which allows us to create functions in a shorter way, it keeps
I>the same context as the scope where it is defined. For more info about
I>check the documentation on the Babel website:
I>[Arrows](https://babeljs.io/docs/learn-es2015/#arrows)

When the action **save** is called, we are first checking if
**isValid** is true. If it is, then we get the model and call
**.save()**. The return of **save()** is a promise, which allow us to
write asynchronous code in a sync manner. The function **.then**
receives a function that will be called when the model has been saved
successfully to the server. When this happens, it returns an instance
of our friend and then we can transition to the route **friends show**
to see our friend's profile.


If we click save and have filled all the required fields, we'll still
get an error: `The route friends/show was not found`. This is because
we haven't defined a **friends show route**. We'll do that in the next
chapter.

T> For a better understanding of promises, I recommend the following talks from Ember NYC called [The Promise Land](https://www.youtube.com/watch?v=mZHO1ZTsoFk#t=2439).

Whenever we want to access a property of an ember object, we need to
use **this.get('propertyName')**. It's almost the same as doing
**object.propertyName**, but it adds extra features like handling
computed properties. If we want to change the property of an object, we
use **this.set('propertyName', 'newvalue')**. Again, it's almost
equivalent to doing **this.propertyName = 'newValue'**, but it adds
support so the observers and computed properties that depend on the
property are updated accordingly.


## Viewing a friend profile

Let's start by creating a **friends show route**

~~~~~~~~
$ ember g route friends/show --path=:friend_id
installing route
  create app/routes/friends/show.js
  create app/templates/friends/show.hbs
updating router
  add route friends/show
installing route-test
  create tests/unit/routes/friends/show-test.js
~~~~~~~~

I> ##Route Generator
I>When creating a new route we can define a custom path for the route
I>with the option `--path`.  We can see the options for every generator
I>with `ember generate route --help`

If we open **app/router.js**, we'll see the route **show** nested
under **friends**.

{title="app/router.js", lang="JavaScript"}
~~~~~~~~
  this.route('friends', function() {
    this.route('new');

    this.route('show', {
      path: ':friend_id'
    });
  });
~~~~~~~~

We have talked previously about **path** but not about dynamic segments.
**path: ':friend_id'** is specifying a dynamic segment.
This means that our route will start with **/friends/**
followed by an id that will be something like **/friends/12** or **/friends/ned-stark**.
Whatever we pass to the URL, it will be available on the model hook under
**params**, so we can reference it like **params.friend_id**. This will
help us to load a specific friend by visiting the URL
**/friends/:friend_id**. A route can have any number of dynamic
segments (e.g., **path: '/friends/:group_id/:friend_id'**.)

Now that we have a **friends show route**, let's start first by editing
the template in **app/templates/friends/show.hbs**:

{title="app/templates/friends/show.hbs", lang="handlebars"}
~~~~~~~~
<ul>
  <li>First Name: {{model.firstName}}</li>
  <li>Last Name: {{model.lastName}}</li>
  <li>Email: {{model.email}}</li>
  <li>twitter: {{model.twitter}}</li>
</ul>
~~~~~~~~

According to what we have covered, the next logical step would be to
add a model hook on the **friends show route** by calling
**this.store.findRecord('friend', params.friend_id)**. However, if we
go to http://localhost:4200/friends/new and add a new friend, we'll be
redirected to the **friends show route** and our friend will be loaded
without requiring us to write a model hook.

Why? As we have said previously, ember is based on convention over
configuration. The pattern of having dynamic segments like
**model_name_id** is so common that if the dynamic segment ends with
**_id**, then the model hook is generated automatically and it calls
**this.store('model_name', params.model_name_id)**.


### Visiting a friend profile

We can navigate to http://localhost:4200/friends to see all of our
friends, but we don't have a way to navigate to their profiles!

Fear not. ember has a helper for that as well, and it is called  **{{link-to}}**.

Let's rewrite the content on **app/templates/friends/index.hbs** to
use the helper:

{title="app/templates/friends/index.hbs", lang="handlebars"}
~~~~~~~~
{{#each model as |friend|}}
  <li>
    {{#link-to 'friends.show' friend}}
      {{friend.firstName}} {{friend.lastName}}
    {{/link-to}}
  </li>
{{/each}}
~~~~~~~~

When we pass our intended route and an instance of a friend to
**link-to**, it maps the property **id** to the parameter
**friend_id**(we could also pass **friend.id**). Then, inside the
block, we render the content of our link tag, which would be the first
and last name of our friend.

One important item to mention is that if we pass an
instance of a friend to **link-to**, then the model hook in the
**friends show route** won't be called. If we want the hook to be
called, instead of doing `{{#link-to 'friends.show' friend}}`, we'll
have to do `{{#link-to 'friends.show' friend.id}}`.

I>Check this example in JS BIN  http://emberjs.jsbin.com/bupay/2/
I>that shows the behavior of **link-to** with an object and with an id.

The resulting HTML will look like the following

{title="Output for link-to helper", lang="HTML"}
~~~~~~~~
<a id="ember476" class="ember-view" href="/friends/1">
 Jon Snow
</a>
~~~~~~~~

If our friend model had a property called **fullName**, we could have
written the helper with this other form of `link-to` which doesn't
include the block:

{title="Using a computed for the link content", lang="handlebars"}
~~~~~~~~
  {{link-to friend.fullName 'friends.show' friend}}
~~~~~~~~

We already talked about computed properties, so let's add one called
**fullName** to **app/models/friend.js**

{title="app/models/friend.js", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';
import Ember from 'ember';

export default DS.Model.extend({
  firstName: DS.attr('string'),
  lastName: DS.attr('string'),
  email: DS.attr('string'),
  twitter: DS.attr('string'),
  totalArticles: DS.attr('number'),
  fullName: Ember.computed('firstName', 'lastName', {
    get() {
      return this.get('firstName') + ' ' + this.get('lastName');
    }
  })
});
~~~~~~~~

The computed property depends on **firstName** and **lastName**. Any
time either of those properties changes, so will the value of
**fullName**.

Once we have the computed property, we can rewrite **link-to** without
the block as follows:

{title="Using friend.fullName in app/templates/friends/index.hbs", lang="handlebars"}
~~~~~~~~
{{link-to friend.fullName 'friends.show' friend}}
~~~~~~~~

Now we'll be able to visit any of our friends! Next, let's add support
to edit a friend.

### Quick Task

1. Add a link so we can move back and forth between a friend's profile and the friends index.
2. Add a link so we can move from **app/templates/index.hbs** to the list of friends (might need to generate the missing template).


## Updating a friend profile

By now it should be clear what we need to update a friend:

1. Create a route with the **ember generator**.
2. Fix path in routes.
3. Update the template.
4. Add Controller and actions.

To create the **friends edit route** we should run:

~~~~~~~~
$ ember g route friends/edit --path=:friend_id/edit
installing route
  create app/routes/friends/edit.js
  create app/templates/friends/edit.hbs
updating router
  add route friends/edit
installing route-test
  create tests/unit/routes/friends/edit-test.js
~~~~~~~~

The nested route **edit** should looks as follows under the the
resource **friends**:

{title="app/router.js", lang="JavaScript"}
~~~~~~~~
  this.route('friends', function() {
    this.route('new');

    this.route('show', {
      path: ':friend_id'
    });

    this.route('edit', {
      path: ':friend_id/edit'
    });
  })
~~~~~~~~

T> Since the route's path follows the pattern **model_name_id**, we
T> don't need to specify a model hook.

Then we should modify the template **app/templates/friends/edit.hbs**
to render the friend's form:

{title="app/templates/friends/edit.hbs", lang="handlebars"}
~~~~~~~~
<h1>Editing {{model.fullName}}</h1>
{{partial "friends/form"}}
~~~~~~~~

With that in place, let's go to a friend's profile and then append
**/edit** in the browser (e.g., http://localhost:4200/friends/2/edit.)

![Friends Edit](images/friends-edit.png)

Thanks to the partial, we have the same form as in the **new template**
without writing anything extra. If we open the browser's console and
click on **Save** and **Cancel**, we'll see that nothing is handling those
actions in the **Friend Edit Controller** and that they are bubbling up
the hierarchy chain.

Let's now implement those actions. The **save** action will behave
exactly as the one in **new**. We'll do the validations and then, when it
has saved successfully, redirect to the profile page. **cancel** will be
different; instead of redirecting to the **friends index route**, we'll
redirect back to the profile page.

We'll create the controller using **ember g controller**.

~~~~~~~~
$ ember g controller friends/edit
installing controller
  create app/controllers/friends/edit.js
installing controller-test
  create tests/unit/controllers/friends/edit-test.js
~~~~~~~~

Then we can write the same computed property to check whether the
object is valid, as well as to check the save and cancel actions.

Write the following in **app/controllers/friends/edit.js**:

{title="app/controllers/friends/edit.js", lang="handlebars"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Controller.extend({
  isValid: Ember.computed(
    'model.email',
    'model.firstName',
    'model.lastName',
    'model.twitter',
    function() {
      return !Ember.isEmpty(this.get('model.email')) &&
        !Ember.isEmpty(this.get('model.firstName')) &&
        !Ember.isEmpty(this.get('model.lastName')) &&
        !Ember.isEmpty(this.get('model.twitter'));
    }
  ),
  actions: {
    save() {
      if (this.get('isValid')) {
        var _this = this;
        this.get('model').save().then(function(friend) {
          _this.transitionToRoute('friends.show', friend);
        });
      } else {
        this.set('errorMessage', 'You have to fill all the fields');
      }
      return false;
    },
    cancel() {
      this.transitionToRoute('friends.show', this.get('model'));
      return false;
    }
  }
});
~~~~~~~~

If we refresh our browser, edit the profile, and click save, we'll
see our changes applied successfully! We can also check that clicking
**cancel** takes us back to the user's profile.

To transition from a controller, we have been using
**this.transitionToRoute**. It's a helper that behaves similarly to
the **{{link-to}}** helper but from within a controller. If we were in
a **Route**, we could have used **this.transitionTo**.

### Refactoring

Both our **friends new controller** and **friends edit controller**
share pretty much the same implementation. Let's refactor that
creating a base class from which both will inherit.

The only thing that will be different is the **cancel** action. Let's
create our base class and then override in every controller according
to our needs.

Create a base controller:

~~~~~~~~
$ ember g controller friends/base
installing controller
  create app/controllers/friends/base.js
installing controller-test
  create tests/unit/controllers/friends/base-test.js
~~~~~~~~

And put the following content in it


{title="app/controllers/friends/base.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Controller.extend({
  isValid: Ember.computed(
    'model.email',
    'model.firstName',
    'model.lastName',
    'model.twitter',
    {
      get() {
        return !Ember.isEmpty(this.get('model.email')) &&
          !Ember.isEmpty(this.get('model.firstName')) &&
          !Ember.isEmpty(this.get('model.lastName')) &&
          !Ember.isEmpty(this.get('model.twitter'));
      }
    }
  ),
  actions: {
    save() {
      if (this.get('isValid')) {
        this.get('model').save().then((friend) => {
          this.transitionToRoute('friends.show', friend);
        });
      } else {
        this.set('errorMessage', 'You have to fill all the fields');
      }

      return false;
    },
    cancel() {
      return true;
    }
  }
});
~~~~~~~~

We left **isValid** and **save** exactly as they were, but we have no
implementation in the **cancel** action (we just let it bubble up).


We can now replace **app/controllers/friends/new.js** to inherit from **base**
and override the cancel action:

{title="app/controllers/friends/new.js", lang="JavaScript"}
~~~~~~~~
import FriendsBaseController from './base';

export default FriendsBaseController.extend({
  actions: {
    cancel() {
      this.transitionToRoute('friends.index');
      return false;
    }
  }
});
~~~~~~~~

And **app/controllers/friends/edit.js** with:

{title="app/controllers/friends/edit.js", lang="JavaScript"}
~~~~~~~~
import FriendsBaseController from './base';

export default FriendsBaseController.extend({
  actions: {
    cancel() {
      this.transitionToRoute('friends.show', this.get('model'));

      return false;
    }
  }
});
~~~~~~~~

If we don't override the action, ember will use the one specified in
the base class.

### Visiting the edit page.

We can edit a friend now, but we need a way to reach the **edit**
screen from the **user profile page**. To do that, we should add a
**{{link-to}}** in our **app/templates/friends/show.hbs**.

{title="app/templates/friends/show.hbs", lang="handlebars"}
~~~~~~~~
<ul>
  <li>First Name: {{model.firstName}}</li>
  <li>Last Name: {{model.lastName}}</li>
  <li>Email: {{model.email}}</li>
  <li>twitter: {{model.twitter}}</li>
  <li>{{link-to "Edit info" "friends.edit" model}}</li>
</ul>
~~~~~~~~

If we go to a friend's profile and click **Edit info**, we'll be taken
to the edit screen page.

## Deleting friends

We have decided not to lend anything to a couple of friends ever again
after they took our beloved **The Dark Side of the Moon** vinyl and
returned it with scratches.

It's time to add support to delete some friends from our application.
We want to be able to delete them directly within their profile
page or when looking at the index.

By now it should be clear how we will do this. Let's use
actions.

Our destroy actions will call
[model#destroyRecord()](http://emberjs.com/api/data/classes/DS.Model.html#method_destroyRecord)
and then **this.transitionTo** to the **friends index route**.

Let's replace our **app/templates/friends/index.hbs** so it includes the
delete action:

{title="app/templates/friends/index.hbs", lang="handlebars"}
~~~~~~~~
<h1>Friends Index</h1>

<h2>Friends: {{model.length}}</h2>

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    {{#each model as |friend|}}
      <tr>
        <td>{{link-to friend.fullName "friends.show" friend}}</td>
        <td><a href="#" {{action "delete" friend}}>Delete</a></td>
      </tr>
    {{/each}}
  </tbody>
</table>
~~~~~~~~

And then add the action **delete**. This time let's put
the delete action on the route **app/routes/friends/index.js**:

{title="app/routes/friends/index.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.store.findAll('friend');
  },
  actions: {
    delete(friend) {
      friend.destroyRecord();
      return false;
    }
  }
});
~~~~~~~~

To support deletion on **friends show route**, we just need to add
the same link with the action delete and implement the action. Again,
we'll put it in the route's actions. In this case, **app/routes/friends/show.js**:

{title="app/routes/friends/show.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  actions: {
    delete(friend) {
      friend.destroyRecord().then(() => {
        this.transitionTo('friends.index');
      });
    }
  }
});
~~~~~~~~

With that we can now create, update, edit, and delete any of our
friends!


### Refactoring Time

If we check what we just did, we'll notice that both delete actions
are identical except that the one in the index doesn't need to
transition since it is already there.

For this specific scenario, calling
**this.transitionTo('friends.index')** from within the **friends index route**
will behave like a no-op. This is important to mention because we
could have one single implementation for the delete action and access
it via event bubbling.

We can put the delete action in **app/routes/friends.js**, which is the
parent route for both **friends index route** and **friends new route**:


{title="app/routes/friends.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  actions: {
    save() {
      console.log('+--- save action bubbled up to friends route');

      return true;
    },
    cancel() {
      console.log('+--- cancel action bubbled up to friends route');

      return true;
    },
    delete: function(friend) {
      friend.destroyRecord().then(() => {
        this.transitionTo('friends.index');
      });
    }
  }
});
~~~~~~~~

And delete both actions from **app/routes/friends/index.js** and **app/routes/friends/show.js**.

{title="app/routes/friends/index.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.store.findAll('friend');
  }
});
~~~~~~~~~

{title="app/routes/friends/show.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({});
~~~~~~~~

Let's breathe slowly and take a moment to enjoy that fresh feeling of
deleting repeated code...

Done?

Next, let's add some styling to our project. We don't want to show
this to our friends as it is right now.

## Mockups

Before changing our templates, we'll review a couple of mockups to
have an idea of how our pages are going to look.

### Friends Index

![Friends Index](images/friend-index-mockup.png)

We'll have a header that will take us to a dashboard, the friends
index page, and about page. Additionally, we can insert some content
depending on which route we are visiting. In the **Friends Index
Route** we'll see a search box to filter users.

Then we'll have a table that can be ordered alphabetically or by
number of items.

### friend profile

![friend profile](images/friend-profile-mockup.png)

Our friend profile will show us the user's data with an avatar that we
might pull from Gravatar.

We have links to add new articles, edit the user's info,
or delete the user's profile.

At the bottom we'll have the list of all the articles the user has
borrowed with options to mark them as returned or to send a
reminder.

If we are careful, we'll also notice that the URL looks a little different
from what we currently have. After the friend **id**, we see
**/articles** (**..com/friends/1/articles**). Whenever we visit the user
profile, the nested resource articles will be rendered by default. We
haven't talked about it yet, but basically we are rendering a resource
under our **friends show route** that will defer all responsibility
of managing state, handling actions, etc. to a different **Controller**
and **Route**.


### Dashboard

![Dashboard](images/dashboard-mockup.png)

The third mockup is a dashboard where we can ask questions like, "how many
articles have we lent to our friends" and "who's the friend with the most
articles?" We can also see the number of articles borrowed per day.

## Installing Dependencies

To save time, we'll be using [picnicss](http://picnicss.com) as our
base CSS and **fontello** for icons.

### Including picnicss

Since picnicss is a front-end dependency, we can use **Bower** to
manage such a dependency for us.

Let's install picnic running the following command:

{title="Adding picnic to bower.json", lang="bash"}
~~~~~~~~
bower install picnic --save
~~~~~~~~

Once it is finished, we'll find the picnic assets under
**bower_components/picnic/**.

The fact that they are there doesn't mean that they'll be included in
our assets. We still need to tell **ember CLI** that we want to
**import** those assets into our application. To do so, we need to add
the following line to our ember-cli-build.js before `return app.toTree();`

{title="Adding picnic to the ember-cli-build.js"}
~~~~~~~~
/* global require, module */
var EmberApp = require('ember-cli/lib/broccoli/ember-app');

module.exports = function(defaults) {
  var app = new EmberApp(defaults, {
  });

  app.import('bower_components/picnic/releases/plugins.min.css');
  app.import('bower_components/picnic/releases/picnic.min.css');

  return app.toTree();
};
~~~~~~~~

**app.import** is a helper function that tells **ember CLI** to append
**bower_components/picnic/releases/picninc.min.css** into our assets
(we also import `plugins.min.css` since it is required by picnic).
**By default it will put any **CSS** file we import into
****/vendor.css** and any JavaScript file into **/vendor.js**.

If we check **app/index.html**, we'll see 2 CSS files included:

{title="app/index.html", lang="handlebars"}
~~~~~~~~
<link rel="stylesheet" href="assets/vendor.css">
<link rel="stylesheet" href="assets/borrowers.css">
~~~~~~~~

The first one contains all the imported (vendor) **CSS files** and the
second one contains the **CSS files** we defined under **app/styles**.

I>Why do we have two separate CSS and JavaScript files?  Vendor files are
I>less likely to change, so we can take advantage of caching when we
I>deploy our application. While our app CSS and JS might change, vendor
I>files will stay the same, allowing us to take advantage of the cache.

After modifying our `ember-cli-build.js` we need to stop and start the
server again so the changes are applied. Once we have done that, we
can we refresh our browser and go to
**http://localhost:4200/assets/vendor.css**, we'll see that the code
for **picnicss** is there.

### Including fontello

Because [fontello](http://fontello.com/) doesn't have a custom
distribution we can't download it with **bower**, we'll download a
bundle of icons and fonts that we can manage manually by putting it
under **vendor/fontello**.

I>With bower dependencies, we don't have to worry about keeping things
I>under our revision control system because bower will take care of
I>downloading them for us. Howevever, we do have to keep track of
I>dependencies not managed by bower.

We can download a bundle from the following URL
http://cl.ly/3y1W1B3Y4028 and then put the content under **vendor/**,
which will give us the directory **vendor/fontello**.

In order to tell **ember CLI** that we want to include fontello's
CSS and fonts, we need to modify our Brocfile  as follows:

{title="ember-cli-build.js", lang="JavaScript"}
~~~~~~~~
/* global require, module */
var EmberApp = require('ember-cli/lib/broccoli/ember-app');

module.exports = function(defaults) {
  var app = new EmberApp(defaults, {
  });

  app.import('vendor/fontello/fontello.css');
  app.import('vendor/fontello/font/fontello.ttf', {
    destDir: 'font'
  });
  app.import('vendor/fontello/font/fontello.eot', {
    destDir: 'font'
  });
  app.import('vendor/fontello/font/fontello.svg', {
    destDir: 'font'
  });
  app.import('vendor/fontello/font/fontello.woff', {
    destDir: 'font'
  });

  app.import('bower_components/picnic/releases/picnic.min.css');
  app.import('bower_components/picnic/releases/plugins.min.css');

  return app.toTree();
};

~~~~~~~~

We are already familiar with the line to import **fontello.css**, but
the following ones are new to us since we have never passed any option to
**import**.

The option **destDir** tells **ember CLI** that we want to put those
files under a directory called **font**. If we save and refresh our
browser, **vendor.css** should now include **fontello.css**.

T> Check the change on GitHub by visiting the following
T> commit: [Add fontello and picnicss](https://github.com/abuiles/borrowers/commit/90a1ea3fe6320ad1746b4c0ab4069401d2fd6247).

With that, we know the basics of including vendor files. Now that we
have our basic dependencies on hand, let's improve the appearance of
our templates.

### The header

We'll use partials as much as possible to simplify our templates.
In this case, we'll create a partial that contains the code for the
navigation bar. Create the file **app/templates/partials/-header.hbs**
with the following content:


{title="app/templates/partials/-header.hbs", lang="handlebars"}
~~~~~~~~
<nav>
  {{link-to "Borrowers" "index" class="brand"}}

  <!-- responsiveness-->
  <input class="show" id="bmenu" type="checkbox">
  <label class="burger toggle pseudo button" for="bmenu">menu</label>
  <!-- responsiveness-->

  <div class="menu">
    {{link-to "Dashboard" "index" class="pseudo button icon-gauge"}}
    {{link-to "Friends" "friends" class="pseudo button icon-users-1"}}
    {{link-to "New Friend" "friends.new" class="pseudo button icon-user-add"}}
  </div>
</nav>
~~~~~~~~

The header should always be visible in our application. In ember, the
right receptacle for that content would be the application template
since it will contain any other template inside its **{{outlet}}**.


Modify **app/templates/application.hbs** as follows:

{title="app/templates/application.hbs", lang="handlebars"}
~~~~~~~~
{{partial "partials/header"}}

<main>
  {{outlet}}
</main>
~~~~~~~~

We will render the header and wrap the outlet in a row using
**picnicss** classes.

If we refresh, the header should display nicely.

### Friends Index

First, let's remove the **<h1>** from **app/templates/friends.hbs** so
it only contains **{{outlet}}**. Next, clean up
**app/templates/friends/index.hbs** so it adds the class **primary** to
the table:

{title="app/templates/friends/index.hbs", lang="handlebars"}
~~~~~~~~
<table class="primary">
  <thead>
    <tr>
      <th>Name</th>
      <th>Articles</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    {{#each model as |friend|}}
      <tr>
        <td>{{link-to friend.fullName "friends.show" friend}}</td>
        <td>{{friend.totalArticles}}</td>
        <td><a href="#" {{action "delete" friend}}>delete</a></td>
      </tr>
    {{/each}}
  </tbody>
</table>
~~~~~~~~

Then we need to add some extra styling to the table. We want it
to be full width, so let's modify **app/styles/app.css** as follows:

{title="app/styles/app.css", lang="CSS"}
~~~~~~~~
body {
  display: block;
  text-align: center;
  color: #333;
  background: #FFF;
  margin: 80px auto;
  width: 100%;
}

table {
  width: 100%;
}
~~~~~~~~

Now if we visit http://localhost:4200/friends, we should see:

![Friends Index](images/friends-index-picnic1.png)

### New Friend And Friend profile template

Next let's modify **app/templates/friends/-form.hbs**

{title="app/templates/friends/-form.hbs", lang="handlebars"}
~~~~~~~~
<form {{action "save" on="submit"}}>
  <h2>{{errorMessage}}</h2>
  <fieldset>
    {{input value=model.firstName placeholder="First Name"}}<br>
    {{input value=model.lastName  placeholder="Last Name"}}<br>
    {{input value=model.email     placeholder="email"}}<br>
    {{input value=model.twitter   placeholder="twitter"}}<br>
    <input type="submit" value="Save" class="primary">
    <button {{action "cancel"}}>Cancel</button>
  </fieldset>
</form>
~~~~~~~~

And finally, change **app/templates/friends/show.hbs**.

{title="app/templates/friends/show.hbs", lang="handlebars"}
~~~~~~~~
<div class="card friend-profile">
  <p>{{model.firstName}}</p>
  <p>{{model.lastName}}</p>
  <p>{{model.email}}</p>
  <p>{{model.twitter}}</p>
  <p>{{link-to "Edit info" "friends.edit" model}}</p>
  <p><a href="#" {{action "delete" model}}>delete</a></p>
</div>
~~~~~~~~

### The Dashboard

By default, we'll use the **application index route** as the dashboard.
For now, we are going to create the file **app/templates/index.hbs**
and write a simple title:

{title="app/templates/index.hbs", lang="handlebars"}
~~~~~~~~
**<h2>Dashboard</h2>**.
~~~~~~~~

Let's move on with more functionality.


## Articles Resource

With our **Friends** CRUD ready, we can start lending articles.

Let's create an articles resource:

~~~~~~~~
$ ember generate resource articles createdAt:date description:string notes:string state:string
  create app/models/article.js
  create tests/unit/models/article-test.js
  create app/routes/articles.js
  create app/templates/articles.hbs
  create tests/unit/routes/articles-test.js
~~~~~~~~

Let's check the model.

{title="app/models/article.js", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';

export default DS.Model.extend({
  createdAt: DS.attr('date'),
  description: DS.attr('string'),
  notes: DS.attr('string'),
  state: DS.attr('string')
});
~~~~~~~~

We have defined our **Articles** model successfully, but we need to
wire the relationship between **Friends** and **Articles**. Let's do
that next.


## Defining relationships.

We have to specify that a friend can have many articles and that those
articles belong to a friend. In other frameworks this is known as
**hasMany** and **belongsTo** relationships, and so they are in ember
data.

I>Remember, ember doesn't include data handling support by default.
I>This is accomplished through ember data, which is the official library
I>for this.

If we want to add a **hasMany** relationship to our models, we write:

~~~~~~~~
  articles: DS.hasMany('article')
~~~~~~~~

Or we want a **belongsTo**:

~~~~~~~~
  friend: DS.belongsTo('friend')
~~~~~~~~

Using the previous relationship types, we can modify our **Article** model:

{title="app/models/article.js", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';

export default DS.Model.extend({
  createdAt:   DS.attr('date'),
  description: DS.attr('string'),
  notes:       DS.attr('string'),
  state:       DS.attr('string'),
  friend:      DS.belongsTo('friend')
});
~~~~~~~~

And our **Friend** model to add the **hasMany** to articles:

{title="app/models/friend.js", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';
import Ember from 'ember';

export default DS.Model.extend({
  articles:      DS.hasMany('article'),
  email:         DS.attr('string'),
  firstName:     DS.attr('string'),
  lastName:      DS.attr('string'),
  totalArticles: DS.attr('number'),
  twitter:       DS.attr('string'),
  fullName: Ember.computed('firstName', 'lastName', {
    get() {
      return this.get('firstName') + ' ' + this.get('lastName');
    }
  })
});
~~~~~~~~

With just those two lines, we have added a relationship between our
models. Now let's work on the **articles** resource.

I> ## Specifying relationships with the generator.
I> We can add `hasMany` or `belongsTo` relationships when running
I> the generator, we didn't use it when we created the articles
I> resource so we could explain relationships, but we could have done
I> the following: `ember g resource articles friend:belongsTo ...`.

## Nested Articles Index

In our **friend profile** mockup, we specified that we wanted to render
the list of articles as a nested route inside the friend profile.

If we look again at the mockup now highlighting the nested routes,

![Friend profile with nested routes](images/friend-profile-mockup-nested.png)

the part in red corresponds to the **friends show route** while the part
in blue is where all routes belonging to the resource **Articles**
will go.

We need to make a couple of changes to handle this scenario. First we
need to make sure that **articles** is specified as a nested
resource inside **friends show**. Let's go to our **app/router.js**
and change it to reflect this:

{title="app/router.js", lang="JavaScript"}
~~~~~~~~
Router.map(function() {
  this.route('friends', function() {
    this.route('new');

    this.route('show', {
      path: ':friend_id'
    }, function() {
      this.route('articles', {resetNamespace: true}, function() {
      });
    });

    this.route('edit', {
      path: ':friend_id/edit'
    });
  });
});
~~~~~~~~

D> ## What is resetNamespace?
D> When nesting routes, ember by default combines the parent routes to
D> form the final route name. In the example above if we had excluded
D> `resetNamespace: true` then the final name for the articles routes
D> would have been `friends/show/articles` instead of `articles`. Also
D> the resolver would have expected us to defined our route files inside
D> the `friends show route` directory instead of the top level
D> `articles`. This is a common pattern to use when working with nested
D> resources.


Now let's open the **ember-inspector** and check our newly defined routes:

![Nested Articles Routes](images/articles-routes.png)

We can identify the routes and controllers that ember expects us to
define for the new resource.

Next we need to add an **{{outlet }}** to
**app/templates/friends/show.hbs**, which is where the nested routes will render:

{title="app/templates/friends/show.hbs", lang="handlebars"}
~~~~~~~~
<div class="card friend-profile">
  <p>{{model.firstName}}</p>
  <p>{{model.lastName}}</p>
  <p>{{model.email}}</p>
  <p>{{model.twitter}}</p>
  <p>{{link-to "Edit info" "friends.edit"  model}}</p>
  <p><a href="#" {{action "delete" model}}>delete</a></p>
</div>
<div class="articles-container">
  {{outlet}}
</div>
~~~~~~~~

Any nested route will be rendered by default into its parent's
**{{outlet}}**.

### Rendering the index.

Let's create a new file called **app/templates/articles/index.hbs**
and write the following:

{title="app/templates/articles/index.hbs", lang="handlebars"}
~~~~~~~~
<h2>Articles Index</h2>
~~~~~~~~

If we visit a friend profile, we won't see anything related with
the **articles index route**. Why? Well, we are not visiting that
route, that's why. To get to the **articles index route**, we need to
modify the **link-to** in **app/templates/friends/index.hbs** to reference
the route **articles** instead of **friends.show**. We'll still pass
the **friend** as an argument since the route **articles** is nested
under **friends.show** and it has the dynamic segment **:friend_id**.

{title="app/templates/friends/index.hbs", lang="handlebars"}
~~~~~~~~
<td>{{link-to friend.fullName "articles" friend}}</td>
~~~~~~~~

Now, with the previous change, if we go to the friends index and visit
any profile, we'll see **Articles Index** at the bottom.

Opening the **ember-inspector** and filtering by *Current Route
only**, we'll see:

![articles index route](images/articles-active-route.png)

Routes are resolved from top to bottom, so when we navigate to
**/friends/1/articles** it will go first to the **application route**
and move to **friends show route** to fetch our friend. Once it is
loaded, it will move to **articles index route**.

Next we need to define the model hook for the **articles index route**.

### Fetching our friend articles.

Let's add the **articles index route** to the generator and reply
'no' when it asks us if we want to overwrite the template.

{title="", lang="bash"}
~~~~~~~~
$ ember g route articles/index
installing route
[?] Overwrite app/templates/articles/index.hbs? (Yndh) n

[?] Overwrite app/templates/articles/index.hbs? No, skip
  create app/routes/articles/index.js
  skip app/templates/articles/index.hbs
updating router
  add route articles/index
installing route-test
  create tests/unit/routes/articles/index-test.js
~~~~~~~~

In **app/routes/articles/index.js**, load the data using the model
hook:

{title="app/routes/articles/index.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.modelFor('friends/show').get('articles');
  }
});
~~~~~~~~

In the model hook, we are using a new function
[this.modelFor](http://emberjs.com/api/classes/Ember.Route.html#method_modelFor)
that helps us grab the model for any parent route. In this scenario,
parent routes are all the ones appearing on top of **articles index
route** in the **ember-inspector**.

Once we get the model for **friends show route**, we simply ask for
its articles. And that's what we are returning.

We need to modify the **app/templates/articles/index.hbs** so it
displays the articles:

{title="app/templates/articles/index.hbs", lang="handlebars"}
~~~~~~~~
<table class="primary">
  <thead>
    <tr>
      <th>Description</th>
      <th>Borrowed since</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    {{#each model as |article|}}
      <tr>
        <td>{{article.description}}</td>
        <td>{{article.createdAt}}</td>
        <td></td>
        <td></td>
      </tr>
    {{/each}}
  </tbody>
</table>
~~~~~~~~

If our friend doesn't have articles yet, we can use the
**ember-inspector** to add some manually.

Let's open the **ember-inspector** and select the model from the route
**friends show**:

![Select Friend Model](images/select-friend.png)

Once we have the instance of a friend assigned to the variable **$E**,
let's run the following on the browser's console:

~~~~~~~~
$E.get('articles').createRecord({description: 'foo'})
$E.get('articles').createRecord({description: 'bar'})
~~~~~~~~

We will notice that our friend index updates automatically with the
created records.

So far we are only putting records into the store, but they are not
being saved to the backend. To do that we'll need to call **save()**
on every instance. Let's try to call save:

~~~~~~~~
$E.get('articles').createRecord({description: 'foo'}).save()
~~~~~~~~

We will notice that a **POST** is attempted to our backend,
but it gets rejected because the model is not valid:

~~~~~~~~
Error: The backend rejected the commit because it was invalid: {state: can't be blank,is not included in the list}
~~~~~~~~

Let's add the route **articles new** and the template so we can lend
new articles to our friends.


T> Check the following commit to review all the changes of the
previous chapter: [Add articles index](https://github.com/abuiles/borrowers/commit/f9e5d27)

### Sideloading Articles

If we visit
[http://api.ember-cli-101.com/api/friends](http://api.ember-cli-101.com/api/friends),
we'll notice that there is no information about any of our friends'
articles. We omit that information intentionally so the early
version of the application won't break.

However, from now on, we need to include the articles so that they are
displayed when we visit a friend's profile. To accomplish this we'll
use version 2 (V2) of the borrowers backend API, which includes the
articles for every user.  We can try it out by visiting
[http://api.ember-cli-101.com/api/v2/friends](http://api.ember-cli-101.com/api/v2/friends).

How do we use the new version of the API? We need to modify
the property **namespace** in the application adapter so it refers to
**api/v2**. Let's change  **app/adapters/application.js** to look like
the following:

{title="app/adapters/application.js", lang="JavaScript"}
~~~~~~~~
import ActiveModelAdapter from 'active-model-adapter';

export default ActiveModelAdapter.extend({
  namespace: 'api/v2'
});
~~~~~~~~

Once we have made that change, we'll consume the new version of
the API.

I> Sideloading data is one of the different strategies we have in
I> ember data to work with relationships. We'll explore other alternatives
I> in a later chapter dedicated to ember data.

## Lending new articles

Let's start by adding the route. We've done it
with the generator up to this point, but now we'll do it manually.

We need to add the nested route **new** under the resource **articles**:

{title="app/router.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';
import config from './config/environment';

var Router = Ember.Router.extend({
  location: config.locationType
});

Router.map(function() {
  this.route('friends', function() {
    this.route('new');

    this.route('show', {
      path: ':friend_id'
    }, function() {
      this.route('articles', {resetNamespace: true}, function() {
        this.route('new');
      });
    });

    this.route('edit', {
      path: ':friend_id/edit'
    });
  });
});

export default Router;
~~~~~~~~

Then let's create the route **app/routes/articles/new.js** with the
model hook and actions:

{title="app/routes/articles/new.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.store.createRecord('article', {
      state: 'borrowed',
      friend: this.modelFor('friends/show')
    });
  },
  actions: {
    save() {
      var model = this.modelFor('articles/new');

      model.save().then(() => {
        this.transitionTo('articles');
      });
    },
    cancel() {
      this.transitionTo('articles');
    }
  }
});
~~~~~~~~

In the model hook we use
[this.store.createRecord](http://emberjs.com/api/data/classes/DS.Store.html#method_createRecord),
which creates a new instance of a model in the store. It takes the
name of the model we're creating and its properties.

We pass the property **friend** and **state**. The former will
make sure that the article is linked with our friend, and the latter is
simply setting the state attribute. We'll start it in **borrowed**.

ember data allows us to specify a **defaultValue** for our attributes.
We can use that instead of doing it explicitly in the model hook. In
**app/models/article.js**, let's replace the definition of **state** so
it looks as follows:

{title="app/models/article.js", lang="JavaScript"}
~~~~~~~~
  state: DS.attr('string', {
    defaultValue: 'borrowed'
  })
~~~~~~~~

Then we can modify our model in **app/routes/articles/new.js** so it
doesn't add the initial state:

{title="app/routes/articles/new.js", lang="JavaScript"}
~~~~~~~~
  model() {
    return this.store.createRecord('article', {
      friend: this.modelFor('friends/show')
    });
  },
~~~~~~~~

In our friends example we put the **save** and **cancel** actions in
the controller, but this time we are defining it in the route. The
question is: where do we need to put this kind of action?

We used both strategies as an example that we can get to the same
results using either the route or controller. However, the rule of thumb is
that we keep every action that modifies our application state in the
routes and use the controllers as decorators for our templates.
Actions like saving, destroying, and creating new objects are best fit for
the route.

I> ## Common patterns on resource routes model hooks
I> - Edit and Show Route:  **return this.store.findRecord('modelName', modelId)**
I> - Create Route: **return this.store.createRecord('modelName', properties)**
I> - Index Route: **return this.store.findAll('modelName')**


Next we need to add the **new** template. Since we might want to reuse
the **form**, let's add it in a partial and then include it in the
template **app/templates/articles/new.hbs**.

We'll create the **-form** partial in
**app/templates/articles/-form.hbs**. Remember, partial names begin
with a dash:

{title="app/templates/articles/-form.hbs", lang="handlebars"}
~~~~~~~~
<form>
  <h2>{{errorMessage}}</h2>
  <fieldset>
    {{input value=model.description placeholder="Description"}}<br>
    {{input value=model.notes  placeholder="Notes"}}<br>
    <button {{action "save"}} class="primary">Save</button>
    <button {{action "cancel"}}>Cancel</button>
  </fieldset>
</form>
~~~~~~~~

Then include it in **app/templates/articles/new.hbs**:

{title="app/templates/articles/new.hbs", lang="handlebars"}
~~~~~~~~
<h2> Lending new articles</h2>
{{partial "articles/form"}}
~~~~~~~~

We are almost done. We have set up the route and template, but we still
haven't added a link to navigate to the **articles new route**. Let's
add **link-to** to **articles.new** in **app/templates/friends/show.hbs**:

{title="app/templates/friends/show.hbs", lang="handlebars"}
~~~~~~~~
<div class="card friend-profile">
  <p>{{model.firstName}}</p>
  <p>{{model.lastName}}</p>
  <p>{{model.email}}</p>
  <p>{{model.twitter}}</p>
  <p>{{link-to "Lend article" "articles.new"}}</p>
  <p>{{link-to "Edit info" "friends.edit"  model}}</p>
  <p><a href="#" {{action "delete" model}}>delete</a></p>
</div>
<div class="articles-container">
  {{outlet}}
</div>
~~~~~~~~

We are creating the link with `{{link-to "Lend articles"
"articles.new"}}`. Since we're already in the context of a
friend, we don't need to specify the dynamic segment. If we want to add
the same link in the **friends index route**, we'll need to
pass the parameter as **{{link-to "Lend articles" "articles.new"
friend}}** where **friend** is an instance of a **friend**.

X> ## Tasks
X>
X> Create an **articles new controller** and validate that the
X> model includes **description**. If it is valid, let
X> the action bubble to the route. Otherwise, set an **errorMessage**.
X>

I> Click the following link for a list of changes introduced in this
I> chapter: [Lending articles](https://github.com/abuiles/borrowers/commit/262f8c1).

{pagebreak}

## Computed Property Macros

In **app/controllers/friends/base.js**, we define the computed property
**isValid** with the following code:

{title="Computed Property isValid is app/controllers/friends/base.js", lang="JavaScript"}
~~~~~~~~
  isValid: Ember.computed(
    'model.email',
    'model.firstName',
    'model.lastName',
    'model.twitter',
    {
      get() {
        return !Ember.isEmpty(this.get('model.email')) &&
          !Ember.isEmpty(this.get('model.firstName')) &&
          !Ember.isEmpty(this.get('model.lastName')) &&
          !Ember.isEmpty(this.get('model.twitter'));
      }
    }
  ),
~~~~~~~~

Although the previous code does what we expect, it is not the most
pleasant to read, especially with all those nested **&&'s**. As it
turns out, Ember has a set of helper functions that will allow us to
write the previous code in a more idiomatic way using something called
computed property macros.

Computed property macros are a set of functions living under
**Ember.computed.** that allow us to create computed properties in an
easier, more readable and clean way.


As an example, let's take two computed property macros and write our
**isValid** on terms of them:

- [Ember.computed.and](http://emberjs.com/api/classes/Ember.computed.html#method_and)
- [Ember.computed.notEmpty](http://emberjs.com/api/classes/Ember.computed.html#method_notEmpty)


{title="Computed Property With Macros in app/controllers/friends/base.js", lang="JavaScript"}
~~~~~~~~
export default Ember.Controller.extend({
  hasEmail:     Ember.computed.notEmpty('model.email'),
  hasFirstName: Ember.computed.notEmpty('model.firstName'),
  hasLastName:  Ember.computed.notEmpty('model.lastName'),
  hasTwitter:   Ember.computed.notEmpty('model.twitter'),
  isValid:      Ember.computed.and(
    'hasEmail',
    'hasFirstName',
    'hasLastName',
    'hasTwitter'
  ),

// actions omitted
~~~~~~~~

This is certainly much cleaner and less error-prone than original
implementation.

We can see the full list of computed properties under the
[Ember.computed
namespace](http://emberjs.com/api/classes/Ember.computed.html).


## Using components to mark an article as returned.

We previously lent our favorite Whisky glass to one of our friends and
they just returned it. We need to mark the item as returned.

Our interface will look similar to the following. We can select the
state of the article within the articles index. Whenever that article
has pending changes, we'll see a **save** button.


![Articles Index with Selector](images/articles-return.jpg)

Using components we'll encapsulate the behavior per row into its own
class, removing responsibility from the model and delegating it to a
class. This class will handle how every row should look and
additionally when it should fire a save depending on the state of
every article.

We'll create an **articles/article-row** component which will wrap
every element.  We'll pass the necessary data to render the list of
possible states and also the article.

Let's create the **articles/article-row** using the components
generator.

{title="Creating an component", lang="bash"}
~~~~~~~~
$ ember g component articles/article-row
installing component
  create app/components/articles/article-row.js
  create app/templates/components/articles/article-row.hbs
installing component-test
  create tests/integration/components/articles/article-row-test.js
~~~~~~~~

Let's modify the component so it looks as follows:

{title="app/components/articles/article-row.js", lang="JavaScript"}
~~~~~~~~
import  Ember from 'ember';

export default Ember.Component.extend({
  tagName: 'tr',
  article: null, // passed-in
  articleStates: null,   // passed-in
  actions: {
    saveArticle(article) {
      this.sendAction('save', article);
    }
  }
});
~~~~~~~~

We are specifying that the html tag for this component is going to be a
`tr` meaning that whatever content we put in the template, it will be
wrapped in table row using the HTML tag **tr**, by default it is a
**div**. Also we defined two properties `articles` and `articleStates`
with value `null` and the comment: "passed-in". It will help people
consuming the component to identify which data they should pass-in.

We also added an action "saveArticle" which receives an article and
then calls `this.sendAction` with the arguments `save` and the
`article`.

Unlike controllers, actions in components won't bubble up
automatically, so if we want to call an action in a Route or
Controller from a component we'll need to bind such action to a
property and then call it using `sendAction`, we'll see shortly how
that look in a template.

We need to add the the markup for the component as follows:

{title="app/templates/components/articles/article-row.hbs", lang="handlebars"}
~~~~~~~~
<td>{{article.description}}</td>
<td>{{article.notes}}</td>
<td>{{article.createdAt}}</td>
<td>
  {{#x-select value=article.state}}
    {{#each articleStates as |state|}}
      {{#x-option value=state}}{{state}}{{/x-option}}
    {{/each}}
  {{/x-select}}
</td>
<td>
  {{#if article.isSaving}}
    <p>Saving ...</p>
  {{else if article.hasDirtyAttributes}}
    <button {{action "saveArticle" article}}>Save</button>
  {{/if}}
</td>
~~~~~~~~

In the template we are defining the cells for every article row and
reading the value from the "passed-in" property article, notice also
that we are calling the action "saveArticle" not "save".

We are also using the
[emberx-select](https://github.com/thefrontside/emberx-select) addon,
which includes a select component based on the native html select
element, in the `value` attribute we are binding the selection to the
article's state and then for every possible state we are rendering an
option element.

Before continuing we need to install the addon and then restart the
server:

```
$ ember install emberx-select
$ ember server --proxy http://api.ember-cli-101.com
```

We are also using the properties **article.isSaving** and
**article.hasDirtyAttributes**, which belong to the article we passed-in.

The previous properties are part of
[DS.Model](http://emberjs.com/api/data/classes/DS.Model.html) and they
help us to know things about a model. In the previous scenario,
**article.hasDirtyAttributes** becomes true if there is a change to the model and
**article.isSaving** is true if the model tries to persist any changes to
the backend.

Before using our components, let's add to our articles index
controller a property called `possibleStates` which we'll use to pass
down to the component:

{title="app/controllers/articles/index.js", lang="handlebars"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Controller.extend({
  possibleStates: ["borrowed", "returned"]
});
~~~~~~~~

I> We can create the controller with the controller generator:
I> `ember g controller articles/index`.

Now let's use our component in the articles index template:

{title="app/templates/articles/index.hbs", lang="handlebars"}
~~~~~~~~
<table class="primary">
  <thead>
    <tr>
      <th>Description</th>
      <th>Notes</th>
      <th>Borrowed since</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    {{#each model as |article|}}
      {{articles/article-row
        article=article
        save="save"
        articleStates=possibleStates
      }}
    {{/each}}
  </tbody>
</table>
~~~~~~~~

We are iterating over every article in the model and then rendering an
articles-row component for each of them, we are passing as attributes
the article, bounding the save action to another action which is also
called "save" and finally binding the `articleStates` to the list of
`possibleStates` in the controller. We could have call both properties
the same but we did not, so we could demonstrate that we are only
assigning variables, currently is a two-way binding, meaning that if
we modify the list in any of the component then it will be modified in
the controller too, in future versions will be able to specify a
one-way binding meaning that this data will be read-only.

To clarify a bit further, with `save="save"` as soon as we call
`this.sendAction('save', article)` then if the controller has an
action called save, it will be called otherwise it will keep bubbling
as any other action (controller, route, parents and so on).

I> In upcoming versions of ember, we'll be able to use components as
I> if they were just another HTML tag, so we could write
I> <articles/article-row> instead of {{articles/article-row}}.

If we open the **ember-inspector**, open the view tree and then select
components, we will notice that every component is displayed
independently.


I> ## is-attributes
I> The following are the attributes of the type **isSomething** and can be found in
I> [DS.Model documentation](http://emberjs.com/api/data/classes/DS.Model.html#property_isDeleted):
I> * isDeleted
I> * hasDirtyAttributes
I> * isEmpty
I> * isError
I> * isLoaded
I> * isLoading
I> * isNew
I> * isReloading
I> * isSaving
I> * isValid

If we go to the browser and try what we just created, everything should
work. Except that if we click save, our object is not saved because
we don't have a handler for the **save** action.

We can add one in **app/routes/articles/index.js**:

{title="Add save action to app/routes/articles/index.js"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.modelFor('friends/show').get('articles');
  },
  actions: {
    save(model) {
      model.save();
      return false;
    }
  }
});
~~~~~~~~

I> Remember that actions always bubble to the parents. If we had a
I> **save** action in the index controller, it would have been called
I> first and then bubbled up if we returned **true**.

## Implementing auto save.

Instead of clicking the save button every time we change the
state of the model, we want it to save automatically.

We'll rewrite our template so the save button is not included and then
use the "action" property which will be called whenever the select tag
receives a change event.

{title="app/templates/components/articles/article-row.hbs", lang="handlebars"}
~~~~~~~~
<td>{{article.description}}</td>
<td>{{article.notes}}</td>
<td>{{article.createdAt}}</td>
<td>
  {{#x-select value=article.state action="saveArticle"}}
    {{#each articleStates as |state|}}
      {{#x-option value=state}}{{state}}{{/x-option}}
    {{/each}}
  {{/x-select}}
</td>
<td>
  {{#if article.isSaving}}
    <p>Saving ...</p>
  {{/if}}
</td>
~~~~~~~~


On the component, we need to change the save article action, instead
of receiving the article, it will get it directly from the component:

{title="app/components/articles/article-row.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Component.extend({
  tagName: 'tr',
  article: null, // passed-in
  articleStates: null,   // passed-in
  actions: {
    saveArticle() {
      let article = this.get('article');

      if (article.get('hasDirtyAttributes')) {
        this.sendAction('save', article);
      }
    }
  }
});
~~~~~~~~

Now, every time we change the state of the article, the `saveArticle`
action will be called.


## Route hooks

If we go to
[http://localhost:4200/friends/new](http://localhost:4200/friends/new)
and click cancel without entering anything, or we write something and
then click cancel, we'll still see the unsaved record in our **friends
index**. It only goes away if we refresh the app.

![Unsaved friends](images/new-friend-records.jpg)

The same happens with an article. If we try to create one but we click
cancel, it will appear in the index anyway.

![Unsaved articles](images/unsaved-articles.jpg)


It is important to remember that the **ember data Store** not only
keeps all the data we load from the server, but it also keeps the one
we create on the client. We were actually pushing a new record to the
store when we did the following on the **friends new route**:

{title=""app/routes/friends/new.js", lang="JavaScript"}
~~~~~~~~
  model() {
    return this.store.createRecord('friend');
  },
~~~~~~~~

Such records will live in the store with the state **new**. We can
call **save** on it, which will persist it to the backend and make it
move to a different state, or we can remove it and our backend will
never know about it.

We might ask ourselves: but aren't we doing a **store.findAll** on the
**friends index route**, which loads our data again from the server? And
shouldn't that remove the unsaved records?

That's partially true. It is correct that when we do **this.store.findAll('friend')**,
a **GET** request is made to the server. When we load our existing
records again, instead of throwing out all the records in the
store, **ember data** merges the results, updating existing records
and leaving untouched the ones that the server doesn't know about.
That's why we see the new but unsaved record in the index.

To mitigate this situation, if we are leaving the **friends new
route** and the model was not saved, we'll need to remove the record
we created from the store. How do we do that?


[Ember.Route](http://emberjs.com/api/classes/Ember.Route.html) has a
set of hooks that are called at different times during the route
lifetime. For instance, we can use
[activate](http://emberjs.com/api/classes/Ember.Route.html#method_activate)
to do something when we enter a route,
[deactivate](http://emberjs.com/api/classes/Ember.Route.html#method_deactivate)
when we leave it or
[resetController](http://emberjs.com/api/classes/Ember.Route.html#method_resetController)
to reset values on some actions.

Let's try them in **app/routes/friends/new.js**:

{title="Using Route Hooks in app/routes/friends/new.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.store.createRecord('friend');
  },
  activate() {
    console.log('----- activate hook called -----');
  },
  deactivate() {
    console.log('----- deactivate hook called -----');
  },
  resetController: function (controller, isExiting, transition) {
    if (isExiting) {
      console.log('----- resetController hook called -----');
    }
  },
  // actions omitted for clarity
});
~~~~~~~~

And then visit
[http://localhost:4200/friends/new](http://localhost:4200/friends/new)
and click cancel or friends.

We should see something like the following in our browser's console:

![Activate and Deactivate hooks](images/activate-deactivate-hook.jpg)

Coming back to our original problem of the unsaved record in the
store, we can use the **resetController** hook to clean up our state.

Let's rewrite **app/routes/friends/new.js** so the **resetController** hook does what we expect:

{title="Cleaning up the store on resetController in app/routes/friends/new.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.store.createRecord('friend');
  },
  resetController(controller, isExiting) {
    if (isExiting) {
      // We grab the model from the controller
      //
      var model = controller.get('model');

      // Because we are leaving the Route we verify if the model is in
      // 'isNew' state, which means it wasn't saved to the backend.
      //
      if (model.get('isNew')) {

        // We call DS#destroyRecord() which removes it from the store
        //
        model.destroyRecord();
      }
    }
  }
});
~~~~~~~~


Another scenario where it is common to use the resetController hook
involves the **edit routes**. For example, if we try to edit a friend
and don't save the changes but click cancel, the friend profile will
still show whatever change we leave unsaved. To solve this problem
we'll use the **resetController** hook, but instead of checking if the
model **isNew**, we'll call **model.rollback()**. This will return the
attributes to their initial state if the model `hasDirtyAttributes`.

{title="Using resetController hook app/routes/friends/edit.js",  lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Route.extend({
  resetController(controller, isExiting) {
    if (isExiting) {
      var model = controller.get('model');
      model.rollback();
    }
  }
});
~~~~~~~~


X> ## Tasks
X> We  have the same problem on the **articles index route**. Implement
X> the **resetController** hook so that any unsaved articles are not shown in the index.
