# Project Documentation
During project development, it's often crucial to meticulously document various functions. Sosise comes equipped with this feature right out of the box.

> Sosise employs the [docsify](https://docsify.js.org/#/?id=docsify) library for documentation. Please refer to its detailed instructions for comprehensive knowledge.

## Documentation Process
Firstly, acquaint yourself with the documentation configuration in `src/config/documentation.ts`. By default, the documentation URL `/docs` employs `basic authentication`. To utilize this feature, ensure you provide the basic auth username and password in your `.env` file or choose to disable it.

Once configured, initiate your application by running `npm run serve` and navigate to the URL `http://127.0.0.1:10000/docs`.

## Documentation Editing
Within your project, you will find a `docs` folder containing several example files:

- `_sidebar.md` houses the documentation sidebar.
- `index.html` is the primary HTML file that renders docsify. Here, you can specify variables like your `project name`.
- `README.md` serves as the opening page of your documentation.

## Routing
The default routing file `src/routes/api.ts` specifies the `/docs` route. Feel free to modify or even eliminate the documentation route as needed.