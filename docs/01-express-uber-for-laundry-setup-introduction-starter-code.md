# Learning Goals

    Add basic user authentication to an Express application.
    Create sessions in an Express application.
    Store user information in the session and use it in some routes.
    Prevent anonymous (logged out) users from accessing parts of your application.

## Setup

Clone the Uber For Laundry repo into your ~/code/labs folder to get the starter code.

```
$ git clone https://github.com/danieljdsgit/uber-for-laundry.git
$ cd uber-for-laundry/
```

Run npm install to get all the modules from the package.json file.

```
$ npm install
```

## Introduction

Let’s face it: everybody has to do laundry. Even Batman.

![](giphy.gif)

Laundry takes so much of our time. Don’t even get me started on folding. It’s often tempting to just not do it. Go out in dirty clothes. Who will know?

Until the day when we we all have robots to do our laundry for us, we have to do it ourselves.

Or do we?

What if we could make an app in which we could have other people do our laundry for us. No, that’s not what your mother is for. I’m talking about strangers with a lot of time on their hands that want to make some extra money on the side. Regular people like you or me!

Let’s create an app to solve the working professional’s laundry problem using Express, Mongoose, bcrypt and sessions. Maybe we really will become Uber for Laundry!

The app will allow users to register and schedule a laundry pickup. They will be able to:

    Sign up as a user.
    Log in.
    Log out.
    Become a launderer (optional).
    Find a launderer.
    Schedule a laundry pickup with a launderer.
    See pending pickups.

## Starter Code

The starter code for this project includes:

    An app structure created by the express-generator.
    Layout already added for you.
    Views for all the pages we are going to be working with.
    The User and LaundryPickup models.
```
        starter-code/
        ├── .gitignore
        ├── app.js
        ├── bin
        │   └── www
        ├── models
        │   ├── laundry-pickup.js
        │   └── user.js
        ├── package.json
        ├── public
        │   └── stylesheets
        │       └── style.css
        ├── routes
        │   ├── index.js
        │   └── users.js
        └── views
            ├── auth
            │   ├── login.hbs
            │   └── signup.hbs
            ├── error.hbs
            ├── index.hbs
            ├── laundry
            │   ├── dashboard.hbs
            │   ├── launderer-profile.hbs
            │   └── launderers.hbs
            └── layout.hbs
```

If you visit the homepage, you will see that there are a bunch of links that don’t work. We will be adding the routes for each of those pages as part of this lesson.

Let’s get to coding!