# Notes - Backend

This repository aims to create the backend of a **Notes application** using NodeJs.

The **express** library was installed to build the backend server.

The **nodemon** package was installed as a development dependency to automatically restart the application everytime a file changes.

The backend application has been implemented in compliance with the RESTful conventions. E.g.: a note is created by making an HTTP POST request to the note path.

The **express-async-errors** library was installed to handle async functions so that errors are catched when they occurs and passed to the error-handling middleware. By doing so, the `try...catch` statements are not needed, keeping the code clean and uncluttered.

The **Visual Studio Code REST client plugin** was installed to test the HTTP requests. A `requests` directory at the root of the backend application was added containing REST client requests as files ending with the .rest extension.

## Directory structure

 This repository separates the different responsabilites into separate modules and is structured as follows:

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

## Logging

The logger functions were defined in the `utils/logger.js` file and are only run in production and development mode in order to avoid obstructing the test execution output when testing the application.

```javascript
const info = (...params) => {
  if (process.env.NODE_ENV !== 'test') {
    console.log(...params)
  }
}
```

## Linting

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

## Testing

The Jest library was installed as a development dependency in order to test the backend application.

The Jest global variable was added to the `env` property in the `.eslintrc.js` file.

The following npm-script was added in order to execute the test serially and display individual test results with the test suite hierarchy. The `-`-forceExit` and `--detectOpenHandles` were added to avoid errors during the execution of the test.

```
"test": "NODE_ENV=test jest --verbose --runInBand --forceExit --detectOpenHandles"
```

The -t option can be used for running tests with a specific name:
```
npm test -- -t "notes are returned as json"
```

Individual test cases were defined in files ending with the `test.js` extension in the `tests` directory.

The **Supertest** package was installed to test endpoints and routes when making HTTP requests to the backend.

The Express application imported form the `app.js` module was passed to the function imported from the supertest package, which converts it into a **superagent object** where some methods (such as `get`, `post`, `send`, `expect`) can be used to test the HTTP responses.

```
const supertest = require('supertest')
const app = require('../app')
const api = supertest(app)
```

An ephemeral port is started by Supertest during the tests as the server is not listening for HTTP connections made from the test files.

A separate database address is used (defined in the `.env` file ) when running the test.

The `test_helper.js` file contains the initial test database state and other functions (fetching notes stored in the database, etc).

The `afterAll` method was used to close the database connection used by Mongoose once all the tests are finished running.
```
afterAll(() => {
  mongoose.connection.close()
})
```

The `beforeEach` method was used to clear out and initialize the database with the same state before every test is run. The `Promise.all` method was used to wait until every promise for saving a note is finished (meaning that the database has been initialized) before ending the execution of the callback function passed as parameter to the `beforeEach` method .
```
beforeEach(async () => {
  await Note.deleteMany({})

  const noteObjects = helper.initialNotes.map(note => new Note(note))
  const promiseArray = noteObjects.map(note => note.save())

  await Promise.all(promiseArray)
})
```

The `beforeAll` method was used when testing the addition and deletion of a note in order to generate a valid token for a newly created user. The token can be referenced in each of those tests.

```
  let token = null
  beforeAll(async () => {
    await User.deleteMany({})

    const passwordHash = await bcrypt.hash('sekret', 10)
    const user = new User({
      username: 'root',
      passwordHash,
    })

    const savedUser = await user.save()

    const userForToken = { username: savedUser.username, id: savedUser._id }

    token = jwt.sign(userForToken, process.env.SECRET, {
      expiresIn: 60 * 60,
    })

    return token
  })
  ```

## User administration

Deleting and editing a note is only allowed for the user who created it.

**References** are stored in both documents: each **note** references the user who created it, and each **user** has an array of references to all of the notes created by him/her.

The `notes` field was added to the Mongoose schema for users to store an array of IDs representing each note created by a user.

The schema of the user was defined in the `models/user.js` file. The `notes` field stores an array of IDs representing each note created by a user.

The type of the field was set to `ObjectId` in order to specify that the data stored is a reference to a `note` document.

```javascript
const userSchema = new mongoose.Schema({
  //
  notes: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Note'
    }
  ],
})
```

The schema of the note defined in the `models/note.js` file was updated so that it contains information about the user who created it:

```javascript
const noteSchema = new mongoose.Schema({
 //
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
})
```

The **bcrypt** package was installed to generate **password hashes**. When a user is created, his/her password hash is stored in the database instead of the user' password. When the user log into the application, through the `/api/login` router, the backend application compares the password to the password hash stored in the user's document.

Some validations were added into the user creation:
- username must be unique,
- username must be long enough,
- username must consist of permitted characters,
- password must be strong enough.

```javascript
usersRouter.post('/', async (request, response) => {
  //
  const existingUser = await User.findOne({ username })
  if (existingUser) {
    return response.status(400).json({
      error: 'username must be unique'
    })
  }
})
```

The `populate()` method was used in order to show the contents (`date` and `content` fields) of the users' notes when an HTTP GET request is made to the `/api/users` route.

### User authentication

A token must be sent in the Authorization header in order to create a new note.

The **jsonwebtoken** package was installed in order to implement JSON Web Tokens.

The token is sent to the client once the user login to the application.

```javascript
const token = jwt.sign(userForToken, process.env.SECRET, {
  expiresIn: 60 * 60,
})
```

The secret key is stored in the .env file.

The token expiration time was set to 1 hour.



## Enviroment variables

The execution mode of the application environment variable is defined in the following npm-scripts: `start`, `dev, and `test` which set the NODE_ENV variable to production, development and test, respectively.

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
- Jest
- Supertest
- Bcrypt
- Jsonwebtoken
- Git
- GitHub
- Cyclic

## Resources

- [How to Deploy Your Node.js Application for Free with Render - FreeCodeCamp](https://www.freecodecamp.org/news/how-to-deploy-nodejs-application-with-render/)

- [Serving static files in Express](https://expressjs.com/en/starter/static-files.html)

- [Validation - Mongoose](https://mongoosejs.com/docs/validation.html)

- [ESlint Docs](https://eslint.org/)

- [Jest Docs](https://jestjs.io/docs/getting-started)

- [Supertest repository](https://github.com/ladjs/supertest)
