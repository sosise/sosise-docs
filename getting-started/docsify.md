# Project documentation
Often developing projects, it is necessary to document their functions in detail. Sosise offers this functionality out of the box.

> Sosise uses [docsify](https://docsify.js.org/#/?id=docsify) library. Please refer to it for full documentation.

## How it works
First of all take a look at the documentation config at `src/config/documentation.ts`. By default the documentation URL `/docs` is protected with `basic authentication`, if you want to use it you will need to provide basic auth user and password in your `.env` file, or disable it.

After configuration just start your application by executing `npm run serve` and go to url `http://127.0.0.1:10000/docs`.

## How to edit the documentation
In your project you will find `docs` folder with some example files in it:

- `_sidebar.md` documentation sidebar
- `index.html` main HTML file which renders docsify, you may specify some variables in it, like your `project name`
- `README.md` first page of the documentation

## Routing
The `/docs` route is specified in your default routing file `src/routes/api.ts` you can completely delete the documentation route or change the url.
