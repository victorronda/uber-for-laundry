# Iteration #1: Sign Up

First thing we should do is allow our users to sign up. The route we will use for sign up is /signup.

Create a route file in the routes/ folder called routes/auth.js. This route file will have all the routes related to user registration and authentication. In other words, this file will contain all the routes necessary for sign up and log in.

```
$ touch routes/auth.js
```

We have to make sure to wire our routes/auth.js with the app.js. We do this on by requiring it on the app.js:

```js
// ... inside of app.js

const index = require('./routes/index');
const authRoutes = require('./routes/auth');

const app = express();
// ...
```

We also need to configure our app variable to use those routes on the app.js:

```js
// ... inside of app.js

app.use('/', index);
app.use('/', authRoutes);
```

Now that the wiring is in place, we can add the contents of routes/auth.js:

```js
// routes/auth.js
const express = require('express');

const router = express.Router();


router.get('/signup', (req, res, next) => {
  res.render('auth/signup', {
    errorMessage: ''
  });
});


module.exports = router;
```

The /signup route renders the views/auth/signup.hbs view. Notice the errorMessage local variable. That’s there to display feedback messages to the user. When you first visit the page, the feedback message is blank, thus the empty string.

The /signup route contains a form that a user will submit to register with the app. In other words, the form will submit to a POST route that will require us to save the user’s information in the database. We’ve already got the code for our User model in models/user.js, but we haven’t included Mongoose in the app.

Aside from saving things the database, registration in our app will require us to encrypt the user’s password.

Install the bcryptjs package in the terminal:

```
$ npm install --save bcryptjs
```

Now we can finish our POST route that will receive the submission of the Sign Up form. Here’s our updated routes/auth.js file:

```js
// routes/auth.js
const express = require('express');
const bcrypt = require('bcryptjs');

const User = require('../models/user');

const router = express.Router();
const bcryptSalt = 10;

router.get('/signup', (req, res, next) => {
  res.render('auth/signup', {
    errorMessage: ''
  });
});

router.post('/signup', (req, res, next) => {
  const nameInput = req.body.name;
  const emailInput = req.body.email;
  const passwordInput = req.body.password;

  if (emailInput === '' || passwordInput === '') {
    res.render('auth/signup', {
      errorMessage: 'Enter both email and password to sign up.'
    });
    return;
  }

  User.findOne({ email: emailInput }, '_id', (err, existingUser) => {
    if (err) {
      next(err);
      return;
    }

    if (existingUser !== null) {
      res.render('auth/signup', {
        errorMessage: `The email ${emailInput} is already in use.`
      });
      return;
    }

    const salt = bcrypt.genSaltSync(bcryptSalt);
    const hashedPass = bcrypt.hashSync(passwordInput, salt);

    const userSubmission = {
      name: nameInput,
      email: emailInput,
      password: hashedPass
    };

    const theUser = new User(userSubmission);

    theUser.save(err => {
      if (err) {
        res.render('auth/signup', {
          errorMessage: 'Something went wrong. Try again later.'
        });
        return;
      }

      res.redirect('/');
    });
  });
});

module.exports = router;
```

To point out a few interesting things about the code we’ve added:

    Lines 2 and 4: Require bcryptjs and the User model for use in our POST route.
    Line 15: Define our POST route with the /signup URL. It can have the same URL because it uses a different HTTP verb (GET vs. POST).
    Lines 16-18: Make variables for the inputs submitted by the form (stored in req.body).
    Lines 40-41: Use the bcrypt methods genSaltSync() and hashSync() to encrypt the submitted password.
    Lines 43-49: Create an instance of the User model with the correct properties (values from the form submission).
    Line 51: Call Mongoose’s save() model method to actually save the new user to the database.
    Line 59: If everything goes as planned, redirect back to the home page.

That was all assuming that there weren’t any problems with the submission. Our code does check for different situations in which we wouldn’t want to save a new user. In those cases, we render the sign up form again with an error message. That’s what the errorMessage local variable was for. Here are the problems we are checking for:

    Lines 20-25: First thing we check for is if the email or password is blank.
    Lines 33-38: Check if there’s already a user with the submitted email.
    Lines 52-57: Check for database errors when we save.

Now that the sign up code is in place, try to sign up! We can verify that the registration worked by going into the MongoDB Compass and querying the database:

Next up - Log In.