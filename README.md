# Starting an Express app using MVC

## First steps to making the app

- Create a server.js file inside your new project folder
- Run `npm init -y` to start a node project and automatically add a `package.json `file
- Run `npm i express ejs` to install the Express and EJS modules to the project

## The entry point: **the `server.js` file**

1. *Load NPM or custom modules*
   - NPM modules are loaded using the `require` keyword
     - **CONVENTION:** Use the module name for the variable name
     - **CODE:** `const express = require("express");`
   - Custom modules like routers also need to be loaded here
     - **CONVENTION:** Use `pathRouter` for the module name for a given `path`
     - Routers are stored in a folder within root, so the argument to `require` is `./routes/[path]`
     - **CODE:** `const indexRouter = require("./routes/index");`

2. *Initialize the Express app*
   - Express is initialized using the `express` function with no arguments
     - `express` is a constructor function that returns an Express object
     - **CONVENTION:** Use `app` for the variable name of the new Express object
     - **CODE:** `const app = express();`
     
3. *Configure application settings*
   - Application settings are configured using the `set` method of `app`
     - View engines like EJS are loaded here
     - **CODE:** `app.set("view engine", "ejs");`

4. *Mount application middleware*
   - Middleware is mounted to the application using the `use` method of `app`
     - **CODE:** `app.use()`

5. *Mount route handlers*
   - Links to the route handlers are mounted using the `use` method of `app`
   - The first argument of `use` is the path from the `routes` folder to the page
   - The second argument is the `pathRouter` variable from the `require` statement in #1
   - **CODE:** `app.use("/", indexRouter);` OR `app.use("/subpath", subpathRouter);`

6. *Tell the application to listen on a port for requests*
   - The port will be used to run the Node process
   - Ports must be unique
   - **CONVENTION:** Use `port 3000` for Express applications
   - **CODE:** `app.listen(3000, function() { console.log("Express is listening on port 3000") });`
     - The first argument is the number of the port for your app, usually `3000`
     - The second argument of `listen` (in this case, the `console.log`) is a callback function that is invoked when the listener is initialized

## The M in MVC: **/models**
- The `models` folder houses JS files for the data or database that we are referencing
  - These JS files may include:
    - Data
    - Functions to manipulate and return the data
	- A `module.exports` line that allows the file to pass the data if it is called as a dependency
	  - **CONVENTION:** `module.exports` should consist only of functions that return the data rather than the data itself
- To connect the models to the application, the `models` JS files will be linked to the controllers that interface with the data

## The V in MVC: **/views**
- The `views` folder houses EJS files for the page templates in your app
- The files are organized with subpaths in nested folders inside the `views` folder
  - Nesting is based on the path needed to get to the page
    - EJS files usually include:
      - `index.ejs` for main page for path
      - `show.ejs` for details page for the elements within the path
  - The EJS files are linked to the app through the controller files
- EJS files
  - EJS files are set up and work similar to HTML files
	- One big difference is that EJS files can embed JavaScript code inside
	  - To insert JS code that is not printed, wrap the code snippet with `<%` and `%>`
	  - To insert JS code to be printed, wrap what needs to be printed with `<%=` and `%>`


## The C in MVC: **/controllers**

## The listeners: **/routes**
	• Normal routing
		○ Has non-nested JS files for the routes
		○ Linked to the server.js by
			§ Including the files as modules at the top
			§ Mounted using app.use( path, routerVar )
		○ Initialized by including/requiring Express and Express's Router() method
			§ const express = require("express");
			§ const router = express.Router();
		○ Links to controller using const pathCtrl = require( "../controllers/path" );
		○ Defines the controller response for a request
			§ For example, router.get("/", pathCtrl.index);
			§ Because we loaded the path in the app.use line in server.js, we are already at the proper path - don't need to go deeper
		○ Export the "router" Router object
	• Show routing
	Set of instructions that tells our server what to do when it receives a request
		○ Maps HTTP requests to code
		○ An example of a request is a "GET" request
			§ app.[http method]( route, callback(request, response) { } );
				□ If route is "/", it's called a root route and points to the domain
				□ Methods of request include .query
					® Request methods gather data about the request itself
					® Query will return the entire query string as object
				□ Methods of response include .send, .render, .redirect, etc
					® Send will send a message into the page
					® Render will render an EJS file from the views folder
					® Redirect will redirect to a path
		○ Route handlers are like front-end event handlers, but for server requests
			§ But route handlers take in more information than event handlers do

## Code Testing
- Remember to save all of your files, especially `server.js`
- Use Nodemon to automatically shutdown and restart your server after you save the entry point
  - Run `nodemon` in the terminal to start Nodemon
- Once all of your files are saved, go to http://127.0.0.1:3000/ or http://localhost:3000/
