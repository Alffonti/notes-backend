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

The **Mongoose** library was installed to provide schema validation, and to map objects in the code into documents in MongoDB.

The communication between the backend and MongoDB was extracted into its own module `model/note.js`

The `findbyId()`, `findByIdAndDelete()`, and `findByIdAndUpdate()` methods were used to get a note, delete a note and update a note, respectively.

Errors are handled by the **error-handler middleware**. E.g.: if the id query parameter is invalid, the error handler will send a response to the browser with the response object passed as a parameter.
```
{ error: 'malformatted id' }
```

## Enviroment variables

A `.env` file was created at the root of the project, after installing the **dotenv** library, to define environment variables during develpment mode.

The environment variables defined in the .env file can be taken into use with the expression `require('dotenv').config()` and can be referenced in the code with the `process.env` syntax.

The environment variables were defined direclty in the Cyclic dashboard to reference those variables during production mode.

## Tools

- VS Code
- Node
- Espress
- Visual Studio Code REST client plugin
- Cyclic

## Resources

- [How to Deploy Your Node.js Application for Free with Render - FreeCodeCamp](https://www.freecodecamp.org/news/how-to-deploy-nodejs-application-with-render/)

- [Serving static files in Express](https://expressjs.com/en/starter/static-files.html)
