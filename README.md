# mern-app-boilerplate
create a MERN app from scratch including:
  * continuous integration via Travis CI
  * continous delivery via heroku flow + pipeline (no review ('paid') apps)

Asides: 
  * prior to starting please make and account on github (https://github.com/), travis ci (https://travis-ci.org/) and heroku (https://id.heroku.com/login)
  * I use vs code

## Step 1: create gitup repo
  * go to repositories tab on your Github Profile page and hit the New button (green button on the right)
  * type name of app (app_name will be used throughout this tutorial; replace with the name of your choice), description, and initialize with README.md
 
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
yarn init -y
yarn add express
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
  res.send('yolo')
};

// catch all handler
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname+'/client/build/index.html'));
});

// listen to port
const PROT = process.env.PORT || 8000;
app.listen(PORT, function() {
    console.log(
      "==> 🌎  Listening on port %s. Visit http://localhost:%s/ in your browser.",
      PORT,
      PORT
    );
  });
```


## Resources:
  1) https://daveceddia.com/deploy-react-express-app-heroku/
