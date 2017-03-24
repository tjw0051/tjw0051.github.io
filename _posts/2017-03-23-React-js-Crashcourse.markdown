---
layout:     post
title:      "CrashCourse: React js from start to deployment"
subtitle:   "Up and running with React apps as fast as possible."
date:       2017-03-23 12:30:00
author:     "Tom"
header-img: "img/2017-03-23_cover.jpg"
header-mask: 0.5
catalog: true
tags:
    - Crashcourse
    - Web 
    - React
    - Javascript
---

# Overview

A while ago I wanted to set up a web app with react. I had the idea but no experience using react, and wanted to get up and runnning as fast as possible. Unfortunately I couldn't find a tutorial that got straight to the point, from creating the project to deployment. 

This tutorial will hopefully fill that void to help you create a project, start tinkering, and deploy as fast as possible. This tutorial will be structured with each section about how to achieve a certain task, rather than us spending 30 minutes programming a weather app that you have no interest in. 

Its also worth noting that this will be a static app. What this means is the server will supply the client (web browser) with the files, and all the javascript processing will happen on the client. This won't cover intercommunication between server and client (e.g. dynamically requesting data from a database).

---

# Setting up Node

Before we start we need to install Node. If you've already completed this step -> [Jump to the Next Section](#section_create).

## What is Node

Node.js is described as an 'asynchronous event-driven javascript runtime'. Simply think of it as a literal server, in that serves content to users, such as web pages to a site visitor. Another important component of node (to this project) is npm, the Node Package Manager. This is basically the same as the package manager on Mac or Linux (e.g. apt-get), but it installs node libraries into a node project.

For more info about node, check out their more detailed description: [nodejs.org](https://nodejs.org/en/about/).

## Installing Node

If you don't have node installed it can be downloaded from their official site at: [Here](https://nodejs.org/en/download/)

Alternatively it can be downloaded via package manager: [Here](https://nodejs.org/en/download/package-manager/)

Once installed, open up a terminal window and navigate to where you want to set up the project.

<p id = "section_create"></p>

# Create-React-App

The [Create-React-App](https://github.com/facebookincubator/create-react-app) package is excellent for setting up a simple React App as fast as possible. First we need to install it globally (`-g`) with npm. This allows us to use the library across many projects, so that it doesn't need to be installed for each one.


```bash
npm install -g create-react-app
```

When it finishes installing, we can create the project (replace 'my-app' with the name of your project):

```bash
create-react-app my-app
```

Navigate into the newly created project directory and run the application.

```bash
cd my-app/
npm start
```

This will start the node server running our application in development mode. `npm start` should open the site in a browser window, but if it doesn't, open your web browser and navigate to `http://localhost:3000/`.

You should see something like this:

![Screenshot](http://imgur.com/FXHOytp.png)

At this point the app is done, Create-React-App has taken care of everything for us.

# Deploying the App

Next we'll deploy this app to see how it works. Stop the development build from running (CTRL+C in terminal). while still in the project's directory (/my-app/) enter the following:

```bash
npm run build
```

This has now built our static app to the /build/ directory. To serve the production app up, we need one more library which we'll install globally.

```bash
npm install -g serve
```

The [Serve](https://github.com/zeit/serve) package allows us to serve up static files (our application) to users. This is great for getting up and running with an app quickly, without having to set up something more elaborate (e.g. [nginx](http://nginx.org/en/)).

```bash
serve -s build -p 80
```
Breaking down the above command: we call the 'serve' command, and tell it to run a single-page app (`-s`) from the 'build' directory, and to run on port 80.

## "It won't run on port 80"

![Screenshot](http://imgur.com/UghQRN9.png)

This is likely a permissions problem, as ports below 1024 can only be opened by the root user. You CAN run the app as root, however you're advised NOT TO as you're giving your app root access:

```bash
sudo serve -s build -p 80
```

A better practice is to redirect port 80 to another port (above 1024), where your server can be run as a normal user. Do this on your production server (DigitalOcean, heroku, AWS, etc) as you'll only be running your development build from your home PC (probably?). 

**Linux:**
```bash
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

Remove Forwarding:
```bash
iptables -t nat --line-numbers -n -L
```

[Source](http://serverfault.com/questions/112795/how-to-run-a-server-on-port-80-as-a-normal-user-on-linux)

**Mac:**
```bash
echo "
rdr pass inet proto tcp from any to any port 80 -> 127.0.0.1 port 8080
" | sudo pfctl -ef -
```

Remove Forwarding: 
```bash
sudo pfctl -F all -f /etc/pf.conf
```

[Source](https://salferrarello.com/mac-pfctl-port-forwarding/)

Now we can re-run the server on the redirected port - 8080 in this example:

```bash
sudo serve -s build -p 8080
```

Your server should now be accessible from its hostname/IP address on the http port, without the need to append the port name:

```bash
http://localhost/
```

If you see your web app then it worked!

![Screenshot](http://imgur.com/B2s315A.png)

You've now set up a react web app thats accessible to anyone connecting to your server. You're free to start exploring the files and start tinkering. The next sections will explain the structure of the application and how to achieve some common tasks (e.g. navigating between 'screens', links, events from buttons, input boxes, importing js libraries). Use the links on the right to jump to any section you might need.

# Project Structure

Create-React-App creates the following for us:

```
my-app/
    node_modules/
        ...
    build/
        ...
    public/
        index.html
        favicon.ico
    src/
        App.css
        App.js
        App.test.js
        index.css
        index.js
        logo.svg
    README.md
    package.json
```

The package.json defines our whole app package in node. If you don't know what this is check out the official npm description: [Here](https://docs.npmjs.com/files/package.json) and this excellent interactive guide to the package.json file: [Here](http://browsenpm.org/package.json).

The entry-point to our application is the `my-app/public/index.html` file. Open this file and you'll find:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico">
    <!--
      ...
    -->
    <title>React App</title>
  </head>
  <body>
    <div id="root"></div>
    <!--
      ...
    -->
  </body>
</html>
```

The key here is the `<div id="root">` in the `<body>` tags. This is where the react app attaches and renders its content. To see this, open the `my-app/src/index.js` file:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import './index.css';

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

This is our first introduction to react. The first two lines import the `react` and `react-dom` packages. The third line imports the App component from `my-app/src/App.js`, and the 4th simply imports the index css file.

After importing, the script renders the `<App />` component, imported above, into the `root` element we saw earlier in the `index.html`. To see what that `<App />` component is, open the `my-app/src/App.js` file.

```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  render() {
    return (
      <div className="App">
        <div className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h2>Welcome to React</h2>
        </div>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>
      </div>
    );
  }
}
export default App;
```

The first line imports the `Component` class from React, as we want to make a component. The second line imports an image we want to render, and the third line imports the stylesheet to be applied to the HTML rendered by this component.

Next we create a class called `App` that extends the `Component` class. This class has 1 method: `render()`. This method returns the HTML we want to render when we used the `<App />` tag in `index.js`.

Finally, we export the react class with the line `export default App;`, so that it can be imported into `index.js`.

This class also shows how a variable can be used in the rendered HTML. The logo that was imported is referenced in the img tag at `<img src={logo} ...>`.

# Class Methods

We can add additional methods to the App component like so:

```javascript
class App extends Component {
  methodA() {
    this.methodB();
  }

  methodB() {
  }
```

React also features some useful lifecycle methods. We've already seen the `render()` method. Some additional methods are shown below:

```javascript
class App extends Component {
  constructor() {
    super();
    // Initialize component here
  }
  componentWillMount() {
    // Called before the component is mounted
  }
  componentDidMount() {
    // Called after the component is mounted
  }
  componentWillUnmount() {
    // Called before the component is unmounted
  }

```

These are only some of the `Component` methods. The full API can be found [here.](https://facebook.github.io/react/docs/react-component.html)

# Events (buttons, input box)

![Screenshot](http://imgur.com/7W5CBcI.png)

We'll extend our application by adding a small form to the html consisting of 1 input box and 1 input button:

```javascript
class App extends Component {
  render() {
    return (
      <div className="App">
        <div className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h2>Welcome to React</h2>
        </div>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>

        <form onSubmit={this.onSubmitForm}>
          <input type="text" value={this.state.value} onChange={this.onChangeInput} />
          <input type="submit" value="Search" />
          <h2>{this.state.submittedValue}</h2>
        </form>

      </div>
    );
  }
}
export default App;
```

Here we create the form, and add a callback to the `onSubmit` form (we'll create this method later). The next input derives its value from the `this.state.value` property, and also has an `onChange` callback - `this.onChangeCallback`. For our form button we add another input to the form. At the end of the form, we create a `<h2>` header to display the value of our form when we hit submit. The value of this header is derived from `this.state.submittedValue`.

Lets create the methods for these callbacks:

```javascript
class App extends Component {
  constructor() {
    super(); // MUST call superclass constructor

    // Create properties for 'App' state.
    this.state = {
      value: '',            // Value displayed in input box
      submittedValue: ''    // Value we set in <h2>
    }

    // Bind events
    this.onChangeInput = this.onChangeInput.bind(this);
    this.onSubmitForm = this.onSubmitForm.bind(this);
  }

  // Called when user types in input field
  onChangeInput(event) {
    this.setState({value: event.target.value});
  }

  // Called when form is submitted
  onSubmitForm(event) {
    this.setState({submittedValue: this.state.value});
    event.preventDefault(); // Prevent default form submit behaviour
  }
```

In the constructor we first create 2 properties in the component's state - `value` and `submittedValue`. We then bind the callbacks to `this` class, to ensure that 'this' refers to the correct instance (this is the this=that/self quirk for those familiar with js callbacks).

The **onChangeInput** event is called when the user types into the input field. We set the state of the 'value' property to whatever the user inputs, which is then displayed in the 'input' field on the webpage. If you don't add this, then nothing will be displayed in the field when the user types. 

The **onSubmitForm** event is triggered when the user click the 'submit' button or presses enter. We call set state and set the 'submittedValue' to the value of the input field, and prevent the default form behaviour. 

Its important to note that in order for the UI to be updated, the state must be changed using the `setState` method. 

Re-run the development build with `npm start` and type into the input field. Press submit and the value from the field should be displayed below in the `<h2>` header.

![Screenshot](http://imgur.com/EsLxskT.png)

# Multiple Screens/Pages

What if we want to navigate to a new screen? With react, we can conditionally render different components. If we take our 'App' component as our first screen, we can create a second component to navigate to, when the user presses submit.

This will also demonstrate passing data between components.

We'll take the `App` class from the previous section and rename it to `HomeComponent`. Alternatively, copy and paste the below code into a new file called `HomeComponent.js:

```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class HomeComponent extends Component {

  constructor(props) {
    super(props);

    this.state = {
      value: '',
      submittedValue: ''
    }

    this.onChangeInput = this.onChangeInput.bind(this);
    this.onSubmitForm = this.onSubmitForm.bind(this);
  }

  onChangeInput(event) {
    this.setState({value: event.target.value});
  }

  onSubmitForm(event) {
    event.preventDefault();
    this.setState({submittedValue: this.state.value});
    //this.props.setScreenFunc(2, this.state.value);
    
  }

  render() {
    return (
      <div className="App">
        <div className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h2>Welcome to React</h2>
        </div>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>
        <form onSubmit={this.onSubmitForm}>
          <input type="text" value={this.state.value} onChange={this.onChangeInput} />
          <input type="submit" value="Search" />
          <h2>{this.state.submittedValue}</h2>
        </form>
        
      </div>
    );
  }
}

export default HomeComponent;
```



Next we need to re-create our `App.js`, and also create a new file called `ResultComponent.js`. Your src folder should now contain:

```
my-app/
    src/
        App.css
        App.test.js
        index.css
        index.js
        logo.svg

        App.js
        HomeComponent.js
        ResultComponent.js
```

Open `App.js` and add the following:

```javascript

import React, { Component } from 'react';
import './App.css';
// Import our newly created classes
import HomeComponent from './HomeComponent';
import ResultComponent from './ResultComponent';

class App extends Component {

  constructor() {
    super();
    // Store the current screen and value from HomeComponent
    this.state = {
      screen: 1,
      submittedValue: ''
    };
  }
  // Called by HomeComponent to change screen
  setScreen = (index, value) => {
    this.setState({
      submittedValue: value,
      screen: index
    });
  }

  render() {
    // screen 1 = HomeComponent, screen 2 = ResultComponent
    switch(this.state.screen) {
      case 1:
        return <HomeComponent setScreenFunc={this.setScreen}/>;
      case 2:
        return <ResultComponent submittedValue={this.state.submittedValue}/>;
    }
  }
}
export default App;
```
Firstly we store the current `screen` and `submittedValue` in the constructor. We then create `setScreen = (index, value)` which can be called from our HomeComponent. For why this looks different, see [Property Initializer Syntax](https://facebook.github.io/react/docs/handling-events.html). For simplicity we can think of it as a method. 

Finally in the render method, we choose which screen to render using a switch statement, which conditionally renders either the `HomeComponent` or `ResultComponent`. We also pass properties to each of these components. The `setScreenFunc` property of HomeComponent is given the value of our `setScreen` 'method', so that this component can call this method from its own component. 

```javascript
<HomeComponent setScreenFunc={this.setScreen}/>
```

We also set the value of the `submittedValue` property of the ResultComponent to `this.state.submittedValue`, which is set when HomeComponent calls `setScreen`. 

```javascript
<ResultComponent submittedValue={this.state.submittedValue}/>
```

So how do we access these components from the classes?

Open `HomeComponent`. Our constructor should also look like this:

```
constructor(props) {
    super(props);

    this.state = {
      value: '',
      submittedValue: ''
    }

    this.onChangeInput = this.onChangeInput.bind(this);
    this.onSubmitForm = this.onSubmitForm.bind(this);
  }
```

The class receives the properties that we set in the `App.js` class as the `props` property in the constructor. 

Find the `onSubmitForm(event)` method. Uncomment the `this.props...`line, so that the method now looks like this:

```javascript
onSubmitForm(event) {
    event.preventDefault();
    this.setState({submittedValue: this.state.value});
    this.props.setScreenFunc(2, this.state.value);
  }
```

Here we call the `this.props.setScreenFunc` property we created, passing it the screen we want: `2` and the value of the input box: `this.state.value`.

Open the `ResultComponent.js` file and add the following:

```javascript
import React, { Component } from 'react';
import './App.css';

class ResultComponent extends Component {

  constructor(props) {
    super(props);
    this.state = {
      result: props.submittedValue
    };
  }

  render() {
    return (
      <h2>{this.state.result}</h2>
      );
  }
}
export default ResultComponent;
```

Again we take the properties from the constructor, and this time we set the state's property `result` to `props.submittedValue`.

Finally, we render the value of `this.state.result` in the render method.

Re-run the application and type something into the input field. Press enter and the new ResultComponent screen should be rendered with your input text:

![Screenshot](http://imgur.com/BFVGAZz.png)

---

# Resources

Hopefully this has established a good starting-point from which to explore more of what react has to offer. 

A great example application that was helpful to me is [React-Living-App](https://github.com/djirdehh/react-living-app) created by [Hassan Djirdeh on Github](https://github.com/djirdehh). 


---

# Download - Github

<p id = "section_download"></p>

The full source code for the project can be found on Github at:

[https://github.com/codebasecampprojects/React-Tutorial](https://github.com/codebasecampprojects/React-Tutorial)


—— Tom 2017.03
