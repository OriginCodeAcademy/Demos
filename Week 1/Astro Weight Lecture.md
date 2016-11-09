# Astro Weight 

- [ ] Show the finished product

- [ ] Install reload, to setup a local development server.
- [ ] Install package manager for Sublime
	- [ ] Install Prettify
- [ ] Create a dev folder
- [ ] Create an index.html file
	- [ ] input#inputWeight
	- [ ] select#selectPlanet
	- [ ] button#calculateWeight
	- [ ] p#output
- [ ] Create an index.js file
	- [ ] Add planets array

	```js
	var planets = [
	  { name: 'Sun', gravity: 27.9 },
	  { name: 'Mercury', gravity: 0.377 },
	  { name: 'Venus', gravity: 0.9032 },
	  { name: 'Earth', gravity: 1 },
	  { name: 'Moon', gravity: 0.1655 },
	  { name: 'Mars', gravity: 0.3895 },
	  { name: 'Jupiter', gravity: 2.640 },
	  { name: 'Saturn', gravity: 1.139 },
	  { name: 'Uranus', gravity: 0.917 },
	  { name: 'Neptune', gravity: 1.148 },
	  { name: 'Pluto', gravity: 0.06 }
	];
	``` 
	- [ ] Create a function to populate the select dropdown through the DOM
	
```js
// Dynamically generate the select element
for (var i = 0; i < planets.length; i++) {
  var planet = planets[i];
	
  var opt = document.createElement('option');
  opt.value = planet.gravity;
  opt.innerHTML = planet.name;
	
  document.getElementById('selectPlanet').appendChild(opt);
}	
```

	- [ ] Bind a function to the click event.

```js
// Bind the button click event
document.getElementById('calculateWeight').onclick = function() {
  var selectEl = document.getElementById('selectPlanet');

  var weight = document.getElementById('inputWeight').value;

  var planet = {
    name: selectEl.options[selectEl.selectedIndex].text,
    multiplier: selectEl.value
  };

  var message = 'If you were on ' + planet.name + ' you would weigh ' + Math.round(weight * planet.multiplier) + 'lbs!';

  document.getElementById('output').innerHTML = message;
};
```