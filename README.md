# Notes - Backend

This repository aims to create the backend of a **Notes application** using NodeJs.

The **express** library was installed to build the backend server.

The **nodemon** package was installed to automatically restart the application everytime a file changes.

The **Visual Studio Code REST client plugin** was installed to test the HTTP requests. A `requests` directory at the root of the backend application was added containing REST client requests as files ending with the .rest extension.

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

## MongoDB

The **MongoDB Atlas** cloud data platform is used to built the MongoDB database.

The **Mongoose** library was installed to provide schema validation, and to map objects in the code into documents in MongoDB.

The communication between the backend and MongoDB was extracted into its own module `model/note.js`

The `findbyId()`, `findByIdAndDelete()`, and `findByIdAndUpdate()` methods were used to get a note, delete a note and update a note, respectively.

Errors are handled by the **error-handler middleware**. E.g.: if the id query parameter is invalid, the error handler will send a response to the browser with the response object passed as a parameter.
```
{ error: 'malformatted id' }
```

## Validation

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

The validation errors are handled by the error hanlder:

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


The following error is sent to the browser if the validation when creating a note with less than 5 characters in the content field.

```
{
  "error": "Note validation failed: content: Path `content` (`Mor`) is shorter than the minimum allowed length (5)."
}
```

When using findOneAndUpdate method (editing a note), Mongoose doesn't automatically run validation. To trigger this, a configuration object need to be passed in the third (options) parameter.

```
{ runValidators: true, context: 'query' }
```

## Enviroment variables

A `.env` file was created at the root of the project, after installing the **dotenv** library, to define environment variables in development mode.

The environment variables defined in the .env file can be taken into use with the expression `require('dotenv').config()` and can be referenced in the code with the `process.env` syntax.

The environment variables were defined direclty in the Cyclic dashboard to reference those variables in production mode.

## Tools

- VS Code
- Node
- Espress
- Visual Studio Code REST client plugin
- MongoDB Atlas
- Mongoose
- Cyclic

## Resources

- [How to Deploy Your Node.js Application for Free with Render - FreeCodeCamp](https://www.freecodecamp.org/news/how-to-deploy-nodejs-application-with-render/)

- [Serving static files in Express](https://expressjs.com/en/starter/static-files.html)

- [Validation - Mongoose](https://mongoosejs.com/docs/validation.html)
