#MEAN Session Eleven

##Homework
- get the gulpfile working with gulp-nodemon and browser sync working together
- FINAL PROJECT - build an admin interface to the recipesApp that allows a user to create a new recipe 
- this should include form validation

In this class we pick up our Recipes page and add an api to the (already built) front end.

`$ npm install`

`$ gulp`

Review the project. 

Trace the use of components from index.html to the recipe-detail and recipe-lst directories. Note the use of static json files for data.

Our exercise used BrowserSync for its server. We'll begin by using Express instead. Halt the gulp process and install express:

`$ npm install --save express`

Create `server.js` for express:

```js
var express = require('express');
var app = express();

var port = process.env.PORT || 8080;        // e.g.  $ PORT=9543 node app.js

app.get('/', function(req, res) {
  res.send('Return JSON or HTML View');
});

app.listen(port);
console.log('Server running at ' + port);
```

Now run `node server` and the view is gone. Recall, we need to specify static assets. We'll do this later.

###Rest API
* A URL route schema to map requests to app actions
* With a Controller to handle each action
* Data to respond with
* Place to store the data
* An interface to access and change data

###Routes

Predefined URL paths your API responds to. Think of each Route as listening to three parts:

* A specific HTTP Action
* A specific URL path
* A handler method

###GET

This example of routing handles all GET Requests. The URL path is the root of the site, the handling method is an anonymous function, and the response plain text:

```js
app.get('/', function (req, res) {
    res.send('Return JSON or HTML View');
    console.dir(res);
});
```
Note that the server needs to be restarted in order for us to see the results.

Refresh the browser and note the res (response) object being dumped into the console.

##Install nodemon

`sudo npm install -g nodemon`

`sudo npm install --save-dev nodemon`

Run the app using `$ PORT=9003 nodemon server.js` 

Any changes to server.js will restart the app. Comment `console.dir(res);` out and save to see the refresh.

You wil need to keep an eye on the nodemon process during this exercise to see if it is hanging.

###GET Requests with Parameters

```js
app.get('/recipe/:name', function(req, res) {
   console.log(req.params.name)
});
```

Will log the param to the terminal.

Run `http://localhost:XXXX/recipe/lasagne` noting the terminal console's output.


###Mongoose.js

A [Mongo Driver](http://mongoosejs.com) to model your application data.

`npm install mongoose -g`

Use NPM to install this dependency and update your package.json file.

`npm install mongoose --save`

[Quickstart guide](http://mongoosejs.com/docs/) for Mongoose.

###Body Parser

[Body Parser](https://www.npmjs.com/package/body-parser) parses and places incoming requests in a `req.body` property so our handlers can use them.

Run `npm install body-parser --save`


###Update server.js:

```js
var express = require('express');
var mongoose = require('mongoose');
var app = express();
var bodyParser = require('body-parser');
var mongoUri = 'mongodb://localhost/recipe-api';
var db = mongoose.connection;
mongoose.connect(mongoUri);

// configure app to use bodyParser()
// this will let us get the data from a POST
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

var port = process.env.PORT || 8080;        // set our port

// ROUTES FOR OUR API
// =============================================================================
var router = express.Router();              // get an instance of the express Router

// test route accessed at GET http://localhost:XXXX/api
router.get('/', function (req, res) {
    res.json({ message: 'hello from our api' });
});

// more routes for our API will happen here

// all of our routes will be prefixed with /api
app.use('/api', router);

app.listen(port);
console.log('Listening on port ' + port);
```

Note that we are using Express' Router: `express.Router();` and have directed express to use `/api` for all its calls: `app.use('/api', router);`

See the [Express api documentation](http://expressjs.com/en/api.html) for details.

Test the api url in a browser. You should see the return json.


###Define a Mongoose Model

Create a new folder called models and add a new file recipe.js for our Recipe Model.

Require Mongoose into this file, and create a new Schema object:

```js
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var RecipeSchema = new Schema({
    name: String
});

module.exports = mongoose.model('Recipe', RecipeSchema);
```

A schema makes sure we're getting and setting well-formed data to and from the Mongo collection. We're starting out with a simple String element.
 
The last line creates the Recipe model object, with built in Mongo interfacing methods. We'll refer to this Recipe object in other files.
 
Ensure that `var Recipe = require('./models/recipe');` is in server.js

```js

var Recipe = require('./models/recipe');

```

Add a function for the router that will allow us to perform logging. This is middleware that fires on all requests:

```js
router.use(function(req, res, next) {
    // do logging
    console.log('Something is happening.');
    next(); // make sure we go to the next routes and don't stop here
});
```

`router.use()` and `next()` - we can perform logging here and then continue to the next route using `next()`. Using middleware we can do validations, throw errors in case something is wrong or do some extra logging for analytics or any statistics we’d like to keep. This is frequently used to check for logged in status.

Here is the completed server file to this point:

```js
// call the packages we need

var express = require('express');
var mongoose = require('mongoose');
var app = express();
var bodyParser = require('body-parser');
var mongoUri = 'mongodb://localhost/recipe-api';
var db = mongoose.connection;
mongoose.connect(mongoUri);

var Recipe = require('./app/models/recipe');

// configure app to use bodyParser()
// this will let us get the data from a POST
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

var port = process.env.PORT || 8080;        // set our port

// ROUTES FOR OUR API
// =============================================================================
var router = express.Router();              // get an instance of the express Router

// middleware to use for all requests
router.use(function (req, res, next) {
    // do logging
    console.log('Something is happening.');
    next(); // make sure we go to the next routes and don't stop here
});

// test route to make sure everything is working (accessed at GET http://localhost:XXXX/api)
router.get('/', function (req, res) {
    res.json({ message: 'hooray! welcome to our api!' });
});

// more routes for our API will happen here

// REGISTER OUR ROUTES -------------------------------
// all of our routes will be prefixed with /api
app.use('/api', router);

// START THE SERVER
// =============================================================================
app.listen(port);
console.log('Magic happens on port ' + port);
```


##Saving a Simple Recipe

Create the recipes post route (it will use `/api`) 

```js
// more routes for our API will happen here

// on routes that end in /recipes
// ----------------------------------------------------
router.route('/recipes')

    // create a recipe (accessed at POST http://localhost:8080/api/recipes)
    .post(function (req, res) {
        
        var recipe = new Recipe();      // create a new instance of the Recipe model
        recipe.name = req.body.name;  // set the recipes name (comes from the request)

        // save the recipe and check for errors
        recipe.save(function(err) {
            if (err)
                res.send(err);

            res.json({ message: 'Recipe created!' });
        });
        
    });
```

###Test in Postman

Since this is a post we need to use postman to test it. Use `{ "name" : "berries" }` on a post. We might also change the res.send value to something more useful:

```js
recipe.save(function(err) {
    if (err)
        res.send(err);

    res.send(recipe);
});
```

###Mongo

Run mongo in a new terminal to check the creation.

```
$ mongo
> show dbs
> use recipe-api
> show collections
> db.recipes.find()
```


Here's the complete server.js file to this point.

```js
// call the packages we need

var express = require('express');
var mongoose = require('mongoose');
var app = express();
var bodyParser = require('body-parser');
var mongoUri = 'mongodb://localhost/recipe-api';
var db = mongoose.connection;
mongoose.connect(mongoUri);

var Recipe = require('./app/models/recipe');

// configure app to use bodyParser()
// this will let us get the data from a POST
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

var port = process.env.PORT || 8080;        // set our port

// ROUTES FOR OUR API
// =============================================================================
var router = express.Router();              // get an instance of the express Router

// middleware to use for all requests
router.use(function (req, res, next) {
    // do logging
    console.log('Something is happening.');
    next(); // make sure we go to the next routes and don't stop here
});

// test route to make sure everything is working (accessed at GET http://localhost:8080/api)
router.get('/', function (req, res) {
    res.json({ message: 'hello from our api!' });
});

// more routes for our API will happen here

// on routes that end in /recipes
// ----------------------------------------------------
router.route('/recipes')

    // create a recipe (accessed at POST http://localhost:XXXX/api/recipes)
    .post(function (req, res) {

        var recipe = new Recipe();      // create a new instance of the Recipe model
        recipe.name = req.body.name;  // set the recipes name (comes from the request)

        // save the recipe and check for errors
        recipe.save(function(err) {
            if (err)
                res.send(err);

            res.send(recipe);
        });

    });

// REGISTER OUR ROUTES -------------------------------
// all of our routes will be prefixed with /api
app.use('/api', router);

// START THE SERVER
// =============================================================================
app.listen(port);
console.log('Magic happens on port ' + port);
```


##Getting All Recipes

Let's use GET to see the recipes we are creating.

```js
    .get(function (req, res) {
        Recipe.find(function (err, recipes) {
            if (err)
                res.send(err);
            res.json(recipes);
        });
    });
```

e.g.

```js
router.route('/recipes')
    .post(function (req, res) {
        var recipe = new Recipe();
        recipe.name = req.body.name;
        recipe.save(function (err) {
            if (err)
                res.send(err);
            res.send(recipe);
        });
    })
    .get(function (req, res) {
        Recipe.find(function (err, recipes) {
            if (err)
                res.send(err);
            res.json(recipes);
        });
    });
```

Test getting in postman. Use the mongo cli `db.recipes.remove({})` to destroy them if necessary.


##Routes for Single Recipes (id)

We’ve handled the group for routes ending in /recipes. Let’s now handle the routes for when we pass in a parameter like a recipe's id.

The things we’ll want to do for this route, which will end in /recipes/:recipe_id will be:

- Get a single recipe.
- Update a recipe's info.
- Delete a recipe.

Add another router.route() to handle all requests that have a :recipe_id attached to them.

```js
// on routes that end in /recipe/:recipe_id
// ----------------------------------------------------
router.route('/recipes/:recipe_id')

    // get the recipe with that id (accessed at GET http://localhost:XXXX/api/recipes/:recipe_id)
    .get(function (req, res) {
        Recipe.findById(req.params.recipe_id, function (err, recipe) {
            if (err)
                res.send(err);
            res.json(recipe);
        });
    });

// REGISTER OUR ROUTES -------------------------------
```

Test using postman.


##Integration with the Frontend

Serve our static content. Add `app.use(express.static('app'))` to server.js

See http://expressjs.com/en/api.html

```js
// this will let us get the data from a POST
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

app.use(express.static('app'))
...
```

Test by going to the root in the browser e.g. `http://localhost:XXXX/#!/`


##Adding / Removing Recipes

Add a recipe using Mongo `db.recipes.insert()` and our recipes.json.

https://docs.mongodb.com/v3.2/tutorial/insert-documents/

https://docs.mongodb.com/v3.2/tutorial/remove-documents/

```
$ mongo
> show dbs
> use recipe-api
> show collections
> db.recipes.insert() 
> db.recipes.find()
> db.recipes.deleteOne( { name : "recipe1404" } ) // ???
> db.recipes.remove({})
```

Add a recipe using postman and our json. 

Note that only the name is inserted. This is our mongoose schema at work.

Use the mongoose create function:

```js
    // create a recipe (accessed at POST http://localhost:8080/api/recipes)
    .post(function (req, res) {
        var recipe = new Recipe(req); 
        Recipe.create(req.body, function (err) {
            if (err)
                res.send(err);
            res.send(recipe);
        });
    })
```

Edit our schema - note the use of server date as a default:

```js
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var RecipeSchema = new Schema({
    name: String,
    title: String,
    date: { type: Date, default: Date.now },
    description: String,
    image: String
});

module.exports = mongoose.model('Recipe', RecipeSchema);
```

Test in postman again with no date being passed.


##Editing the Frontend

Change the $http call to get info from the db in `recipe-list.component.js`:

```js
angular.module('recipeApp').component('recipeList', {
    templateUrl: 'recipe-list/recipe-list.template.html',
    controller: function RecipeListController($http) {
        var self = this;
        self.orderProp = 'date';

        $http.get('api/recipes').then(
            function (response) {
                self.recipes = response.data;
            }
```

View in the browser. Note - the detail view is still using our .json file.


##Detail View

Change the url for recipe to use _id in the list template:

```
<ul class="recipes-list">
	<li ng-repeat="recipe in $ctrl.recipes | filter:$ctrl.query | orderBy:$ctrl.orderProp">
		<img ng-src="img/home/{{ recipe.image }}">
		<h1><a href="#!recipes/{{ recipe._id }} ">{{ recipe.title }}</a></h1>
		<p>{{ recipe.description }}</p>
	</li>
</ul>
```

Test by clicking on the link in the list view.

Ammend the $http call in our detail controller:

```js
angular.module('recipeDetail').component('recipeDetail', {
    templateUrl: 'recipe-detail/recipe-detail.template.html',
    controller: ['$http', '$routeParams',
        function RecipeDetailController($http, $routeParams) {
            var self = this;

            self.setImage = function setImage(imageUrl) {
                self.mainImageUrl = imageUrl;
            };
            console.log($routeParams)
            $http.get('/api/recipes/' + $routeParams.recipeId).then(function (response) {
                self.recipe = response.data;
                self.setImage(self.recipe.images[0]);
            });
        }
    ]
});
```

Test in browser - this only return the description because the other data is missing.

Review the json, remove all recipes in mongo cli `db.recipes.remove({})` and add one with the additional content using postman:

```js
{
  "name": "recipe1309", 
  "title": "Lasagna", 
  "date": "2013-09-01", 
  "description": "Lasagna noodles piled high and layered full of three kinds of cheese to go along with the perfect blend of meaty and zesty, tomato pasta sauce all loaded with herbs.", 
  "mainImageUrl": "img/home/lasagna-1.png",
  "image": "lasagne.png",
  "images": ["lasagna-1.png","lasagna-2.png","lasagna-3.png","lasagna-4.png"],
  "ingredients": ["lasagna pasta", "tomatoes", "onions", "ground beef", "garlic", "cheese"]
}
```

Again, content not specified in our model is not added.

Ammend the schema:

```js
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var RecipeSchema = new Schema({
    name: String,
    title: String,
    date: { type: Date, default: Date.now },
    description: String,
    image: String,
    images: [],
    ingredients: []
});

module.exports = mongoose.model('Recipe', RecipeSchema);
```


##Notes

`https://github.com/sogko/gulp-recipes/blob/master/browser-sync-nodemon-expressjs/gulpfile.js`

`$ sudo npm install --save-dev gulp-nodemon`

In gulpfile:

`var nodemon = require('gulp-nodemon');`

https://www.npmjs.com/package/gulp-nodemon

```
gulp.task('start', function () {
  nodemon({
    script: 'server.js'
  , ext: 'js html'
  , env: { 'NODE_ENV': 'development' }
  })
})
```

https://www.browsersync.io/docs/options/#option-proxy

`proxy: "localhost:8888"`

=======


#Notes

Start with index.html

Add CSS to create the animations.

```
    .box {

      transition: width 0.2s, height 0.6s;

    }

    .box.opening {
      width:500px;
      height:500px;
    }
```

Refresh and test by adding the opening class to the box div.

```
    .box h2 {

      transform:translateX(-200%);
      transition: all 0.5s;

    }

    .box p {

      transform:translateX(200%);
      transition: all 0.5s;

    }

    .box.open > * {
      transform:translateX(0%);
    }

```

Refresh and test by adding the opening and open class to the box div.


##JavaScript


###Variables - var

1. var can change
```
var width = 100;
width = 200;
```

and can be used 2x without errors:
```
var width = 100;
var width = 200;
```


2. They have function scope
```
function setWidth(){
	var width = 100;
	console.log('inside' + width);
}

setWidth();

console.log('outside' + width);
```

3. Problem with var (scope leakage)
```
var age = 100;
if(age > 12){
	var dogYears = age * 7
	console.log(`You are ${dogYears} dog years old.`);
}

console.log(dogYears);
```

`let` and `const` are scoped to the block (block scoped).

`let dogYears = age * 7`
`const dogYears = age * 7`

`let` and `const` vars cannot be reused.

```
let width = 100;
let width = 200;
```

You can update a `let` var

```
let width = 100;
    width = 200;
```

`let` is scoped to the block (contrast with var)

```
let points = 50;
let winner = false;

if(points > 40){
	let winner = true
}

console.log(winner)
```

`const` cannot be updated

```
const key = '123abc';
key = 'abcd1234'
```
##Arrow Functions

1. concise
2. implicit returns
3. don't rebind the value of `this`

```
const names = ['alex', 'sarah', 'julio'];

const fullNames = names.map(function(name){
	return `${name} developer`;
});

console.log(fullNames);
```

1. concise
```
const names = ['alex', 'sarah', 'julio'];

const fullNames2 = names.map((name) => {
  return `${name} developer`;
});

// if only one param is passed no need for brackets

const fullNames3 = names.map(name => {
  return `${name} developer`;
});

console.log(fullNames3);
```

2. Implicit return 

```
const names = ['alex', 'sarah', 'julio'];

// delete return, move to one line and delete curly brackets
const fullNames4 = names.map(name => `${name} developer`);

// no arguments
const fullNames5 = names.map(() => `good developer`);

console.log(fullNames5);
```

Named Functions

```
function sayMyName(name){
	alert(`hello ${name}`);
}
```

##All Arrow Functions are Anonymous Functions

But you can put them in a variable

```
const sayMyName = (name) => { alert(`hello ${name}`)}

sayMyName('Daniel');
```

In arrow functions the `this` keyword does not get rebound.

##Our script

```
<script>
const box = document.querySelector('.box');
box.addEventListener('click', function() {
  	console.log(this)
});
</script>
```

vs arrow function - still bound to the parent scope.

```
<script>
const box = document.querySelector('.box');
box.addEventListener('click', () => {
  console.log(this)
});
</script>
```

Arrow functions are good for simple one line utility functions.

```
<script>
const box = document.querySelector('.box');
box.addEventListener('click', function() {
  this.classList.toggle('opening');
});
</script>
```

When do we use arrow functions?

Error here. `this` is scoped to the outer function:

```
<script>
const box = document.querySelector('.box');
box.addEventListener('click', function() {
  this.classList.toggle('opening');
  setTimeout(function(){
    // console.log(this.classList);
    // console.log(this);
    this.classList.toggle('open');
  }, 500)
});
</script>
```

In the above the second use of this is bound to the window because we have entered a new function.

Fixed by declaring a variable:

```
<script>
const box = document.querySelector('.box');
box.addEventListener('click', function() {
  var self = this; 
  this.classList.toggle('opening');
  setTimeout(function(){
    self.classList.toggle('open');
  }, 500);
});
</script>
```

Arrow functions inherit the value of this: 

```
<script>
const box = document.querySelector('.box');
box.addEventListener('click', function() {
  this.classList.toggle('opening');
  setTimeout(() => {
    this.classList.toggle('open');
  }, 500);
});
</script>
```

Reverse the sequence (uses destructuring):

```
<script>
const box = document.querySelector('.box');
box.addEventListener('click', function() {

  let first = 'opening';
  let second = 'open';

  if(this.classList.contains(first)){
    [first, second] = [second, first];
  }
  this.classList.toggle(first);
  setTimeout(() => {
    this.classList.toggle(second);
  }, 500);
});
</script>
```






Old school techniques.



=============


```
<script>
var box = document.querySelector('.box');
box.addEventListener('click', function() {
  console.log(this)
});
</script>
```

```
<script>
var box = document.querySelector('.box');
box.addEventListener('click', function() {
  this.classList.toggle('opening');
});
</script>
```

Error here. `this` is scoped to the outer function:

```
<script>
var box = document.querySelector('.box');
box.addEventListener('click', function() {
  this.classList.toggle('opening');
  setTimeout(function(){
    // console.log(this.classList);
    // console.log(this);
    this.classList.toggle('open');
  }, 500)
});
</script>
```

Fixed:

```
<script>
var box = document.querySelector('.box');
box.addEventListener('click', function() {
  var self = this; 
  this.classList.toggle('opening');
  setTimeout(function(){
    self.classList.toggle('open');
  }, 500);
});
</script>
```

```
<script>
var box = document.querySelector('.box');
box.addEventListener('click', function() {
  var self = this; 
  var first = 'opening';
  var second = 'open';
  
  this.classList.toggle(first);
  setTimeout(function(){
    self.classList.toggle(second);
  }, 500);
});
</script>
```


```
<script>
var box = document.querySelector('.box');
box.addEventListener('click', function() {
  var self = this; 
  var first = 'opening';
  var second = 'open';
  if(this.classList.contains(first)){
    [first, second] = [second, first];
  }
  this.classList.toggle(first);
  setTimeout(function(){
    self.classList.toggle(second);
  }, 500);
});
</script>
```


```
  <script>
  const box = document.querySelector('.box');
  box.addEventListener('click', function() {

    let first = 'opening';
    let second = 'open';

    if(this.classList.contains(first)){
      [first, second] = [second, first];
    }
    this.classList.toggle(first);
    setTimeout(() => {
      this.classList.toggle(second);
    }, 500);
  });
  </script>
  ```
















































