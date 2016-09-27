# MongoDB Demo

This demo shows students how to build MEAN applications from scratch, without using the yeoman generator. 

## High level steps
- [ ] Supporting Tools
- [ ] MongoDB Introduction
- [ ] Project Setup


## Supporting Tools
- [ ] MongoDB
	- [ ] Install on OSX
		- [ ] Grab homebrew
		- [ ] `brew update`
		- [ ] `brew install mongodb`
	- [ ] Install on Windows
		- [ ] Grab chocolatey
		- [ ] `choco install mongodb`

## MongoDB

### Explanations
```
MongoDB Server
	MongoDB Database
		MongoDB Collection (Table)
			MongoDB Documents (Row)
				MongoDB Properties (Column)
```

### Commands

`show dbs` - talk about databases

`use origincodeacademy` - this points to a database. (origincodeacademy doesn't exist yet, but it will when we start adding things to it. everything in MongoDB is lazy loaded. If a collection doesn't exist - it'll be created when you add something to it.

`db.students.add({})` - create a record

`db.students.find({})` - gets all records (talk about primary key)

`db.students.add({})*3` - make a few records

`db.students.find({ name: 'X' })` - demo querying using **filters**

`help` - shows commands you can use

`db.help` - shows commands for working with dbs

`db.mycoll.help()` - shows commands for working with collections

`db.students.deleteOne(<filter>)` - demo deleting using **filters**

`db.students.update({ name: 'X' }, { name: 'Y' })` - demo updating using **filters**

`db.students.count()` - counts how many records in the database

### Relationships

**One to many**<br />
- A post on a social network has many comments. These comments are always unique to that post.

`use facebook`<br />
`db.posts.insert({ text: 'Hello world!', comments: [{ text: 'So insightful' }, { text: 'So wow' }, { text: 'I really like this, it is so true }] })`

- A customer has an array of phone numbers. These will always be unique to that customer.

`use salesforce`<br />
`db.contacts.insert({ name: 'Cameron Wilby', phones: ['9092247557', '9098108106'] })`

**Many to many**<br />
- A user likes many movies on facebook. That movie can be liked by many users.

`use facebook`<br />
`db.movies.insert({ name: 'Shawshank Redemption' })`<br />
`db.movies.find()` - grab the id of the movie
`db.users.insert({ name: 'Cameron Wilby', movies: ['<id>'] })`<br />
`db.users.insert({ name: 'Jason Land', movies: ['<id>'] })`<br />

to find out who liked the movie

`db.users.find({ likedMovies: '' })`

- A project is assigned to multiple students. That student can be assigned multiple projects.

`use classroom`<br />
`db.projects.insert({ name: 'AstroWeight Calculator' })`
`db.projects.find()` - grab the id of the project
`db.students.insert({ name: 'Cameron Wilby', assignments: [{ project: '<projectid>', duedate: '2 days'}] })`

### RoboMongo
An awesome visual tool for MongoDB databases