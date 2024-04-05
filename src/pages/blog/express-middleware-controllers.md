---
layout: "../../layouts/BlogPost.astro"
title: "Express Middleware vs. Controllers"
description: "Express Middleware vs. Controllers"
pubDate: "Apr 5 2024"
heroImage: "/node-hero.png"
previewText: "A common design pattern in Express.js applications is the concept of middleware and controllers, where a route can be defined with a sequence of middleware functions followed by a final handler function, which often acts as a controller. However, there are a few nuances and best practices to consider for clarity and accuracy."
---

# Overview

A common design pattern in Express.js applications is the concept of middleware and controllers, where a route can be defined with a sequence of middleware functions followed by a final handler function, which often acts as a controller.

For example

```javascript
app.get("/myroute", middleware1, middleware2, function (req, res) {
  controller1();
  controller2(req, res);
});
```

However, there are a few nuances and best practices to consider for clarity and accuracy:

1. **Middleware Sequence**: The `middleware1` and `middleware2` are middleware functions that will be executed in the order they are listed. Each middleware has the opportunity to process the request, make changes to the request or response objects, and either pass control to the next middleware in the chain or terminate the request-response cycle.

2. **Controller as Final Handler**: The final function in the route definition often acts as a controller. It's responsible for handling the request given the processing done by preceding middleware and generating a response. In Express.js, it's common for this function to directly interact with models (if following an MVC pattern) and send a response back to the client.

3. **Controller Execution**: In this example, `controller1();` is called without passing the `req` and `res` objects, which might be appropriate if `controller1` does not need to interact with the request or response. However, it's more common for controller functions to require access to `req` and `res` to perform their operations and send a response. Therefore, `controller2(req, res);` is a more typical usage where the controller function takes `req` and `res` as arguments to process the request and return a response.

4. **Sending a Single Response**: It's important to ensure that only one response is sent back to the client for a given request. This means that in a typical scenario, you would not call multiple controller functions that each attempt to send a response (e.g., using `res.send()` or `res.json()`). Instead, you would have a single controller function or handler at the end of your middleware chain that concludes the request-response cycle by sending a response.

Given these considerations, a more typical structure might look like this:

```javascript
app.get("/myroute", middleware1, middleware2, (req, res) => {
  // This acts as the controller
  controllerFunction(req, res);
});
```

Or, if the controller function is defined elsewhere and designed to handle the request and response objects:

```javascript
app.get("/myroute", middleware1, middleware2, controllerFunction);
```

In this structure, `controllerFunction` is expected to be a function that takes `req` and `res` as parameters, performs the necessary business logic, and sends a response to the client.

An example project structure where middleware and controllers are defined in separate files can be found [here](https://github.com/geshan/expressjs-structure/tree/master/src). This allows for more readable code, since not all business logic and routing occurs in a single file.

![Example Express.js project structure](/express-middleware/example_project_structure.png)

# Example Middleware

Middleware can be implemented globally, in which case it will be used by all routes. For example, when using CORS middleware

```javascript
const cors = require("cors");
app.use(cors());
```

Or when using logging middleware

```javascript
const morgan = require("morgan");
app.use(morgan("dev"));
```

Or when defining a route to static files

```javascript
app.use(express.static("public"));
```

Of course we can apply middleware on a route-by-route basis.

```javascript
function isAuthenticated(req, res, next) {
  if (req.session.userid) {
    next();
  } else {
    req.session.redirectTo = req.originalUrl;
    return res.redirect("/login");
  }
}
app.use("/profile", isAuthenticated);
```

# Examples Controllers

```javascript
// UserController.js
exports.createUser = (req, res) => {
  // Logic to create a new user

    const email = req.body.email;
    const password = req.body.password;
    const confirmPassword = req.body['confirm-password'];

    if (!email || !password || !confirmPassword) {
        logger.info(`Failed registration attempt for user: ${email} from IP: ${req.ip}`)
        return res.render('register.njk', { message: 'Please fill out all fields' });
    }

    // check if user already exists
    const user = await User.findOne({
        where: { email: email }
    });

    if (user) {
        logger.info(`Failed registration attempt for user: ${email} from IP: ${req.ip}`)
        return res.render('register.njk', { message: 'User already exists' });
    }

    if (!validatePassword(password)) {
        logger.info(`Failed registration attempt for user: ${email} from IP: ${req.ip}`)
        return res.render('register.njk', {
            message: 'Password contains invalid characters.'
        });
    }

    if (password !== confirmPassword) {
        logger.info(`Failed registration attempt for user: ${email} from IP: ${req.ip}`)
        return res.render('register.njk', { message: 'Passwords do not match' });
    }

    // async mode recommended because hashing is potentially CPU intensive
    try {
        const hashedPassword = await bcrypt.hash(password, saltRounds);
        const user = await User.create({ email: email, hashedPassword: hashedPassword });
        logger.info(`User created: ${email}`);
        return res.redirect('/login');
    } catch (error) {
        logger.error(`Registration Error: ${error}`);
        return res.render('register.njk', { message: 'Registration failed, please try again' });
    }
};

exports.loginUser = (req, res) => {
    // Logic for user login

    const email = req.body.email;
    const password = req.body.password;

    if (!email || !password) {
        return res.render('login.njk', { message: 'Please fill out all fields' });
    }

    const user = await User.findOne({ where: { email: email } });

    if (!user) {
        logger.info(`Failed login attempt for user: ${email} from IP: ${req.ip}`)
        return res.render('login.njk', { message: 'Invalid email or password' });
    }

    try {
        const result = await bcrypt.compare(password, user.hashedPassword);
        if (!result) {
            logger.info(`Failed login attempt for user: ${email} from IP: ${req.ip}`)
            return res.render('login.njk', { message: 'Invalid email or password' });
        }
    } catch (error) {
        logger.error(`Server Error: ${error}`);
        return res.render('login.njk', { message: `Server Error. Please try again.` });
    }

    // Redirect to the original URL or default to the homepage
    const redirectTo = req.session.redirectTo ? req.session.redirectTo : '/';
    delete req.session.redirectTo; // Remove the property once used

    // regenerate the session, which is good practice to help
    // guard against forms of session fixation
    req.session.regenerate(function (err) {
        if (err) next(err)

        // store user information in session, typically a user id
        req.session.userid = user.id

        // save the session before redirection to ensure page
        // load does not happen before session is saved
        req.session.save(function (err) {
            if (err) return next(err)
            res.redirect(redirectTo)
        })
    })
};
```

# Conclusion

Implementing Middleware and Controllers allows you to abstract away unnecessary details from your routes, resulting in fewer lines, and more readable code.
