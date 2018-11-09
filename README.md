# mern-app-boilerplate
create a MERN app from scratch including:
  * continous delivery via heroku flow + pipeline (no review ('paid') apps)
  * unit testing

Asides: 
  * prior to starting please make and account on github (https://github.com/), and heroku (https://id.heroku.com/login)
  * I use vs code
  * if servers are in use kill all servers: killall -9 node

## Step 0: create gitup repo
  * go to repositories tab on your Github Profile page and hit the New button (green button on the right)
  * type name of app (app_name will be used throughout this tutorial; replace with the name of your choice), description, and initialize with README.md
  
## Step 1: setup on MERN app on local machine
### Step 1a: Set up Express server
```shell
git clone https://github.com/<username>/app_name.git
cd app_name
code .
yarn init -y
yarn add express path
touch .gitignore index.js
code .gitignore
```
  * add node_modules to .gitignore
```shell
node_modules/
```

  * add the following the index.js file
```javascript
code index.js
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

### Step 1b: Set up front-end react app
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
## Step 2: add canary test
  * create test folder in root directory (app_name/) and add a random test file
```shell
yarn add --dev mocha chai
mkdir test
cd test
touch canary.test.js
code canary.test.js
```
  * add the following lines in the canary.test.js file
```shell
const expect = require("chai").expect;

describe("canary test", function() {
  it("should pass this canary test", function() {
    expect(true).to.be.true;
  });
});
```
  * got to app_name/package.json and add a test property in the scripts property
```shell
"scripts": {
    "start": "node index.js",
    "test": "NODE_ENV=test mocha -u tdd --reporter spec --exit"
  }
```
**Note: test property must exist in scripts for travis ci to work**

## Step 3: Test MERN APP
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
```

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
  * always be in the root folder (app_name/) when pushing to git (NOT client/)
```javascript
git add .
git commit -m "create mern app"
git push origin master
```

## Step 4: Heroku flow
### Step 4a: add Heroku script for postbuild
  * got to app_name/package.json and add the following property to the scripts property. this tells heroku to build the client app 
```javascript
"scripts": {
    "start": "node index.js",
    "test": "NODE_ENV=test mocha -u tdd --reporter spec --exit",
    "heroku-postbuild": "cd client && yarn && yarn run build"
  }
```

### Step 4b: create dev branch
```shell
git checkout -b dev
```

### Step 4c: Heroku pipeline
  * create staging and production apps
```shell
heroku create --remote staging-app_name
heroku create app_name
git remote -v
```
  * git remote -v should list the origin (git repo), staginig-app_name (staging app) and heroku (production app)
  * in my case I have: 
```shell
heroku  https://git.heroku.com/merntest0.git (fetch)
heroku  https://git.heroku.com/merntest0.git (push)
origin  https://github.com/singha53/merntest.git (fetch)
origin  https://github.com/singha53/merntest.git (push)
staging-merntest0       https://git.heroku.com/cryptic-fjord-42648.git (fetch)
staging-merntest0       https://git.heroku.com/cryptic-fjord-42648.git (push)
```

  * add the two heroku apps to a pipeline. First add the production app to the pipeline using the production apps name:
```shell
heroku pipelines:create -a merntest0
```
  * this command will prompt you to specific a pipeline name, stage of app (develpment, staging, and production). I typed in merntest0-pipeline for the name, production for the stage. My terminal looks like this:
```shell
Amrits-MacBook-Pro:merntest asingh$ heroku pipelines:create -a merntest0
? Pipeline name merntest0-pipeline
? Stage of merntest0 production
Creating merntest0-pipeline pipeline... done
Adding ⬢ merntest0 to merntest0-pipeline pipeline as production... done
```
  * then add the staging app using its name assigned by heroku (in my case its cryptic-fjord-42648, yours will be different)
```shell
Amrits-MacBook-Pro:merntest asingh$ heroku pipelines:add merntest0-pipeline -a cryptic-fjord-42648
? Stage of cryptic-fjord-42648 staging
Adding ⬢ cryptic-fjord-42648 to merntest0-pipeline pipeline as staging... done
```
  * to confirm you can login in heroku and see the a pipeline (merntest0-pipeline) located with your other apps.

### Step 4d: create new feature and create a PR for review
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
  * go to the app_name repo page: https://github.com/singha53/app_name --> to to the Pull requests tab ==> click the green button (Compare & pull request).
  * make sure base: dev and compare: feature
  * add comment and then click Create pull request (green button)
  * Git then checks for merge conflicts and will let you know if there is a conflict.
  * if none, merge feature branch to dev

### Step 4e: Staging
  * 

## Resources:
  1) https://daveceddia.com/deploy-react-express-app-heroku/
  2) https://developer.okta.com/blog/2018/01/11/two-approaches-to-setting-up-a-mern-stack-app
