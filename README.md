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
