# Creating an Express App using MVC

## First steps to making the app

- Create a server.js file inside your new project folder.
- Run `npm init -y` to start a node project and automatically add a `package.json` file.
- Run `npm i express ejs morgan method-override` to install the Express, EJS, logger, and method override modules to the project.

## The entry point: **the `server.js` file**

1. *Load NPM or custom modules.*
   - NPM modules are loaded using the `require` keyword.
     - **CONVENTION:** Use the module name for the variable name.
     - **CODE:** `const express = require("express");`
   - Key middleware also need to be loaded here.
     - **CODE:** `const morgan = require("morgan");`
     - **CODE:** `const methodOverride = require("method-override");`
   - Custom modules like routers also need to be loaded here.
     - **CONVENTION:** Use `pathRouter` for the module name for a given `path`.
     - Routers are stored in a folder within root, so the argument to `require` is `./routes/[path]`.
     - **CODE:** `const indexRouter = require("./routes/index");`

2. *Initialize the Express app.*
   - Express is initialized using the `express` function with no arguments.
     - `express` is a constructor function that returns an Express object.
     - **CONVENTION:** Use `app` for the variable name of the new Express object.
     - **CODE:** `const app = express();`
     
3. *Configure application settings.*
   - Application settings are configured using the `set` method of `app`.
     - View engines like EJS are loaded here.
     - **CODE:** `app.set("view engine", "ejs");`

4. *Mount application middleware.*
   - Middleware is mounted to the application using the `use` method of `app`.
   - Key middleware:
     - `morgan` - HTTP logger that logs requests in terminal
       - **CODE:** `app.use(morgan("dev"));`
       - `morgan()` accepts configuration settings as arguments.
     - `express.urlencoded` - aka body-parser; parses data sent in forms and populates a `req.body` object with the data
       - **CODE:** `app.use(express.urlencoded({ extended: false, }));`
       - `extended: false` specifies that the parser doesn't need to expand to input body to include unnecessary information.
     - `express.static` - static asset server; allows `css`, `js`, and `img` files to be included in the application
       - **CODE:** `app.use(express.static("public"));`
       - **CONVENTION:** Use `public` as the directory for the static assets.
       - If `public` is used as the directory, links to CSS files in views should be in relation to `public`, or `/css/style.css`.
     - `method-override` - override module that allows PUT and DELETE methods for environments that don't support them
       - **CODE:** `app.use(methodOverride("_method"));`
   - Custom middleware:
     - Custom middleware is called using `app.use` with a callback function as an argument.
       - The parameters to the callback function are `req`, `res`, and `next`.
         - `req` is the request object that contains data and metadata about the request itself.
         - `next` is mandatory to indicate that this function is custom middleware.
       - Within the body of the callback function, you MUST call `next()` to move the HTTP request along the pipeline.
     - When `next()` is called, the request moves on immediately to the next middleware.
     - **CODE (most minimal):** `app.use(function(req, res, next) { next(); });`

5. *Mount route handlers.*
   - Links to the route handlers are mounted using the `use` method of `app`.
   - The first argument of `use` is a path. If a request is made to this path, then the request will be directed to the appropriate router module.
   - The second argument is the `pathRouter` router module variable from the `require` statement in Step 1: *Load NPM or custom modules*.
   - **CODE:** `app.use("/", indexRouter);` OR `app.use("/subpath", subpathRouter);`

6. *Tell the application to listen on a port for requests.*
   - The port will be used to run the Node process.
   - Ports must be unique.
   - **CONVENTION:** Use `port 3000` for Express applications.
   - **CODE:** `app.listen(3000, function() { console.log("Express is listening on port 3000.") });`
     - The first argument is the number of the port for your app, usually `3000`.
     - The second argument of `listen` (in this case, the `console.log`) is a callback function that is invoked when the listener is initialized.

## The M in MVC: **/models**
- The `models` folder houses JS files for the data or database that we are referencing.
  - These JS files may include:
    - Data
    - Functions to manipulate and return the data, such as CRUD operations
	- A `module.exports` line that allows the file to pass the data if it is called as a dependency
	  - **CONVENTION:** `module.exports` should consist only of functions that return the data rather than the data itself.
- **RELATIONSHIP TO APP:** Models JS files export functions that allow controllers to perform CRUD actions on the database.
- **CODE:** `module.exports = { `insert CRUD functions here` };`

## The V in MVC: **/views**
- The `views` folder houses EJS files (aka "views") for the page templates in your app.
- The files are organized with subpaths in nested folders inside the `views` folder.
  - Nesting is based on the path needed to get to the page.
  - **CONVENTION:** Views usually include:
    - `index.ejs` for main view for path
    - `show.ejs` for details view for the elements within the path
    - `new.ejs` for pages where new data entries are created
    - `edit.ejs` for forms where database elements are updated or edited
- **RELATIONSHIP TO APP:** Views are targets of `res.render` calls in the controller files; views may be passed context objects when this happens.
- EJS files (views)
  - Views are set up and work similar to HTML files.
	- One big difference is that views can embed JavaScript code inside.
	  - To insert JS code that is not printed, wrap the code snippet with `<%` and `%>`.
	  - To insert JS code to be printed, wrap what needs to be printed with `<%=` and `%>`.
  - To run code, views will likely need JS variables, which should be passed in as a context object in the `render` function in the controller.
	  - The properties inside the context object serve as local variables for the view.
  - To include CSS files, a static asset module, like `express.static`, is needed. After this type of module is installed, CSS files can be linked in the EJS file via `/css/style.css`.
  - Links and form actions to other addresses require a starting slash for relative paths.
    - Forms that perform HTTP methods should have the following attributes:
      - **CODE:** `method="GET"` for GET requests, or `method="POST"` for POST, PUT, PATCH, and DELETE requests.
      - **CODE:** `action="(URI endpoint here)"`
        - For PUT, PATCH, and DELETE actions, you will need to append `?_method=(action here)` to the URI, before the string close. `_method` in this case calls the `method-override` module.
          - For example, a sample DELETE URI might have the attribute `action="/items/<%= id %>?_method=DELETE"`.
        - **CONVENTION:** Different combinations of HTTP methods and URI endpoints will conventionally signify the responses to certain requests. See [this guide](https://gist.github.com/myDeveloperJourney/dfb5b8728c54fce5e0e997ac3ce466a0) for combinations.
    

## The C in MVC: **/controllers**
- The `controllers` folder houses "controller" JS files that define the functions that respond to HTTP requests.
  - Controllers are also known as "route handlers".
- Controller files do not need to be nested.
- **RELATIONSHIP TO APP:** Controllers form the heart of your app and interface with many of the files in your app.
  - Controllers **export** their functions to the router modules.
    - **CODE:** `module.exports = {` insert route handlers here `};`
  - Controllers **import** models files to perform CRUD operations on the database.
    - **CODE:** `const ModelFileName = require("../models/modelFileName");`
    - **CONVENTION:** The variable for the import is written as UpperCamelCase.
  - Controllers direct the app to **render** view templates and can pass context objects to the templates.
- **CONVENTION:** The functions inside controllers are named after their CRUD operations.
  - These functions take in `req` and `res` as parameters.
  - For "read" or "show" operations, use `res.render` and link to the view template.
    - **CONVENTION:** `res.render` will, by default, start from the `views` folder in the app, so addresses passed as the first argument to `render` need to be in relation to `views`. Addresses to relative paths in controller JS files do not need starting slashes.
    - `res.render` will accept a context object to be passed to the view as a second argument.
      - To add request-based data to the context object, call methods on `req`, such as `req.params.id`.
  - **CONVENTION:** For "update", "create", and "delete" operations, include `res.redirect` to bring the user back to a relevant page after the operation is completed.
    - **CONVENTION:** Just as in `res.render`, the argument to `res.redirect` will also be in relation to the views folder. Once again, addresses to relative paths in controller JS files do not need starting slashes.

## The router modules: **/routes**
- The `routes` folder contains "router module" JS files that direct the HTTP requests to the proper controller functions.
- Router modules do not need to be nested.
- Router modules need to import express and initialize a `router` variable using Express's `Router` method.
  - **CODE:** `const express = require("express");`
  - **CODE:** `const router = express.Router();`
  - The `router` variable will be used to map the requests to the controller functions.
- **RELATIONSHIP TO APP:** Router modules import functions from controller files and export the `router` variable to `server.js`.
  - **CODE (beginning of file):** `const pathCtrl = require("../controllers/path");`
    - Addresses to the relative paths of controller JS files do not need starting slashes.
  - **CODE (end of file)**: `module.exports = router;`
- Router modules map the **HTTP method** at a certain **URI endpoint** to a CRUD **controller function**.
  - **CODE (general format of route):** `router.httpMethod(endpoint, pathCtrl.ctrlFunction);`
    - HTTP methods include `get`, `post`, `put`, `patch`, and `delete`.
      - `put`, `patch`, and `delete` may need a methodOverride module to work with HTML5. Alternatively, you can use `post` with an "update" or "delete" endpoint.
    - **CONVENTION:** Different combinations of HTTP methods and URI endpoints will conventionally map to certain controller functions. See [this guide](https://gist.github.com/myDeveloperJourney/dfb5b8728c54fce5e0e997ac3ce466a0) for combinations.
    - The base path of the URI for a given router module was already specified when module was mounted in the `server.js` file. The URIs should be in relation to the subpath, NOT the root, and do require starting slashes.

## Code Testing
- Remember to save all of your files, especially `server.js`.
- Use Nodemon to automatically shutdown and restart your server after you save the entry point.
  - Run `nodemon` in the terminal to start Nodemon.
- Once all of your files are saved, go to http://127.0.0.1:3000/ or http://localhost:3000/ to test your application.
