# mern-app-boilerplate
create a MERN app from scratch including:
  * continuous integration via Travis CI
  * continous delivery via heroku flow + pipeline (no review ('paid') apps)

Asides: 
  * prior to starting please make and account on github (https://github.com/), travis ci (https://travis-ci.org/) and heroku (https://id.heroku.com/login)
  * I use vs code
  * if servers are in use kill all servers: killall -9 node

## Step 1: create gitup repo
  * go to repositories tab on your Github Profile page and hit the New button (green button on the right)
  * type name of app (app_name will be used throughout this tutorial; replace with the name of your choice), description, and initialize with README.md
  * FYI: if you already have a travis ci account and it is sycned with github, then at the time of initialization, there should be an option to select Travis CI.
 
## Step 2: enable Travis CI
  * click Authorize travis-ci to log in with your GitHub username/password.
  * click on you picture and go to Settings
  * under Repositories search for app_name
  * modify the Settings of your app: under General: check-off build push branches and leave build pushed pull requests checked ( this is my personal preference)
  
## Step 3: setup on MERN app on local machine
### Step 3a: Set up Express server
```shell
git clone https://github.com/<username>/app_name.git
cd app_name
code .
yarn init -y
yarn add express path
touch index.js
code index.js
```
  * add the following the index.js file
```javascript
const express = require('express');
const app = express();

// Serve React build
app.use(express.static(path.join(__dirname, 'client/build')));

// simple API end point
app.get('/api/msg', (req, res) => {
  const msg = 'LEARN REACT';
  res.json(msg)
});

// catch all handler
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname+'/client/build/index.html'));
});

// listen to port
const PORT = process.env.PORT || 8000;
app.listen(PORT, function() {
    console.log(
      "==> 🌎  Listening on port %s. Visit http://localhost:%s/ in your browser.",
      PORT,
      PORT
    );
  });
```

### Step 3b: Set up front-end react app
  * install create-react-app globally if not already installed
```shell
yarn global add create-react-app
```

  * create react app called client (if you change client to something else, remember to also change it in the index.js file above)
```shell
create-react-app client
```

  * go to client/src/index.js and comment out serviceWorker lines (prevents you from accessing API end points after the React app has been load once)
```javascript
// import * as serviceWorker from './serviceWorker';
// serviceWorker.unregister();
```

  * change client/src/App.js to:
```javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  // initialize state
  state = { msg: '' }

  // fetch msg after mounting
  componentDidMount(){
    this.getMsg();
  }

  getMsg = () => {
    fetch('/api/msg')
      .then(res => res.json())
      .then(msg => this.setState( {msg}))
  }

  render() {
    const { msg } = this.state;

    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <p>
            Edit <code>src/App.js</code> and save to reload.
          </p>
          <a
            className="App-link"
            href="https://reactjs.org"
            target="_blank"
            rel="noopener noreferrer"
          >
            { msg }
          </a>
        </header>
      </div>
    );
  }
}

export default App;
```


## Step 4: Test MERN APP
  * first go to app_name/package.json and add a new property (scripts)
```javascript
{
  "name": "app_name",
  .
  .
  .,
  "scripts": {
    "start": "node index.js"
  }
}
```
  * then go to app_name/client/package.json and add a new property (proxy)
```javascript
{
  "name": "client"
  "proxy": "http://localhost:8000"
  .
  .,
  "scripts": {
    "start": "node index.js"
  }
}

 **note**: same port as express server

  * in app_name/ start express server using:
```shell
yarn start
```
  * open a new terminal and go to app_name/client/ window and start react server:
```shell
yarn start
```
#### If working the app should look like the following, but Learn react should be ALL CAPS!!
![alt text](https://d33wubrfki0l68.cloudfront.net/dbdecdbb76e8d2c779ddeda5cbf0be77d077c74a/7f8b4/assets/new-create-react-app.png)

#### commit everything to master
```javascript
git add .
git commit -m "create mern app"
git push origin master
```

#### Test Travis CI 
We will create a feature branch (turn the background of the homepage to yellow), create a PR and see if it passes Travis CI's checks. 
  * create new feature branch
```javascript
git checkout -b feature
git branch
```
  * within the feature branch, go to client/src/App.css and change background-color: #282c34; (line 11) to background-color: black; and save App.css
  * add, commit and push to feature branch
```shell
git status
git add .
git commit -m "change background color"
git push origin feature
```
  * go to the app_name repo page: https://github.com/singha53/app_name --> to to the Pull requests tab ==> click the green button (Compare & pull request). ![alt text](https://www.drupal.org/files/pull_request_test_highlighted.png)


## Resources:
  1) https://daveceddia.com/deploy-react-express-app-heroku/
