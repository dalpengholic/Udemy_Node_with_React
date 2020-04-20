### 11. Express Route Handlers
```
app.get('/', (req, res) =>{
    res.send({hi:'there'});
});
```
- app --> (object) creating new route handler
- get: Watch for incoming requesets wiht this method
- '/': Watch for requests trying to access '/'
- req: Object representing the incoming request
- res: Object representing the outgoing response
- res.send({hi:'there}): Immediately send some JSON back to who ever made this request

```
app.listen(5000);
```
- express telling node to listen to 5000

### 12. Heroku Deployment Checklist
#### Deployment Checklist
- Dynamic Port Binding: Heroku tell us which port our app will use, so we need to make sure we listen to the port they tell us to
`const PORT = process.env.PORT`
- Inject environment variables, underlying runtime that node is running on top of.
- Look at the underlying environment and see if they have declared a port for us to use.
#### Specify Node Environment: 
```
in package.json
  "engines": {
    "node": "8.1.1",
    "npm": "5.0.3"
```
#### Specify start script
```
in package.json instead of test in script
    "start": "node index.js"
```
#### Create .gitignore file
```
don't commit any dependcies. We let Heroku install all of our dependecies itself
```

### 14. Verify Heroku Deployment
- `heroku login`
- `heroku create`
- `first link is address to access using browser`, `second link is our deployment target`
- `git remote add <target>`
- `git push heroku master`
- `heroku open`

### 19. Passport Setup
- `const passport = require('passport');` 
- Giving express the idea of how to handle authentication
- `const GoogleStrategy = require('passport-google-oauth20').Strategy;`
- Instruct passport on exactly how to authenticate our users
- `passport.use(new GoogleStrategy());`
- new GoogleStrategy creates a new instance of the Google passport strategy

### 23. Securintg API Keys
- Make config folder and create keys.js inside
- Save Credentials inside of keys.js
```
module.exports ={
  ID: '<credential id>'
  Secret: '<credential secret>'
}
```
- Update gitignore not to upload keys.js

### 24. Google Strategy Options
- `const keys = require('./config/keys')` in index.js
- create an object
- user click login button --> direct to localhost:5000/auth/google --> Forward request to google's server (something like
google.com/auth?appid=123 --> ask user if they grand permission --> User grants permission --> direct to localhost:5000/auth/google/callbakc?code=456 --> Put user on hold, take the 'code' from the URL
```
passport.use(new GoogleStrategy({
    clientID: keys.googleClientID,
    clientSecret: keys.googleClientSecret,
    callbackURL: '/auth/google/callback'
    }, (accessToken) => {
        console.log(accessToken);
    })
);
```
- callbackURL: user sent after consent granting permission

### 25. Testing OAuth
```
app.get('/auth/google', passport.authenticate('google',{
    scope: ['profile', 'email']
})
);
```
- app: expresss app object
- get: get type HTTP request
- `/auth/google`: firt argument, path we want to handle
- `passport.authenticate('google',{scope: ['profile', 'email']})`: second argument, some code to be executed whenever 
a request comes into this route. Tell express to involve passport
- `('google',{scope: ['profile', 'email']})`: first argument, 'google' --> this string interner identifier
- `scope: ['profile', 'email']`: second argument, options object passed in scope. The scope specifies to GOogle like the actual Google servers what access we want to have inside of this user's profile.
- In other words, we are asking Google to give use access to this user's profile information and their email address as well.

### 26. Authorized Redirect URI's
- `https://accounts.google.com/o/oauth2/v2/auth?`: base url
- `response_type=code&`: query string, response that code back
- `redirect_uri=http%3A%2F%2Flocalhost%3A5000%2Fauth%2Fgoogle%2Fcallback&`: 
- `scope=profile%20email&`: we ask profile and email
- `client_id=658173462832-g6grcsnj52rbmqv6pp2d1nmjo0adrjti.apps.googleusercontent.com`: identifies our app to google's servers
```
The redirect URI in the request, http://localhost:5000/auth/google/callback, does not match the ones authorized for the OAuth client. To update the authorized redirect URIs, visit: https://console.developers.google.com/apis/credentials/oauthclient/658173462832-g6grcsnj52rbmqv6pp2d1nmjo0adrjti.apps.googleusercontent.com?project=658173462832
```
- Access `https://console.developers.google.com/apis/credentials/oauthclient/658173462832-g6grcsnj52rbmqv6pp2d1nmjo0adrjti.apps.googleusercontent.com?project=658173462832` --> update `Authorised redirect URIs`--> `http://localhost:5000/auth/google/callback`

### 27. OAuth Callback
- `app.get('/auth/google/callback', passport.authenticate('google'));`

### 28. Access and Refresh Tokens
- 

### 29. Nodemon Setup
- Setup
- `npm install --save nodemon`: Run node index.js automatically
- Open package.json 
- `"dev": "nodemon index.js"` inside package.json
- `npm run dev`: run nodemon

## Section4: Adding MongoDB
### 30. Server Structure Refactor
- Refactoring
- Create `routes`, `servcies` folder
- Break index.js into several pieces
- Inside `routes`, create `autoRoute.js` and cut and paste below
```
const passport = require('passport');
module.exports = (app) => {

    app.get('/auth/google', passport.authenticate('google',{
        scope: ['profile', 'email']
    })
    );

    app.get('/auth/google/callback', passport.authenticate('google'));
};
```
- Create arrow function 

- Inside `services`, create `passport.js` and cut and paste below
```
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;
const keys = require('../config/keys')

passport.use(new GoogleStrategy({
    clientID: keys.googleClientID,
    clientSecret: keys.googleClientSecret,
    callbackURL: '/auth/google/callback'
    }, (accessToken, refreshToken, profile, done) => {
        console.log('access token', accessToken);
        console.log('refresh token', refreshToken);
        console.log('profile', profile);
    })
);

```

- Edit `index.js`
```
const express = require('express');
require('./services/passport');

const app = express();
require('./routes/authRoutes')(app); 

const PORT = process.env.PORT || 5000;
app.listen(PORT);

```
- `require('./routes/authRoutes')(app)`: When we require the authRoutes file, it returns a function. That's what we returned or thats what we exported from that file. Returns a function with an immediately call that fucntion with the app object. So the second set of parentheses immediately invokes or immediately callst the function that we just required in. Tha app is passed into that arrow function 

### 31. The Theory of Authentication
- HTTP is Stateless
- By default, information by bewteen two requests is not shared
- So we can't really identify who is making any given request between any given number of requests
- How to solve it?
- Log in --> server gives a unique identifying piece of info --> cookie, token is given whatever
- Any follow up request that our browser ever makes to the server, we are going to include that token(cookie) that proves that we ar the same person who had made that original log in request like five migutes ago or one day ago.
- `cookie based authentication`
- cookie is given by server as a header to the browser
- The browser is going to automatically strip off this token. It is going to store it into the browser's memory and then the browser is going to automatically append that cookie with andy follow up request being sent to the server.

### 32. Signing in Users with OAuth
- `Email and Password Flow`: Sign up(getting email/password) --> Sign out --> Login
- The two email/password combinations are the same? Great, must be the same person.
- `OAuth Flow`: Sign up(getting google id out of profile) --> Sign out --> Login
- We need to find some unique indentifying token in the user's Google profile. Is that consistent between logins?
- Use that to decide if the user is the same

### 36. Connecting Mongoose to MongoDB
- Add mongoURI in keys.js in config
- Add `const keys = require('./config/keys')` in index.js
- Add `mongoose.connect('keys.mongoURI');` in index.js

### 37. Breather and Review
- Mongo Installed, Mongoose Installed 
- Need to be able to identify users who sign up and return to our application. We want to save the 'id' in there google profile.
- Use Mongoose to create a new collection in Mongo called 'users'
- When user signs in, save new record to the 'users' collection

### 38. Mongoose Model Classes
- Create `models` folder, create `User.js` inside
- Input below in `User.js`
```
const mongoose = require('mongoose');
const { Schema } = mongoose; ## == const Schema = mongoose.Schema;

const userSchema = new Schema({
    googleId: Strings
});

mongoose.model('users', userSchema);
```
- `const { Schema } = mongoose;` : mongoose has to know all properties. The schema will describe what every individual property personally every individual record is going to look like.
- `const userSchema = new Schema` : So I'm going to say Konst user schema equals new schema. And then we're going to pass it an object.  This object right here is going to describe all the different properties we have.
- `mongoose.model('users', userSchema);`: name of the collection 'users', user schema 'userSchema'

### 39. Saving Model Instances
- However for everything that uses mongoose model classes like what we just created inside of year we are not going to use require statements and there's a very good reason for that.
- In `passport.js`
- `const mongoose = require('mongoose');`
- so back inside of users, we created a user schema andwe loaded it into mongoose by saying model and their first argument of users.
- So one argument(const User = mongoose.model('users');) means we are trying to fetch something out of mongoose. Two arguments(mongoose.model('users', userSchema);) means we're trying to load something into it.
- `const User = mongoose.model('users');`: User object is model class
- `new User({ googleId: profile.id }).save();` in passport.js: create new User with googleID and save it
- Switch order require in index.js
```
require('./models/User');
require('./services/passport');
```
- However we first are running that passport file where we tried to pull the model out of mongoose and then only after that do it executes or require in the user model class file which is where we actually define the model class.
- In other words we are seeing that error message because we are attempting to make use of the user model before we have actually defined it. So to solve that issue all we have to do is change the order of these two require statements.

### 41. Mongoose Queries
- `User.findOne({ googleId: profile.id })`: Initiate a query over all the records inside the collection to do so
- `{ googleId: profile.id }`: Search cliteria googleId equal to profile.id
- `User.findOne({ googleId: profile.id })`: Instead the query returns a promise a promise is a tool that we use with javascript for handling asynchronous code
- `then((existingUser)`: So this(`existingUser`) will be a model instance that represents a user who was found

### 42. Passport Callbacks
- After creation or found a user, call 'done' 
- In passport.js
```
new User({ googleId: profile.id })
.save()
.then(user => done(null, user));
```
- `new User({ googleId: profile.id })`: a new mongoose model instance, single recorde of inside of our collection
- `.save()`: So this creates a new model right here a new model instance we then save that instance 
- `.then(user => done(null, user));`: and in the callback, we get another model instance.


### 43.
- 
```
passport.serializeUser((user, done) => {
    done(null, user.id);
});
```
- `done(null, user.id);` : user.id is the id created and stored by and in MongoDB
- After sign in, using user id in MongoDB


### 44. Deserialize User

## Section 6. Moving to the Client Side
### 59. A Separate Front End Server
- The Express server pulls some information out of Mongo D.B and then it can respond to requests that the browser makes with some amount of Jaison or something that says like hey here's your user user model or here's how you go through the Auth. flow and all that kind of good stuff. So the Express server is really solely concerned with generating Jason data and nothing else that serves data. It doesn't do a darn thing besides that.
- Now the re-act side of our application however this re-act server is going to eventually take a bunch of different component files. So these are all you can imagine being re-act components right here. It's going to take them all together it's going to bundle them all together using web pack and babbel and then it's going to spit out a single bundle that G-S file that will be loaded up into the browser
- Why 2 servers?
- So the reason that we are using two separate servers here is one just very simple one very straightforward and simple reason and that is the fact that create re-act app is bar none the best way to get started with building re-act applications create re-act out has so much pre-built configuration already placed into it that it's going to save us so much time in trying to wire together web pack and babbel and the development server. You know all this bundling stuff and all these other dependencies all the CSSA handling all these other well-packed plug ins all this stuff we get just out of the box for free with create re-act out. And so just using create react to operate here saves us a tremendous amount of time and is worth this extra development or this extra time it takes to figure out exactly how to make the two of them work together nicely. So at the end of the day the reason we're using the separate re-act server is that using Briac create re-act out is absolutely worth it and it gives us a ton of awesome functionality right out of the box.

### 64. Why This Architecture?
- Cookies
- CORS request
- 

