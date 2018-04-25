# An Introduction to Mongoose

We saw some of the weaknesses of document databases this morning.
- We have to use a clunky query language to manipulate data
- There's no protection against entering data in any arbitrary format, and no validation of any sort.  

Mongoose sits in between your app and your Mongo Database, and solves both of these problems. 

## Objectives

* Use Mongoose to enforce completeness and validity requirements on data
* Use Mongoose as an object-document mapper within a Javascript program
* Add Mongoose to an existing Express application
* Seed the database with data

## Set up for this Lesson

For this lesson I will show each part of the code, and then add it to a branch called 'adding-mongo-and-mongoose' on the [express-full-crud](https://git.generalassemb.ly/WDIplus-ATX/express-full-crud) repo. I'll push up this code so you can reference it in the future. On Thursday your warm up will be to do this same thing on the 'demo' branch! 

## Installation and Scaffolding

At the top level of the repository:

```bash
npm install mongoose --save
```

We say `--save` because we want Mongoose to become a required dependency for our project.

At the beginning of your file:

```js
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/contacts');
```
## Mongoose is an Object-Document Mapper

What does that mean?  We have objects in our Javascript and documents in our MongoDB, and Mongoose bridges between them.

So before we start dealing with databases, we're going to start putting together a command-line contacts command.

## Mongoose has Schemas that turn into Models

```js
var Schema = mongoose.Schema;

var contactSchema = newSchema({

    // we define what we're willing to accept in a Contact
    // keys are as usual; values are String, Number, Date,
    // Boolean, ObjectId, Array

});
```

Now data that doesn't look like how we want it won't make it in! I mentioned before, once we have a database we our model won't have actual data, it just tells our Database what the data should look like. 

## Virtual Attributes

We can add calculated attributes to the model too.  These are called 'virtual attributes.'  Assume we have name.first and name.last properties: we can derive a name.full property from them.

```js
contactSchema.virtual('name.full').get(function () {
  return this.name.first + ' ' + this.name.last;
});

contactSchema.virtual('name.full').set(function (name) {
  var split = name.split(' ');
  this.name.first = split[0];
  this.name.last = split[1];
});
```

When all that is done:

```js
var Contact = mongoose.model( 'Contact', contactSchema);
```

## Using models in our classes

We can use Mongo syntax to find things!

We can also use Mongoose query syntax:

```js
var query = Contacts.findOne({ 'name.last': 'Miller' });
query.select('name.first title company');
query.exec(function(err, contact){
    console.log('full name: %s, title: %s, company %s',
        contact.name.full, contact.title, contact.company);
});
```

## Validation

Back to the schema!

```js
var contactSchema = newSchema({

    // first and last names are required

    name: {
        first: { type: String, required: true },
        last: { type: String, required: true },
    },

    // title is not required

    title: String,

    // age must be between 21 and 65

    age: { type: Number, min: 21, max: 65 },

    // political party must be in the list

    politicalParty: {  type: String, 
            enum: { values: 'Republican Democrat Libertarian Green Peace&Freedom',
                    message: 'invalid party' },
            }
        },

    // phone numbers must match the regular expression

    phoneNumber: { type: String,
        match: /(\d?\D*\d{3}\D*\d{3}\D*\d{4})/ },

});
```

```js
var c = Contact.create({ ...info... });
c.save();
```

## Route examples

```js
router.get('/lessons', function(req, res){
    // old code for hard-coded data
     res.render('lessons/index', {lessonsArray: lessonsArray });
    // new code:
    Lesson.find(function(err, lessons) {
      res.render('lessons/index', {lessonsArray: lessons });
    });
});



router.post('/lessons', urlencodedParser, function(req, res) {
    // old code for hard-coded data
	// lessonsArray.push(req.body);
    // using mongoose:
    
    new Lesson({title: req.body.title, githubUrl: req.body.githubUrl }).save();

	res.redirect('/lessons');
});


```
## Seeding the database 

It can be a pain to manually add data to the database, so we can create a JS script that 'seeds' our database automoatically. 

seed.js: 

```js
const mongoose = require('mongoose');
const Lesson = require('./models/lessons');
mongoose.connect('mongodb://localhost/lessons');


// seed our database

const seedLessons = () => {
    // create some lessons
	const Lessons = [
		{ title: "express full crud", githubUrl: "https://git.generalassemb.ly/WDIplus-ATX/express-full-crud", keywords: ["express", "crud", "ejs"]},
		{ title: "mvp and agile", githubUrl: "https://git.generalassemb.ly/WDIplus-ATX/mvp-and-agile/blob/master/README.md", keywords: ["mvp", "project management", "agile"]},
		{ title: "intro to REST", githubUrl: "https://git.generalassemb.ly/WDIplus-ATX/intro-to-rest-with-express/blob/master/README.md", keywords: ["REST", "API", "express"]}
	];

	for ( thisLesson of Lessons) {
		var newLesson = new Lesson(thisLesson);
		newLesson.save();
		console.log(newLesson)
	}

    	console.log('Database seeded!');
}
  
seedLessons();
```

```bash
node seed.js
```
## Adding Mongo + Mongoose to an App

In server.js

```js
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/cats');

//Get the default connection
var db = mongoose.connection;

//Bind connection to error event (to get notification of connection errors)
db.on('error', console.error.bind(console, 'MongoDB connection error:'));

```

In lesson.js
```js
var mongoose = require('mongoose');
var db = mongoose.createConnection('localhost', 'test');

var schema = mongoose.Schema({ name: 'string' });
var Lesson = db.model('Lesson', schema);

module.exports = Lesson;
```

In routes/lessons.js
```js
router.get('/lessons', function(req, res){
    // old code for hard-coded data
     res.render('lessons/index', {lessonsArray: lessonsArray });
    // new code:
    Lesson.find(function(err, lessons) {
      res.render('cats/index', {catsArray: lessons });
    });
});



router.post('/lessons', urlencodedParser, function(req, res) {
    // old code for hard-coded data
	// lessonsArray.push(req.body);
    // using mongoose:
    
    new Lesson({title: req.body.title, githubUrl: req.body.githubUrl }).save();

	res.redirect('/lessons');
});
```

Add the seed file from above in the root directory, and test it by running `node seed.js` in terminial!

## Important: When using Mongoose in your Express app, you have to run `mongod` in terminal from within your project directory!

```bash
mongod
```

&#x1F535; **Activity**
```
- We see above how to GET all lessons, and POST a new lesson! How would we make a route to GET 1 lesson, PUT an update to a lesson, or DELETE a lesson? Check out the docs and find out: http://mongoosejs.com/index.html
- Respond in a thread with your code!
- This is just an experiment. For lab 2 + homework + Thursday warm up, you will have practice implementing this for real!
- 10 minutes
```

## References

* The Mongoose API docs at [http://mongoosejs.com/docs/api.html](http://mongoosejs.com/docs/api.html)
* A sample CRUD app with Mongoose in a single file: https://gist.github.com/arvis/3357699

