# Iteration #2: Log In

Now that we’ve registered, we need to be able to log in. The route for the log in page will /login. Since it’s related to authentication, we want to put that in our routes/auth.js file. On our lines 64-68 you can see our log in route:

```js
// ... inside of routes/auth.js
      res.redirect('/');
    });
  });
});


router.get('/login', (req, res, next) => {
  res.render('auth/login', {
    errorMessage: ''
  });
});


module.exports = router;
```

The /login route renders the views/auth/login.hbs view. Notice again that we have an errorMessage local variable. We will use to give the user feedback after they submit the form. When we first visit the login page, errorMessage will be empty.

The /login route contains a form that a user will submit to authenticate with the app. In other words, the form will submit to a POST route that will require us to create a session for that user. Next step for us: configure sessions.

Install express-session and connect-mongo in the terminal:
```
$ npm install --save  express-session  connect-mongo
```
Now we will require both express-session and connect-mongo on the app.js:
```
// ... inside of app.js
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const session = require('express-session');
const MongoStore = require('connect-mongo')(session);

const index = require('./routes/index');
// ...
```
Then we configure the session and add it as a middleware to our app. See on the app.js:
```js
// ... inside of app.js
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));

app.use(session({
  secret: 'never do your own laundry again',
  resave: true,
  saveUninitialized: true,
  cookie: { maxAge: 60000 },
  store: new MongoStore({
    mongooseConnection: mongoose.connection,
    ttl: 24 * 60 * 60 // 1 day
  })
}));

app.use('/', index);
// ...
```
Notice that we’ve changed the secret option. Every app should have it’s own custom secret.

Now we can finish our POST route that will receive the submission of the Log In form. Check the updates to our routes/auth.js file:
```js
// ... inside of routes/auth.js
router.get('/login', (req, res, next) => {
  res.render('auth/login', {
    errorMessage: ''
  });
});

router.post('/login', (req, res, next) => {
  const emailInput = req.body.email;
  const passwordInput = req.body.password;

  if (emailInput === '' || passwordInput === '') {
    res.render('auth/login', {
      errorMessage: 'Enter both email and password to log in.'
    });
    return;
  }

  User.findOne({ email: emailInput }, (err, theUser) => {
    if (err || theUser === null) {
      res.render('auth/login', {
        errorMessage: `There isn't an account with email ${emailInput}.`
      });
      return;
    }

    if (!bcrypt.compareSync(passwordInput, theUser.password)) {
      res.render('auth/login', {
        errorMessage: 'Invalid password.'
      });
      return;
    }

    req.session.currentUser = theUser;
    res.redirect('/');
  });
});


module.exports = router;
```
Highlights of this POST route:

    Line 75: Find the user by their email.
    Line 83: Use the compareSync() method to verify the password.
    Line 90: If everything works, save the user’s information in req.session.

So we’ve logged in, but you wouldn’t know it by just looking at the home page. It looks the same as before! We need to customize the home page for logged in users. Before we do that though, let’s make it a easier to check the logged in status in the view.

Add our custom middleware to lines 66-75 in app.js:
```js
// ... inside of app.js
app.use(express.static(path.join(__dirname, 'public')));

app.use(session({
  secret: 'never do your own laundry again',
  resave: true,
  saveUninitialized: true,
  cookie: { maxAge: 60000 },
  store: new MongoStore({
    mongooseConnection: mongoose.connection,
    ttl: 24 * 60 * 60 // 1 day
  })
}));

app.use((req, res, next) => {
  if (req.session.currentUser) {
    res.locals.currentUserInfo = req.session.currentUser;
    res.locals.isUserLoggedIn = true;
  } else {
    res.locals.isUserLoggedIn = false;
  }

  next();
});

app.use('/', index);
// ...
```
Before the routes happen, this middle checks if there’s a session. If there is, it sets some locals on the response for the view to access. We’ve got two view locals:

    isUserLoggedIn: a boolean that indicates whether or not there is a logged in user
    currentUserInfo: the user’s information from the session (only available if logged in)

That middleware makes it easier to customize the home page for logged in users. Let’s do that now!

Here’s our updated views/index.hbs template file:
```html
<!-- views/index.hbs -->
<p> Welcome to {{ title }}. </p>

{{#if isUserLoggedIn}}
  <p> Hello, {{ currentUserInfo.name }}. </p>
{{/if}}

<nav>
  <ul>
    {{#if isUserLoggedIn}}
     <li> <a href="/launderers"> Find a Launderer </a> </li>
      <li> <a href="/dashboard"> See Dashboard </a> </li>
      <li> <a href="/logout"> Log Out </a> </li>
    {{else}}
      <li> <a href="/signup"> Sign Up </a> </li>
      <li> <a href="/login"> Log In </a> </li>
    {{/if}}
  </ul>
</nav>
```
Highlights:

    Lines 4-6: An if statement displays a special message for logged in users.
    Lines 10-17: An if..else statement displays some of the links to logged in users and others to anonymous users.

Next up - Log Out.