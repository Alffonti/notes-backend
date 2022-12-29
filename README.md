# Notes - Backend

This repository aims to create the backend of a **Notes application** using NodeJs.

The **express** library was installed to build the backend server.

The **nodemon** package was installed to automatically restart the application everytime a file changes.

The **Visual Studio Code REST client plugin** was installed to test the HTTP requests. A `requests` directory at the root of the backend application was added containing REST client requests as files ending with the .rest extension.

## Serving static files from the backend

The **static** middleware is used to serve the static files (production build of the frontend) from the backend.

## Deployment

The application was deployed to **Cyclic**. It was chosen over Render and Fly.io because Apps do not have to sleep, wake up, spin up or recycle even on free tier. And no credit card is needed.

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
- Espress
- Visual Studio Code REST client plugin
- Cyclic

## Resources

- [How to Deploy Your Node.js Application for Free with Render - FreeCodeCamp](https://www.freecodecamp.org/news/how-to-deploy-nodejs-application-with-render/)

- [Serving static files in Express](https://expressjs.com/en/starter/static-files.html)
