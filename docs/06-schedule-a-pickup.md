# Iteration #6: Schedule a Pickup

Now that we can see a list of launderers, let’s select a launderer and schedule them for a pickup. In the list of launderers, each launderer has a Schedule a Pickup link. That link is supposed to take you to a launderer profile page where you can schedule your pickup. Let’s make a route for that page!

The route in question is /launderers/:id. We add it on lines 57-65 in routes/laundry.js:

```js
// ... inside of routes/laundry.js
    res.render('laundry/launderers', {
      launderers: launderersList
    });
  });
});


router.get('/launderers/:id', (req, res, next) =>
 {
  const laundererId = req.params.id;

  User.findById(laundererId, (err, theUser) => {
    if (err) {
      next(err);
      return;
    }

    res.render('laundry/launderer-profile', {
      theLaunderer: theUser
    });
  });
});


module.exports = router;
```

Highlights from the /launderers/:id route:

    Line 57: Grab the id URL parameter.
    Lines 60: Call Mongoose’s findById() method to retrieve the launderer’s details.
    Lines 66-68: Render the views/laundry/launderer-profile.hbs template.
    Lines 67: Pass in the launderer’s profile information (theUser) as the local variable theLaunderer.

Visit the /launderer/:id page and you’ll see a form to schedule a pickup with that user. When that form is submitted, we need to save the laundry pickup in the database. We’ve already got the code for the LaundryPickup model in models/laundry-pickup.js. We just need to require it and use it in our POST route.

See our new line 5 in routes/laundry.js:

```js
// routes/laundry.js
const express = require('express');

const User = require('../models/user');
const LaundryPickup = require('../models/laundry-pickup');

const router = express.Router();
// ...
```

Now let’s add our POST /laundry-pickups route. In our lines 74-91 in routes/laundry.js:

```js
// ... inside of routes/laundry.js
router.get('/launderers/:id', (req, res, next) => {
  const laundererId = req.params.id;

  User.findById(laundererId, (err, theUser) => {
    if (err) {
      next(err);
      return;
    }

    res.render('laundry/launderer-profile', {
      theLaunderer: theUser
    });
  });
});


router.post('/laundry-pickups', (req, res, next) => {
  const pickupInfo = {
    pickupDate: req.body.pickupDate,
    launderer: req.body.laundererId,
    user: req.session.currentUser._id
  };

  const thePickup = new LaundryPickup(pickupInfo);

  thePickup.save((err) => {
    if (err) {
      next(err);
      return;
    }

    res.redirect('/dashboard');
  });
});


module.exports = router;
```

Highlights from the POST /laundry-pickups route:

    Lines 75-81: Create an instance of the LaundryPickup model with the correct properties.
    Lines 76-77: pickupDate and launderer properties come from the the form. The launderer input is an <input> tag with type="hidden". Check the HTML!
    Line 78: Grab the user _id from the session again.
    Line 83: Call Mongoose’s save() model method to actually save the pickup to the database.
    Line 89: If everything goes as planned, redirect back to the dashboard.

Now we can schedule a pickup! We can verify that it worked by going to MongoDB Compass and querying the database:

Next up - Pending Pickups.