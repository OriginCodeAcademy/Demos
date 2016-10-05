# React

This document serves as a place for my notes learning React.

`React is a javascript library for building user interfaces`<br />
`React is really small, and it does one thing and one thing really well - the view`<br />
`React is just the View in MVC`


### A ReactJS Component

A react component takes in a `State` and some `Properties` as input, and outputs HTML.

```js
var Component = React.createClass({
	render: function() {
		return ()
	}
});
```

A component can change its `State`<br />
A component cannot change its `Properties`

## First Demo

**script.jsx**

```jsx
var Button = React.createClass({
  localHandleClick: function() {
    this.props.localHandleClick(this.props.increment);
  },
  render: function() {
    return (
      <button onClick={this.localHandleClick}>+{this.props.increment}</button>  
    )
  }
});

var Result = React.createClass({
  render: function() {
    return (
     <div>{this.props.localCounter}</div> 
    )
  }
});

var Main = React.createClass({
  getInitialState: function() {
    return {counter: 0};
  },
  handleClick: function(increment) {
    this.setState({ counter: this.state.counter+increment });
  },
  render: function() {
    return (
     <div>
      <Button localHandleClick={this.handleClick} increment={1} />
      <Button localHandleClick={this.handleClick} increment={5} />
      <Button localHandleClick={this.handleClick} increment={10} />
      <Button localHandleClick={this.handleClick} increment={100} />
      <Result localCounter={this.state.counter}/>
     </div>
    )
  }
});

ReactDOM.render(<Main />, document.getElementById("root"));
```

**index.html**

```html
<!DOCTYPE html>
<html>

  <head>
    <link rel="stylesheet" href="style.css">
    <script src="script.js"></script>
  </head>

  <body>
    <div id="root"></div>
    
    <script src="https://unpkg.com/react@15.3.2/dist/react.min.js"></script>
    <script src="https://unpkg.com/react-dom@15.3.2/dist/react-dom.min.js"></script>
    <script src="script.js"></script>
  </body>

</html>
```

## Second Demo

`Building a GitHub fetcher`

**script.jsx**

```jsx
var Card = React.createClass({
  getInitialState: function() {
    return {};
  },
  componentDidMount: function() {
    var component = this;
    $.getJSON("https://api.github.com/users/" + this.props.login, function(user) {
      component.setState(user);
    });
  },
  render: function() {
    return (
      <div>
        <img src={this.state.avatar_url} width="80" />
        <h3>{this.state.name}</h3>
        <hr />
      </div>
    )
  }
});

var Form = React.createClass({
  handleSubmit: function(e) {
    e.preventDefault();
    var loginInput = ReactDOM.findDOMNode(this.refs.login);
    this.props.addCard(loginInput.value);
    loginInput.value = '';
  },
  render: function() {
    return (
      <form onSubmit={this.handleSubmit}>
        <input placeholder="Enter a GitHub username" ref="login" />
        <button>Add</button>
      </form>
    )
  }
})

var Main = React.createClass({
  getInitialState: function() {
    return {logins: []}
  },
  addCard: function(login) {
    this.setState({ logins: this.state.logins.concat(login) });
  },
  render: function() {
    var cards = this.state.logins.map(function(login) {
      return (<Card login={login} />);
    });
    return (
      <div>
        <Form addCard={this.addCard} />
        {cards}
      </div>
    )
  }
});

ReactDOM.render(<Main />, document.getElementById("root"));
```

**index.html**

```html
<!DOCTYPE html>
<html>

  <head>
    <script data-require="jquery@*" data-semver="3.0.0" src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.0.0/jquery.js"></script>
    <link rel="stylesheet" href="style.css" />
  </head>

  <body>
    <div id="root"></div>
    <script src="https://unpkg.com/react@15.3.2/dist/react.min.js"></script>
    <script src="https://unpkg.com/react-dom@15.3.2/dist/react-dom.min.js"></script>
    <script src="script.js"></script>
  </body>

</html>
```