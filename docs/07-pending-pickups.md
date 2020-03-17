# Iteration #7: Pending Pickups

Finally, we need to display a user’s pending pickups. The obvious place to see this list is the dashboard page. Let’s add an extra query to that route.

You can see our overhauled /dashboard route in our lines 18-41 of routes/laundry.js:

```js
// ... inside of routes/laundry.js

router.get('/dashboard', (req, res, next) => {
  let query;

  if (req.session.currentUser.isLaunderer) {
    query = { launderer: req.session.currentUser._id };
  } else {
    query = { user: req.session.currentUser._id };
  }

  LaundryPickup
    .find(query)
    .populate('user', 'name')
    .populate('launderer', 'name')
    .sort('pickupDate')
    .exec((err, pickupDocs) => {
      if (err) {
        next(err);
        return;
      }

      res.render('laundry/dashboard', {
        pickups: pickupDocs
      });
    });
});

router.post('/launderers', (req, res, next) => {
// ...
```

Highlights from the new /dashboard route:

    Lines 19-25: Change the query based on whether or not you are a launderer. If the user is a launderer, find laundry pickups that they will launder. Otherwise, show laundry pickups that they ordered.
    Lines 27-30: Call several Mongoose methods to create a more complicated query, culminating with a call to the exec() method to provide our callback.
    Lines 28-29: Since the user and launderer properties are references to other documents, we are requesting for those references to be pre-populated with the name property from the User model.
    Line 30: Sort by pickupDate in ascending order (dates farthest in the future last).
    Lines 37-39: Render the views/laundry/dashboard.hbs template like before but this time inside the callback.
    Line 38: Pass in the results of the query (pickupDocs) as the local variable pickups.

Now that we are querying for extra information in the route, we need to display it in the view.

On our lines 31-44 in views/laundry/dashboard.hbs:

```html
<!-- ... inside of views/laundry/dashboard.hbs -->
<h3> Pending Pickups </h3>
<ul>
{{#each pickups}}
    <li>
      <h4> {{ this.pickupDate }} </h4>

      <ul>
        <li> <b>User</b>: {{ this.user.name }} </li>
        <li> <b>Launderer</b>: {{ this.launderer.name }} </li>
      </ul>
    </li>
{{/each}}
</ul>
```

Now visit your dashboard and see your pending pickups.