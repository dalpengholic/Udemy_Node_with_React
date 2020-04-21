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
### 65. Async/Await Syntax
```
Refactoring
// function fetchAlbums() {
//     fetch('https://rallycoding.herokuapp.com/api/music_albums')
//     .then(res => res.json())
//     .then(json => console.log(json));
// }
// fetchAlbums();

async function fetchAlbums() {
    const res = await fetch('https://rallycoding.herokuapp.com/api/music_albums')
    const json = await res.json()
    console.log(json)
}
```

- Whenever we make a request with fetch fetch returns a promise.
- That promise is resolved with an object that represents the underline request.Now remember any time we work with a promise to get kind of a little notification or a little callback that the promise has been resolved we'd chain on a dot then statement like so this then will be called with the request coming from the original fecche call right here. So we usually receive that as an argument called rez short response and then to read the actual Jaison data that we retrieved from this API from this route right here. We call rez Jaison like so and make sure you get the set of parentheses.
- So at this point in time we have essentially made some type of asynchronous request that returns a promise when the request is successful. We can change or we can change it then and pass the function.
- That function will be called if the request is successfully resolved so that then is essentially our opportunity to get a little callback something that says hey responses back here you can now work with it.

- The new syntax just in case I didn't actually mention it is called async/await it is specifically for handling a synchronous code inside of any type of function.
- First we identify the function inside of our code base that contains some type of asynchronous requests or some type of asynchronous operation.
- Step one is to put the async keyword in front of that function declaration.So I'm going to say async space function this essentially tells our javascript interpreter that the function that we are about to declare contains some asynchronous code.
- Now step two is to identify all the different promises that are created within that function. In this case we have two different promises.
- We have the one that is being created by calling fetch right here. And then we have the second promise that is being created by calling redstart Jaison in front of each of those promises we're going to add the await keyword.
- finally I'm going to assign the resolve value of fetch and juice on to some intermediate values.
- Just looking at these two functions I think it's very easy to say this is easier to understand as far as like kind of what it's doing we're essentially making request assigned to this very well calling Jaison or the Jason function on that variable assigning the result to a song and then we cancel log out the response.
- The interpreter is still going to make the Fettes request right here. And then it's not going to execute any other code inside the function. So we're still going to make that request right here and then essentially the interpreter is going to
go off and do some other work. It's not going to immediately just like pause an execution right here. It's not going to actually. Oh wait we are not like sleeping or anything like that. Everything is still working the way that it was before.
- Now the last thing I want to say here is that we are not only restricted to using the async await syntax with functions that are declared using the function keyword. We can also use arrow functions if we choose.

- We would get a little bit of an error because I'm using the const key word here and whenever we use const word. remember we can only declare the variable once and so if we were out right now we would

### 68. Client React Setup
- `index.js`: We can think of this indexed file as putting together all of the very initial data layer considerations of our application which for example case in point is going to be the redux side of things. So index.js is really going to be all about redux inside of your I'll basically just say redux.
- `app.js`: Directly underneath the index file, we will make a single component called app jaywalks the update just file is going to be essentially concerned with the rendering layer or the re-act layer of our application. And so this is going to be the primary location for setting up all of our re-act router related logic
- so we can essentially imagine index.js just for redux stuff, app.js for redux stuff.
- 
