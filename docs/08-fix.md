# Iteration #8: Fix

We can add a small fix so we don't see owerselves as available launderers.
Change the search filter in the '/launderers' route so we can see all the launderers except ourselves.

```js
router.get('/launderers', (req, res, next) => {
  User.find(
    {
      $and: [
        { isLaunderer: true },
        { _id: { $ne: req.session.currentUser._id } }
      ]
    },
    (err, launderersList) => {
      if (err) {
        next(err);
        return;
      }
      res.render('laundry/launderers', {
        launderers: launderersList
      });
    }
  );
});
```
