# Notes - Backend

This repository aims to create the backend of a **Notes application** using NodeJs.

The **express** library was installed to build the backend server.

The **nodemon** package was installed as a development dependency to automatically restart the application everytime a file changes.

The **Visual Studio Code REST client plugin** was installed to test the HTTP requests. A `requests` directory at the root of the backend application was added containing REST client requests as files ending with the .rest extension.

## Directory structure

This project separates the different responsabilites into separate modules.

```
├── index.js
├── app.js
├── build
│   └── ...
├── controllers
│   └── notes.js
├── models
│   └── note.js
├── package-lock.json
├── package.json
├── utils
│   ├── config.js
│   ├── logger.js
│   └── middleware.js
```

The `controllers` directory defines the route handling functions.
The `models` directory defines the Mongoose schema for notes.
The `utils` directory defines the environment variables, middlewares, and the logging functions.
The `index.js` file imports the actual application from the `app.js` file and then starts the application.
The `app.js` file establishes the connection to the database and loads the middlewares to the application.

## Database

The **MongoDB Atlas** cloud data platform is used to built the MongoDB database.

The **Mongoose** library was installed to provide schema validation, and to map objects in the code into documents in MongoDB.

The communication between the backend and MongoDB was extracted into its own module `model/note.js`.

The `findbyId()`, `findByIdAndDelete()`, and `findByIdAndUpdate()` methods were used to get, delete, and update a note, respectively.

Errors are handled by the **error-handler middleware**. E.g.: if the id query parameter is invalid, the error handler will send a response to the browser with the response object passed as a parameter.
```
{ error: 'malformatted id' }
```

## Data validation

The format of the data that is stored in the application's database is constrained by specific validation rules defined for each field in the schema.

```
const noteSchema = new mongoose.Schema({
  content: {
    type: String,
    minLength: 5,
    required: true
  },
  date: {
    type: Date,
    required: true
  },
  important: Boolean
})
```

The potential errors originated when breaking the constraints while creating a note are passed to the error handler middleware using the `next(error)` function.

The validation errors are handled by the error handler:

```
const errorHandler = (error, request, response, next) => {
  console.error(error.message)

  if (error.name === 'CastError') {
    return response.status(400).send({ error: 'malformatted id' })
  } else if (error.name === 'ValidationError') {
    return response.status(400).json({ error: error.message })
  }

  next(error)
}
```


The following error is sent to the browser if a note is created with less than 5 characters in the content field.

```
{
  "error": "Note validation failed: content: Path `content` (`Mor`) is shorter than the minimum allowed length (5)."
}
```

To run validations when editing a note, the following configuration object need to be passed in the third (options) parameter of the `findOneAndUpdate()` method (as Mongoose doesn't automatically run validations on the `findOneAndUpdate()` method).

```
{ runValidators: true, context: 'query' }
```

### Linting

The **ESlint** package was installed as a development dependency.

The ESlint default configuration was run with the command:

```
npx eslint --init
```

After answering the questions, the `.eslintrc.js` configuration file was created.

A npm script was created to check every file in the project by ESlint.

```
{
  // ...
  "scripts": {
    "start": "node index.js",
    // ...
    "lint": "eslint ."
  },
  // ...
}
```

The `build` directory was ignored by ESlint by creating a `.eslintignore` file in the project's root.

The VSCode ESlint plugin was installed in order to run the linter continuously and see errors (which are underlined with a red line) in the code immediately.

## Enviroment variables

The `.env` file was created at the root of the project, after installing the **dotenv** library, to define environment variables in development mode.

The environment variables defined in the `.env` file can be taken into use with the expression `require('dotenv').config()` and can be referenced in the code with the `process.env` syntax.

The environment variables were defined direclty in the Cyclic dashboard to reference those variables in production mode.

## Serving static files from the backend

The **static** middleware is used to serve the static files (production build of the frontend) from the backend.

## Deployment

The application was deployed to **Cyclic**. It was chosen over Render and Fly.io because Apps do not have to sleep, wake up, spin up or recycle even on free tier. And no credit card is needed.

The node/express-backend resides in the Cyclic server. When the root address (https://notes-backend.cyclic.app/) is accessed, the browser loads and executes the React app that fetches the json-data from the Cyclic server.

## Streamlining deploying of the frontend

The following npm-scripts were added to the `package.json` in order to create a new production build of the frontend from the CLI.

```json
{
  "scripts": {
  //...
	"build:ui": "rm -rf build && cd ../notes-frontend/ && npm run build && cp -r build ../notes-backend",
	"deploy": "git push origin main",
	"deploy:full": "npm run build:ui && git add . && git commit -m uibuild && npm run deploy",
  }
}
```

## Tools

- VS Code
- Node
- Express
- Visual Studio Code REST client plugin
- MongoDB Atlas
- Mongoose
- ESlint
- Git
- GitHub
- Cyclic

## Resources

- [How to Deploy Your Node.js Application for Free with Render - FreeCodeCamp](https://www.freecodecamp.org/news/how-to-deploy-nodejs-application-with-render/)

- [Serving static files in Express](https://expressjs.com/en/starter/static-files.html)

- [Validation - Mongoose](https://mongoosejs.com/docs/validation.html)

- [ESlint Docs](https://eslint.org/)
