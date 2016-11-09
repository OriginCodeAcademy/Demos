# Passport Authentication

This demo shows students how to implement passport authentication into their Express applications.

## High level steps

## Setup a basic app

- [ ] Install the following dependencies
	
	`npm install express body-parser --save`

- [ ] Implement the following files

	`server.js`

	```js
	const express = require('express');
	const bodyParser = require('body-parser');
	
	// routes
	var apiRouter = express.Router();
	var petRoute = require('./routes/petRoute');
	apiRouter.use('/pets', petRoute);
	
	// app
	var app = express();
	app.use(bodyParser.json());
	app.use(apiRouter);
	
	// server
	app.listen(8000, function() {
		console.log('Listening on port 8000');
	});
	```

	`routes/petRoute.js`
	
	```js
	var express = require('express');
	var router = express.Router();

	// unprotected route	
	router.route('/').get(function(req, res) {
		return res.json([
			{ name: 'Dylan' },
			{ name: 'Darla' }
		]);
	});
	
	// protected route - can be accessed through jwt or facebook authentication
	router.route('/').get(passport.authenticate(['jwt', 'facebook-token'], function(req, res) {
		return res.json([
			{ name: 'Dylan' },
			{ name: 'Darla' }
		]);
	});
	
	module.exports = router;
	```

- [ ] Test the app

## Setup passport

- [ ] Install the following dependencies
	
	`npm install passport passport-jwt --save`

- [ ] Modify the following files to look as follows

	`server.js`
		