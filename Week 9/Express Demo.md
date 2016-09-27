# Express Demo

Express is a "Fast, unopinionated, minimalist web framework for Node.js".

A quick way to understand what Express is - is to look back at our analogy bank.

`Draw the SWAC stack on the surface`<br />
`Draw the MEAN stack on the surface`<br />

## High level steps

- [ ] Supporting Tools
- [ ] Project Setup
- [ ] Hello World
- [ ] Routing
- [ ] Setting up REST endpoints
- [ ] Mongoose

## Supporting Tools
- [ ] Make sure you have NodeJS installed

## Project Setup
`npm init`
`npm install express --save`

## Hello World

**server.js**

```js
var express = require('express');

var app = express();

app.get('/', function(req, res) {
	res.send('hello world');
});

app.listen(8080, function() {
	console.log('Listening on port 8080');
});
```

`node server.js`

## Top Spots and JSON

**server.js**

```js
/* require modules */
var express = require('express');

/* run javascript */
var app = express();

app.get('/', function(req, res) {
	var topspots = [
		{
			name: 'Balboa Park',
			location: [12,14]
		},
		{
			name: 'NY Pizza',
			location: [14,14]
		}
	];

	res.json(topspots);
});

app.listen(8080, function() {
	console.log('app listening on port 8080');
});
```

## Setting up REST endpoints

**server.js**

```js
var express = require('express');

var app = express();
var apiRouter = express.Router();

var projectRoute = require('./routes/project.route');
var studentRoute = require('./routes/student.route');

apiRouter.use('/projects', projectRoute);
apiRouter.use('/students', studentRoute);

app.use(apiRouter);

app.listen(8080, function() {
	console.log('app listening on port 8080');
});
```

**routes/project.route.js**

```js
var express = require('express');
var router = express.Router();

router.route('/')
		  .get(function(req, res) {
		  	// write code to get all projects
		  	res.json([{},{}]);
		  })
		  .post(function(req, res) {
		  	// write code to add incoming project to database
		  	var project = req.body;
		  	res.json({});
		  });

router.route('/:id')
			.get(function(req, res) {
				// write code to get a single project
				res.json({id: req.params.id });
			})
			.delete(function(req, res) {
				// write code to delete a single project
				res.json({id: req.params.id});
			})
			.put(function(req, res) {
				// write code to update a single project
				var id = req.params.id;
				var project = req.body;

				res.json(project);
			});

module.exports = router;
```

**routes/student.route.js**

```js
var express = require('express');
var router = express.Router();

router.route('/')
		  .get(function(req, res) {
		  	// write code to get all students
		  	res.json([{},{}]);
		  })
		  .post(function(req, res) {
		  	// write code to add incoming student to database
		  	var student = req.body;
		  	res.json({});
		  });

router.route('/:id')
			.get(function(req, res) {
				// write code to get a single student
				res.json({id: req.params.id });
			})
			.delete(function(req, res) {
				// write code to delete a single student
				res.json({id: req.params.id});
			})
			.put(function(req, res) {
				// write code to update a single student
				var id = req.params.id;
				var student = req.body;

				res.json(student);
			});

module.exports = router;
```

## Mongoose
Mongoose is an Object Document Mapper that is used to facilitate communication between an express endpoint and a MongoDB database.

### Install mongoose into your project

`npm install mongoose --save`

### Create model classes

**models/project.model.js**

```js
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var projectSchema = new Schema({
	name: {
		type: 'string',
		required: true
	},
	description: {
		type: 'string',
		required: false
	},
	studentIds: [{
		type: Schema.Types.ObjectId,
		ref: 'student'
	}]
});

module.exports = mongoose.model('project', projectSchema);
```

**models/student.model.js**

```js
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var studentSchema = new Schema({
	firstName: {
		type: 'string', // string,number,date,mixed,objectid,array
		required: true
	},
	lastName: {
		type: 'string',
		required: true
	},
	email: {
		type: 'string',
		required: true
	},
	phone: {
		type: 'string',
		required: true
	},
	projectIds: [{
		type: Schema.Types.ObjectId,
		ref: 'project'
	}]
});

module.exports = mongoose.model('student', studentSchema);
```

### Configure connection in `server.js`

```js
/* configure mongoose */
var mongoose = require('mongoose');
mongoose.connect("mongodb://localhost:27017/classroom");

/* configure express */
var express = require('express');

var app = express();
var apiRouter = express.Router();

var projectRoute = require('./routes/project.route');
var studentRoute = require('./routes/student.route');

apiRouter.use('/projects', projectRoute);
apiRouter.use('/students', studentRoute);

app.use(apiRouter);

app.listen(8080, function() {
	console.log('app listening on port 8080');
});
```

### Refactor routes to use Mongoose

**routes/project.route.js**

```js
var express = require('express');
var router = express.Router();
var Project = require('../models/project.model');

router.route('/')
		  .get(function(req, res) {
		  	// write code to get all projects
		  	Project.find(function(err, data) {
		  		if(err) {
		  			res.send(500, err);
		  		}	

		  		res.json(data);
		  	});
		  })
		  .post(function(req, res) {
		  	// write code to add incoming project to database
		  	var project = new Project();
		  	project.name = req.body.name;
		  	project.description = req.body.description;

		  	project.save(function(err, project) {
		  		if(err) {
		  			res.send(500, err);
		  		}

		  		res.json(project);
		  	});
		  });

router.route('/:id')
			.get(function(req, res) {
				// write code to get a single project
				Project.findById(req.params.id, function(err, project) {
					if(err) {
						res.send(500, err);
					}

					res.json(project);
				});
			})
			.delete(function(req, res) {
				// write code to delete a single project
				Project.remove({ _id: req.params.id }, function(err, project) {
					if(err) {
						res.send(500, err);
					}

					res.json(project);
				});
			})
			.put(function(req, res) {
				// write code to update a single project
				Project.findById(req.params.id, function(err, project) {
					if(err) {
						res.send(500, err);
					}

					project.name = req.body.name;
					project.description = req.body.description;

					project.save(function(err, project) {
						if(err) {
							res.send(500, err);
						}

						res.json(200);
					});
				});
			});

module.exports = router;
```

**routes/student.route.js**

```js
var express = require('express');
var router = express.Router();

router.route('/')
		  .get(function(req, res) {
		  	// write code to get all students
		  	res.json([{},{}]);
		  })
		  .post(function(req, res) {
		  	// write code to add incoming student to database
		  	var student = req.body;
		  	res.json({});
		  });

router.route('/:id')
			.get(function(req, res) {
				// write code to get a single student
				res.json({id: req.params.id });
			})
			.delete(function(req, res) {
				// write code to delete a single student
				res.json({id: req.params.id});
			})
			.put(function(req, res) {
				// write code to update a single student
				var id = req.params.id;
				var student = req.body;

				res.json(student);
			});

module.exports = router;
```

### Test the endpoints in Postman
**Note that the post endpoint will not work, because express needs a little extra help parsing JSON into req.body**

`npm install body-parser --save`

**server.js**

```js
/* configure mongoose */
var mongoose = require('mongoose');
mongoose.connect("mongodb://localhost:27017/classroom");

/* configure express */
var express = require('express');
var bodyParser = require('body-parser');

var app = express();
app.use(bodyParser.json());

var apiRouter = express.Router();

var projectRoute = require('./routes/project.route');
var studentRoute = require('./routes/student.route');

apiRouter.use('/projects', projectRoute);
apiRouter.use('/students', studentRoute);

app.use(apiRouter);

app.listen(8080, function() {
	console.log('app listening on port 8080');
});
```

### Assigning projects to students

The old route we had setup to do this was to post to `/api/assignments`.

Let's set this up in Express.

