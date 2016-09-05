# Unit testing with Karma and Jasmine

This demo shows students how to get up and going with unit testing in Karma / Jasmine

## High Level
* [ ] Install Karma + Jasmine
* [ ] Install Angular Test Dependencies (`angular-mocks` and `bardjs`)
* [ ] Initialize Karma
* [ ] Make changes to karma.conf.js
	* [ ] Add script dependencies to the `files` section
	* [ ] Add `sinon` to the `frameworks` array
* [ ] Run Karma
* [ ] Add `todo.controller.spec.js`
	* [ ] Write a few basic calculator tests to demo Karma in action
	* [ ] Write unit tests for TodoController, stubbing todoFactory using Sinon
* [ ] Add `todo.factory.spec.js`
	* [ ] Write unit tests for todoFactory, stubbing $http using $httpbackend

## Steps

### Install Karma + Jasmine
`npm install -g karma-cli`<br />
`npm install karma karma-jasmine jasmine-core karma-chrome-launcher karma-sinon --save-dev`

`karma`: our test runner<br />
`karma-jasmine`: plugin for karma to run jasmine tests<br />
`jasmine-core`: core library for our jasmine tests<br />
`karma-chrome-launcher`: launches chrome to run unit tests<br />
`karma-sinon`: plugin for karma to work with sinon - a mocking framework

### Install Angular Test Dependencies
`bower install angular-mocks bardjs  --save-dev`<br />

`angular-mocks`: a set of pre-written mocks for angular services<br />
`bardjs`: makes it simple to use angular-mocks in your Jasmine tests

### Init karma
"now we have the necessary packages, I'm going to initialize karma in this project."<br />
`karma init`

### Inspect karma.conf.js file

### Add files to karma.conf.js
"These need to be loaded in order for Karma to reference them for testing"
```js
files: [
  'src/bower_components/angular/angular.js',
  'src/bower_components/angular-mocks/angular-mocks.js',
  'src/bower_components/sinon/index.js',
  'src/bower_components/bardjs/dist/bard.js',
  'src/app/**/*.js'
],
```

### Add frameworks
"This tells Karma to use the following plugins as frameworks. Run jasmine tests, and use sinon as a dependency for bardjs"
```
frameworks: ['jasmine', 'sinon']
```

### Run karma for the first time
karma start

### Add spec file (todo.controller.spec.js)

### Write Tests
*Write calculator test in a spec file and demo karma running jasmine tests*
```js
describe('Calculator', function() {
	
	it('should add 2 + 2 correctly', function() {
		expect(2 + 2).toEqual(4);
	});

});
```

*Write todoController test*
```js
describe('TodoController', function() {

	var todoController;

	sinon.stub(window, 'confirm').returns(true);

	beforeEach(function() {
		bard.appModule('app');
		bard.inject('$controller', '$q', '$rootScope', 'todoFactory');
	});
	beforeEach(function() {
		sinon.stub(todoFactory, 'getAll').returns($q.when([{}, {}, {}]));
		sinon.stub(todoFactory, 'add').returns($q.when({}));
		sinon.stub(todoFactory, 'remove').returns($q.when());

		todoController = $controller('TodoController');

		$rootScope.$apply();
	});

	bard.verifyNoOutstandingHttpRequests();

	describe('Todo controller', function() {
		it('should be created successfully', function() {
			expect(todoController).toBeDefined();
		});

		describe('after getTodos is called', function() {
			it('should have 3 todos', function() {
				expect(todoController.todos.length).toEqual(3);
			});
		});

		describe('after adding a todo', function() {
			it('should have 4 todos in the array', function() {
				todoController.addTodo();

				$rootScope.$apply();

				expect(todoController.todos.length).toEqual(4);
			});
		});

		describe('after removing a todo', function() {
			it('should have 2 todos in the array', function() {
				todoController.removeTodo({});

				$rootScope.$apply();

				expect(todoController.todos.length).toEqual(2);
			});
		});
	});
});
```

*Write todoFactory test*
```js
describe('todoFactory', function() {

	beforeEach(function() {
		bard.appModule('app');
		bard.inject('todoFactory', '$httpBackend', 'apiUrl');
	});

	describe('todo factory', function() {
		describe('get all', function() {
			it('should return data on success', function() {
				$httpBackend.whenGET(apiUrl + '/todos').respond({
					data: [{}]
				});

				todoFactory.getAll().then(
					function(data) {
						expect(data.length).toEqual(1);
					}
				);
			});

			it('should return an error on fail', function() {
				$httpBackend.whenGET(apiUrl + '/todos').respond(500);

				todoFactory.getAll().then(
					function() {
					},
					function(error) {
						expect(1).toBe(2);
					}
				);
			});
		});
	});

});
```
