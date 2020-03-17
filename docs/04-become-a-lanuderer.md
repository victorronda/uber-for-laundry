# Iteration #4: Become a Launderer

Now that we’ve gotten all our authentication out of the way, we can start on the actual features of the app. We first want to have a dashboard page for logged in users. In this dashboard, users will be able to become a launderer.

Create a route file in the routes/ folder called routes/laundry.js. This route file will have all the routes related to getting your laundry picked up.
```
$ touch routes/laundry.js
```
We have to make sure to wire our routes/laundry.js with the app.js. We do this by requiring it on the app.js:
```js
// ... inside of app.js
const authRoutes = require('./routes/auth');
const laundryRoutes = require('./routes/laundry');

// ...

We also need to configure our app variable to use those routes in our line 80 of app.js:

// ... inside of app.js

app.use('/', index);
app.use('/', authRoutes);
app.use('/', laundryRoutes);

// ...
```
Now that the wiring is in place, we can add the initial contents of routes/laundry.js:
```js
// routes/laundry.js
const express = require('express');

const router = express.Router();


router.get('/dashboard', (req, res, next) => {
  res.render('laundry/dashboard');
});


module.exports = router;
```
The /dashboard route renders the views/laundry/dashboard.hbs view. That template contains a form that a user will submit to become a launderer. In other words, the form will submit to a POST route that will require us to update user’s information in the database. The updates should save their new status as launderers. We will be updating the isLaunderer and fee properties when this form is submitted.

Out POST route is /launderers. Let’s add that presently. Here’s our updated routes/laundry.js:

```js
// routes/laundry.js
const express = require('express');

const User = require('../models/user');

const router = express.Router();


router.get('/dashboard', (req, res, next) => {
  res.render('laundry/dashboard');
});

router.post('/launderers', (req, res, next) => {
  const userId = req.session.currentUser._id;
  const laundererInfo = {
    fee: req.body.fee,
    isLaunderer: true
  };

  User.findByIdAndUpdate(userId, laundererInfo, { new: true }, (err, theUser) => {
    if (err) {
      next(err);
      return;
    }

    req.session.currentUser = theUser;

    res.redirect('/dashboard');
  });
});


module.exports = router;
```
Highlights of this POST route:

    Line 13: Get the user’s _id from the session.
    Lines 14-17: Prepare the updated information with the fee from the form and isLaunderer hardcoded to true.
    Line 19: Call Mongoose’s findByIdAndUpdate() method to perfom the updates.
    Line 25: Update the user’s information in the session. This works in concert with line 19’s { new: true } option to get the updated user information in the callback.
    Line 27: Redirect back to the dashboard.

Now that the code is in place, try to become a launderer! We can verify that it worked by going into MongoDB Compass and querying the database:

Confirm that the user’s fee and isLaunderer properties have changed.

Users won’t be able to check MongoDB Compass line though. It’s even annoying for us! We should show feedback in the dashboard about the success of becoming a launderer.

Here’s our updated views/laundry/dashboard.hbs template:

```html
<!-- views/laundry/dashboard.hbs -->
<h2> Your laundry Dashboard </h2>

<ul>
  <li> <a href="/launderers"> Find a Launderer </a> </li>
  <li> <a href="/logout"> Log Out </a> </li>
</ul>

{{#if currentUserInfo.isLaunderer}}
  <h3> You are a launderer </h3>

  <p>Your laundering fee is <b>${{ currentUserInfo.fee }}.</b></p>
{{else}}
  <h3> Want to become a launderer? </h3>

  <form action="/launderers" method="post">
    <div>
      <label for="fee-input"> Set your fee </label>
      <input type="number" name="fee" id="fee-input">
    </div>

    <button> Become a Launderer </button>
  </form>
{{/if}}
```

Highlights from the new views/laundry/dashboard.hbs:

    Lines 8-13: We have an if statement that displays new HTML if the user is already a launderer.
    Lines 15-22: The form that was previously there is now being displayed in the else.

End result: if you are not a launderer yet, you see the form and if you are you see a message and your fee.

Finally, we should add some authorization to our dashboard. As it stands, even you if you aren’t logged in, if you visit the /dashboard page directly you will be able to see it.

Let’s add some authorization to all the laundry routes. We will do that by adding a middleware to routes/laundry.js.

On lines 7-14 in routes/laundry.js you can see our middleware:

```js
// routes/laundry.js
const express = require('express');

const User = require('../models/user');

const router = express.Router();


router.use((req, res, next) => {
  if (req.session.currentUser) {
    next();
    return;
  }

  res.redirect('/login');
});


router.get('/dashboard', (req, res, next) => {
// ...
```
This middleware runs before any of our laundry routes.

Highlights:

    Lines 8-11: If there’s a user in the session (logged in), continue with the routes by calling next() and returning.
    Line 13: If there’s no user in the session (anonymous), redirect the browser to the log in page.

Next up - Find a Launderer.