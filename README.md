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

