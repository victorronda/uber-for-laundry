# Iteration #3: Log Out

Now that we’ve logged in, we need to be able to log out. We’ve got links to the /logout route in our app, we just have to define that route.

Here’s the code for the route on our lines 101-115 in routes/auth.js:

```js
// ... inside of routes/auth.js
    req.session.currentUser = theUser;
    res.redirect('/');
  });
});


router.get('/logout', (req, res, next) => {
  if (!req.session.currentUser) {
    res.redirect('/');
    return;
  }

  req.session.destroy((err) => {
    if (err) {
      next(err);
      return;
    }

    res.redirect('/');
  });
});


module.exports = router;
```

Highlights:

    Line 107: Call the req.session.destroy() to clear the session for log out.
    Line 113: Redirect to the home page when it’s done.

Next up - Become a Launderer.