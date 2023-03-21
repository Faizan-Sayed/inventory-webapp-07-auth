# Part 07: Authentication and Authorization  

This tutorial follows after:
[Part 06: First Deployment](https://github.com/atcs-wang/inventory-webapp-05-handling-forms-post-crud)



Some more detailed "next steps" for using Auth0 w/ Node/Express here: [https://github.com/auth0/express-openid-connect/blob/master/EXAMPLES.md#2-require-authentication-for-specific-routes](https://github.com/auth0/express-openid-connect/blob/master/EXAMPLES.md#2-require-authentication-for-specific-routes)


# (7.0) Get started with Auth0

Go to [https://auth0.com/](https://auth0.com/) and create an account.

Once logged into, you can follow the `Getting Started -> Create Application` wizard. 
Choose:
  * At  `Create Application`, give your app a Name and choose `Regular Web Applications` as your application type.

  * At  `What technology are you using for your project`, choose `Node.js (Express)`

  * At  `Choose your path`, click `Integrate Now` beneath `I want to integrate with my app`

  * At  `Configure Auth0`, the `Allowed Callback URL` box should contain `http://localhost:3000/callback` and the `Allowed Logout URLs` box should contain `http://localhost:3000`. Change the number from `3000` to whatever your PORT value is during development. (Mine is 8080)   

  * At `Integrate Auth0`, take the command 

    > `npm install express express-openid-connect --save`

    and run it in your Terminal. This will install the `express-opendid-connect` package.

    Then look at the code provided in the `Configure Router` section, which is already configured with your information. It contains 4 "lines", which we will add to our `app.js`
    
    * Copy the first `const { auth } = require('express-openid-connect');` line and add it among the other `require` statements at the top of the file.

    * Copy the `const config=...` and `app.use(auth(config))` lines except for the last section and add it after the helmet , before any of the other `app.use` lines. 
    
    * Then add the last line `app.get('/',...);` after the other `app.use` lines, but change the route path from `/` to `/authtest`.

    The added code, in context, should look like this:

    ```js
    //Other require lines
    ...
    const { auth } = require('express-openid-connect');
    ...

    // Helmet middleware
    app.use(helmet({...
    });

    // CODE FROM AUTH0:
    const config = {
      authRequired: false,
      auth0Logout: true,
      secret: 'a long, randomly-generated string stored in env',
      baseURL: 'http://localhost:<PORT>',
      clientID: '<YOUR CLIENT ID>',
      issuerBaseURL: 'https://<YOUR DOMAIN>.us.auth0.com'
    };

    // auth router attaches /login, /logout, and /callback routes to the baseURL
    app.use(auth(config));

    // Rest of the app.use(...) middleware
    ...

    // req.isAuthenticated is provided from the auth router
    app.get('/authtest', (req, res) => {
      res.send(req.oidc.isAuthenticated() ? 'Logged in' : 'Logged out');
    });

    // Rest of the code, e.g. routes
    ...
    ``` 

    Now go to the next section.
  
  * At `Test your login`, start up your server with `npm start`
    * Follow the instructions to visit the `/login` and `/logout` routes. Auth0's monitoring should show activity. 

    *  You should also visit `/authtest` after both logging in and logging out to see that the webapp is aware of your logged-in/out status.

  *  At `Get the user profile information`, copy the code provided, and add them to `app.js`:
    * The `const { requiresAuth } = require('express-openid-connect');` can be added to the top of the file.

    * The other line can be added anywhere after the `app.use(auth(config))` line, among the routing code.

    The added code, in context, should look like this:

    ```js
    //Other require lines
    ...
    const { requiresAuth } = require('express-openid-connect');

    //Other code including app.use(auth(config))
    ...

    app.get('/profile', requiresAuth(), (req, res) => {
      res.send(JSON.stringify(req.oidc.user));
    });

    // Rest of the code, e.g. routes
    ...

    ```
  * Restart your server and visit the `/profile` route, both logged in and logged out.
  * The wizard should be complete, and you can click `Go To Application Settings`.


## (7.2) Securing Auth0 secrets in `.env`, setting up for deployment

Our authentication basics work for our development code, but a few things need to be fixed up before we can push any changes and (re-)deploy our app.

The `config` object in `app.js` contains all the Auth0 info in plaintext. These should be kept secret via environment variables.

1. Add these lines to your `.env` file, replacing the values with the same values used in your `config` object.

```
AUTH0_SECRET=<SECRET>
AUTH0_BASE_URL=http://localhost:<PORT>
AUTH0_CLIENT_ID=<YOUR CLIENT ID>
AUTH0_ISSUER_BASE_URL=https://<YOUR DOMAIN>.us.auth0.com
```

The `<SECRET>` can technically be replaced with anything, but Auth0's `Application Settings` page provides a good `Client Secret` that you can copy and use.

2. Add this in the top section (among the `requires`) to your `app.js`:
  ```js
  const dotenv = require('dotenv');
  dotenv.config();
  ```

3. Change the `config` object in `app.js` to: 
  ```js
  const config = {
    authRequired: false,
    auth0Logout: true,
    secret: process.env.AUTH0_SECRET,
    baseURL: process.env.AUTH0_BASE_URL,
    clientID: process.env.AUTH0_CLIENT_ID,
    issuerBaseURL: process.env.AUTH0_ISSUER_BASE_URL
  };
  ```

> Double check that your server is still able to do `/login` and `/logout` successfully.

Now you can push the code without worrying about exposing the secrets. 

However, if you have a deployed app, we need it to be given those environment variables too. 

The only one that needs to be changed for the deployed environment is the `AUTH0_BASE_URL`, which should be the URL of the actual hosted site, not `localhost`.

Then, you need to update the `Application Settings` on Auth0: In the `Application URIs` section, change the `Allowed Callback URL` box from just `http://localhost:PORT/callback` to :
  ```
    http://localhost:PORT/callback,
    https://<HOSTDOMAIN>/callback
  ```
  and the `Allowed Logout URLs` box from just `http://localhost:PORT` to:

  ```
    http://localhost:PORT,
    https://<HOSTDOMAIN>
  ``` 

Save your changes.
## (7.3) Adding a log-in / log-out button (+ nav bar)

We obviously need a way to log in and out that's more intuitive for the user than changing the URL manually. 

We can easily add Login and Logout buttons to our pages, which are just links to `/login` and `/logout` 
```html
    <a href="/login" class="btn blue">Login</a></li>
    <a href="/logout" class="btn red">Logout</a></li>
```

They would look better on a navigation bar that is consistent at the top of the each page.
> We have neglected to make a nav bar thus far in our project. In retrospect, we should have just added one at the prototype stage. 
Materialize has some nice support for good looking nav bars, which can collapse for small screens like mobile devices, and provide an alternative side menu nav.

Go ahead and add one into each of our views at the top of the body... 

```html
...
</head>
<body>
    <!-- Nav bar -->
    <header>
        <nav>
            <div class="nav-wrapper">
                <a href="/" class="brand-logo">Stuff Manager</a>
                <a href="#" data-target="mobile-nav" class="sidenav-trigger"><i class="material-icons">menu</i></a>
                <!-- Main nav, hidden for small screens -->
                <ul id="desktop-nav" class="right hide-on-med-and-down">
                    <li><a href="/"><i class="material-icons left">home</i>Home</a></li>
                    <li><a href="/stuff"><i class="material-icons left">list</i>Inventory</a></li>
                    <li><a href="/login" class="btn blue">Login</a></li>
                    <li><a href="/logout" class="btn red">Logout</a></li>
                </ul>
            </div>
        </nav>
        <!-- Mobile nav menu, shown when menu button clicked -->
        <ul id="mobile-nav" class="sidenav">
            <li><a href="/"><i class="material-icons left">home</i>Home</a></li>
            <li><a href="/stuff"><i class="material-icons left">list</i>Inventory</a></li>
            <li><a href="/login" class="btn blue">Login</a></li>
            <li><a href="/logout" class="btn red">Logout</a></li>
        </ul>
    </header>

    <!-- rest of the body -->
    <div class="container">
      ...
```

If not already included, add Materilize source and initialization scripts, which are needed for the mobile side-nav to work.
```html
    <!-- Materialize JavaScript -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/js/materialize.min.js"></script>

    <!-- initialize Materialize elements -->
    <script src="/scripts/materializeAutoinit.js"></script>

```

> You should test the buttons to make sure they work.

## (7.4) Changing view on authentication; displaying user information

However, we only want to show one of the two buttons at a time - LOGIN if not yet authenticated, and LOGOUT if already authenticated. We might also like to show, if you are logged in, some of your user information, making it clear who you're logged in as.

The `/authtest` route from above demonstrated the `req.oidc.isAuthenticated()` method which tells us whether we are logged in, and the `/profile` route demonstrated the `req.oidc.user` object, which has user info included with the auth token.

In general, the state of an incoming request's authentication is available via `req.oidc`, and that can be used to determine how we respond. We want our server to make the state of authentication available to EJS during the rendering step. 


One way is pass it as part of the context parameter given at `res.render(...)`.

For example we *could* (but don't actually) change the `res.render` in the `/stuff` route handler to something like this:

```js
app.get( "/stuff", ( req, res ) => {
    ...
            res.render('stuff', { inventory : results , 
                                  isLoggedIn : res.oidc.isAuthenticated()
                                  user: res.oidc.user });
    ...
} );
```

This would provide `stuff.ejs` the `isLoggedIn` and `user` variables, and those could be used to change the view. 

However, if we wanted that information for *every* page we render, we'd have to repeatedly pass the value to each and every `res.render()` call - tedious, and easy to miss one as our app grows.

A better way is to create a single middleware that, for all routes, will add a "local" property to the `res` response object, passing the information to EJS for all page renders.

Add this to the `app.js` after the other middleware (`app.use`) but before any route handlers (`app.get`, `app.post`, etc.)

```js
app.use((req, res, next) => {
    res.locals.isLoggedIn = req.oidc.isAuthenticated();
    res.locals.user = req.oidc.user;
    next();
})
```

Now, we can assume the availability of two variables in all of our `.ejs` files:
* `isLoggedIn`, a boolean indicating our authentication status
* `user`, which is either `undefined` if not logged in, or the user info object from the auth token. 

Let's use them to customize our nav bar view ( on every page):

```html
<header>
        <nav>
            <div class="nav-wrapper">
                <a href="/" class="brand-logo">Stuff Manager</a>
                <a href="#" data-target="mobile-nav" class="sidenav-trigger"><i class="material-icons">menu</i></a>
                <ul id="desktop-nav" class="right hide-on-med-and-down">
                    <li><a href="/"><i class="material-icons left">home</i>Home</a></li>
                    <li><a href="/stuff"><i class="material-icons left">list</i>Inventory</a></li>
                    <% if (isLoggedIn) { %>
                        <li><a href="/profile"><i class="material-icons left">person</i> Hello, <%=user.name%></a> </li>
                        <li><a href="/logout" class="btn red">Logout</a></li>                        
                    <% } else { %>
                        <li><a href="/login" class="btn blue">Login</a></li>
                    <% } %>
                </ul>
            </div>
        </nav>
        <ul id="mobile-nav" class="sidenav">
             <li><a href="/"><i class="material-icons left">home</i>Home</a></li>
             <li><a href="/stuff"><i class="material-icons left">list</i>Inventory</a></li>
             <% if (isLoggedIn) { %>
                <li><a href="/profile"><i class="material-icons left">person</i> Hello, <%=user.name%></a> </li>
                <li><a href="/logout" class="btn red">Logout</a></li>                        
            <% } else { %>
                <li><a href="/login" class="btn blue">Login</a></li>
            <% } %>
        </ul>
    </header>
```

Test your nav bar on each page, both logged in and out. 

## (7.5) Authorization for specific routes
Now that our users can log in/out and see their authentication status, we now move the issue of *authorization* - only allowing certain kinds of people into certain parts of the site.

The most basic kind of authorization simply requires that the users have been authenticated at all. Auth0's `express-openid-connect` library provides the `requiresAuth()` function for this purpose, demonstrated by the sample `/profile` route we set up earlier:

```js
    app.get('/profile', requiresAuth(), (req, res) => {
      res.send(JSON.stringify(req.oidc.user));
    });
```

`requiresAuth()` produces a middleware function, which can be applied as a sort of "pre-handler" to any route we want to restrict to just authenticated users.

The only route we really want available to un-authenticated users is the homepage (plus the `/authtest` route); the rest should require authentication.

We can change each of these routes
```js
app.get( "/stuff", ( req, res ) => { ... }
app.get( "/stuff/item/:id", ( req, res ) => { ... }
app.get("/stuff/item/:id/delete", ( req, res ) => { ... }
app.post("/stuff/item/:id", ( req, res ) => { ... }
app.post("/stuff", ( req, res ) => { ... }
```

to include `requiresAuth()` as a route-specific middleware that occurs before the handlers, like this:

```js
app.get( "/stuff", requiresAuth(), ( req, res ) => { ... }
app.get( "/stuff/item/:id", requiresAuth(), ( req, res ) => { ... }
app.get("/stuff/item/:id/delete", requiresAuth(), ( req, res ) => { ... }
app.post("/stuff/item/:id", requiresAuth(), ( req, res ) => { ... }
app.post("/stuff", requiresAuth(), ( req, res ) => { ... }
```

Now, log out and attempt accessing various pages. You should be, in each case, redirected to the Auth0 login page.

> NOTE ON ALTERNATIVES: We could have also used the `requiresAuth()` middleware the way we have other middleware, like this:
> ```js 
> app.use(requiresAuth());
> ```
> This would require authentication for ALL route handlers below this line;  any route handlers above this line will not require authentication.

> If you simply want to require authentication for *every* page,  yet another alternative would be to change the `config` object's `authRequired` property to `true`.

## (7.6) Associating inventory data with users
Now that you can only access certain routes when authenticated, we can confidently utilize their user information in those routes. 

Our final step is a very consequential one: associate every item in our database with the user who created and owns that item. 

For now, we will use the user's email (`req.oidc.user.email`) as a unique identifier. 

### (7.6.1) Add a `userid` column to the database table

First, we need to add a column to the database table called something like "userid".


If you're using it, we can update `db/db_init.js` appropriately so the (re-)created table has that column. 

My `CREATE TABLE` script changed from:
  ```sql
      CREATE TABLE stuff (
        id INT NOT NULL AUTO_INCREMENT,
        item VARCHAR(45) NOT NULL,
        quantity INT NOT NULL,
        description VARCHAR(150) NULL,
        PRIMARY KEY (id)
  ```
  to 

  ```sql
      CREATE TABLE stuff (
        id INT NOT NULL AUTO_INCREMENT,
        item VARCHAR(45) NOT NULL,
        quantity INT NOT NULL,
        description VARCHAR(150) NULL,
        userid VARCHAR(50) NULL,
        PRIMARY KEY (id)
  ```

Then re-run with either:
```
> node db/db_init.js
```
or (if you set up the `npm` script in `package.json`)
```
> npm run initdb
```

### (7.6.2) Update the CREATE operation

First, we need to make it such that whenever a user tries to create a new item, their email is entered into the database along with it.

Find the SQL statement that inserts a new item in `app.js`. From this:

```sql
    INSERT INTO stuff
        (item, quantity)
    VALUES
        (?, ?)
```

to this:
```sql
    INSERT INTO stuff
        (item, quantity, userid)
    VALUES
        (?, ?, ?)
```

Then, in the `/stuff` POST request handler that executes that SQL, we can provide the user's email from the request as the third `?`. Change from this:

```js
app.post("/stuff", requiresAuth(), ( req, res ) => {
    db.execute(create_item_sql, [req.body.name, req.body.quantity], (error, results) => { 
      ...
    }
}
```

to this:

```js
app.post("/stuff", requiresAuth(), ( req, res ) => {
    db.execute(create_item_sql, [req.body.name, req.body.quantity, req.oidc.user.email], (error, results) => { 
      ...
    }
}
```

Then, restart your server and try using the "Add" form. If you check your database, you ought to see that whatever table row you added includes the logged-in user's email.

### (7.6.2) Update the READ (inventory) operation

Now we need to restrict our users to only seeing data that they've created and are associated with.

With the SQL that selects the full list of items, change it from :

```sql 
    SELECT 
        id, item, quantity
    FROM
        stuff
```

to :
```sql
    SELECT 
        id, item, quantity
    FROM
        stuff
    WHERE 
        userid = ?
```

Then, in the `/stuff` GET request handler that executes that SQL, we can once again provide the user's email from the request as the new `?`. Change from this:


```js
app.get( "/stuff", requiresAuth(), ( req, res ) => {
    db.execute(read_stuff_all_sql, (error, results) => {
      ...
    }
}
```

to this:

```js
app.get( "/stuff", requiresAuth(), ( req, res ) => {
    db.execute(read_stuff_all_sql, [req.oidc.user.email], (error, results) => {
```

Now, revisit the `/stuff` page. You should only see the item(s) created after `userid` was incorporated into the database. Add more items and confirm that they appear. Log out and log in as various users, creating new items and confirming that only those items, and not other user's data, is displayed.

### (7.6.3) Update the UPDATE, DELETE, and READ (item) operations

While users can now only see data that they've created themselves on the `/stuff` inventory page, they can still access the item detail pages for items created by *any* user. 

> Try navigating to `/stuff/item/:id` for various ids of items that aren't created by the logged in user to see that this is the case.

Even worse, they can also edit and delete those items as well!

> Try both the edit form and delete buttons on those pages.

We need to restrict the accessibility of the routes associated with the UPDATE, DELETE, and READ operations for individual items. Fortunately, all are quite similar.


#### Restrict UPDATE:

With the SQL that updates a single item, change it from :

```sql 
    UPDATE
        stuff
    SET
        item = ?,
        quantity = ?,
        description = ?
    WHERE
        id = ?
```

to :
```sql
    UPDATE
        stuff
    SET
        item = ?,
        quantity = ?,
        description = ?
    WHERE
        id = ?
    AND
        userid = ?
```

Then, in the `/stuff/item/:id` POST request handler that executes that SQL, we can provide the user's email from the request as the new `?`. Change from this:

```js
app.post("/stuff/item/:id", requiresAuth(), ( req, res ) => {
    db.execute(update_item_sql, [req.body.name, req.body.quantity, req.body.description, req.params.id], (error, results) => {
      ...
    }
}
```

to this:

```js
app.post("/stuff/item/:id", requiresAuth(), ( req, res ) => {
    db.execute(update_item_sql, [req.body.name, req.body.quantity, req.body.description, req.params.id, req.oidc.user.email], (error, results) => {
      ...
    }
}
```
> Check that filling out the Edit form for an item owned by another user results in no changes. It should still refresh the page on submit, but no changes will be made.

#### Restrict DELETE 
Next, the `DELETE` operation needs to be locked down. 

With the SQL that deletes a single item, change it from :

```sql 
    DELETE 
    FROM
        stuff
    WHERE
        id = ?
```

to :
```sql
    DELETE 
    FROM
        stuff
    WHERE
        id = ?
    AND
        userid = ?
```

Then, in the `/stuff/item/:id/delete` GET request handler that executes that SQL, we can once again provide the user's email from the request as the new `?`. Change from this:


```js
app.get("/stuff/item/:id/delete", requiresAuth(), ( req, res ) => {
    db.execute(delete_item_sql, [req.params.id], (error, results) => {
      ...
    }
}
```

to this:
```js
app.get("/stuff/item/:id/delete", requiresAuth(), ( req, res ) => {
    db.execute(delete_item_sql, [req.params.id, req.oidc.user.email], (error, results) => {
      ...
    }
}
```

> Verify that attempting a delete of another users' item  via the button does not, in fact, cause the deletion of the item; it should redirect you to the `/stuff` page, but the database should not have changed, and the item detail page still be accessible. 


#### Restrict READ (single item)

Lastly, with the SQL that selects a single item, change it from :

```sql 
    SELECT 
        id, item, quantity, description 
    FROM
        stuff
    WHERE
        id = ?
```

to :
```sql
    SELECT 
        id, item, quantity, description 
    FROM
        stuff
    WHERE
        id = ?
    AND
        userid = ?
```

Then, in the `/stuff/item/:id` GET request handler that executes that SQL, we can once again provide the user's email from the request as the new `?`. Change from this:


```js
app.get( "/stuff/item/:id", requiresAuth(), ( req, res ) => {
    db.execute(read_stuff_item_sql, [req.params.id], (error, results) => {      
      ...
    }
}
```

to this:

```js
app.get( "/stuff/item/:id", requiresAuth(), ( req, res ) => {
    db.execute(read_stuff_item_sql, [req.params.id, req.oidc.user.email], (error, results) => {
      ...
    }
}
```

Now, revisit various `/stuff/item/:id` pages, attempting to access items not created by the current user. You should get a `404` response.

> Although it's not exactly true that the item doesn't exist, it is appropriate to act like it doesn't since the user should not be aware of the existence of other user's items. More accurately, we'd say that it is not authorized for that user to access that page. In such cases, we sometimes prefer a `403 Forbidden` code instead of `404 Not Found`.

> Now both the `Edit` form and `Delete` button cannot be directly accessed any longer by unauthorized users.  It may seem unnecessary to have restricted those other operations when restricting access to the detail page makes it pretty difficult for normal users to accidentally edit or delete an item they can't see. But it's safer to be thorough, as arbitrary GET and POST requests can in fact still be made by malicious users without forms or buttons. 

### (7.7) Conclusion

At this point, we have integrated authentication and some basic authorization.  We have associated data with specific users, and only authorized users to perform CRUD operations on their own data.

What next? Our app could use a user management system, accessible by a limited number of admin users. 

Before we do that, however, there are some structural changes and improvements that we ought to make to our project that will pay dividends down the road. 
