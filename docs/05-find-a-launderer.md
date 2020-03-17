# Iteration #5: Find a Launderer

Ultimately we want our users to get their laundry picked up. That process starts by finding a launderer for your pickup. Let’s make a page that displays a list of users who have become launderers.

Before we start register and log in as 5 different users. Become a launderer with 3 of them.

Now we have some users who are launderers and others who aren’t.

To display the list of launderers, we will have users visit the /launderers page. That means we need to add a route for /launderers.

On lines 9-16 in routes/laundry.js you can see our new route:
```js
// ... inside of routes/laundry.js
    req.session.currentUser = theUser;

    res.redirect('/dashboard');
  });
});


router.get('/launderers', (req, res, next) => {
  User.find({ isLaunderer: true }, (err, launderersList) => {
    if (err) {
      next(err);
      return;
    }

    res.render('laundry/launderers', {
      launderers: launderersList
    });
  });
});


module.exports = router;
```
Highlights of the /launderers route:

    Line 45: Query users whose isLaunderer property is true.
    Lines 51-53: Render the views/laundry/launderers.hbs template.
    Lines 52: Pass in the results of the query (launderersList) as the local variable launderers.

Visit the page to see the list of launderers.

Next up - Schedule a Pickup.