# Build React application

Build React application following setup from https://jscomplete.com/reactful or https://jscomplete.com/learn/1rd-reactful

## Intilialise the npm package

    $ npm init -y

## Initialise the Git repo

    $ git init

Probably is a good idea to ignore the npm modules when commiting in git.

    $ echo "node_modules/" >> .gitignore

## Install Express

    $ npm i express

## Install React

Install the front end dependencies - React and React-DOM.

    $ npm i react react-dom

## Install & Configure Babel

Install Babel to transform React's JSX code into React's API calls and modern JavaScript features into code that can be understood by any browser.

    $ npm i babel-loader @babel/core @babel/node @babel/preset-env @babel/preset-react

    $ cat >> babel.config.js <<EOF
    module.exports = {
      presets: ['@babel/preset-env', '@babel/preset-react'],
    };
    EOF

## Install Webpack and configure Babel loader

Install Webpack, a module bundler to translate react modules into something that works in all browsers.

    $ npm i webpack webpack-cli
    $ cat >> webpack.config.js << EOF
    module.exports = {
      module: {
        rules: [
          {
            test: /\.js$/,
            exclude: /node_modules/,
            use: {
              loader: 'babel-loader',
            },
          },
        ],
      },
    };
    EOF

Run Webpack in watch mode (-w) and generate a development-friendly bundle (-d).

In production Webpack should be run with -p rather
than -d

    sed -i '' 's/"scripts": {/"scripts": {\'$'\n    "dev-bundle": "webpack -w -d",/1' package.json

## Install ESLint and dependencies for Babel and React

    $ npm i -D eslint babel-eslint eslint-plugin-react eslint-plugin-react-hooks
    $ cat >> .eslintrc.js << EOF
    module.exports = {
      parser: 'babel-eslint',
      env: {
        browser: true,
        commonjs: true,
        es6: true,
        node: true,
        jest: true,
      },
      parserOptions: {
        ecmaVersion: 2020,
        ecmaFeatures: {
          impliedStrict: true,
          jsx: true,
        },
        sourceType: 'module',
      },
      plugins: ['react', 'react-hooks'],
      extends: [
        'eslint:recommended',
        'plugin:react/recommended',
        'plugin:react-hooks/recommended',
      ],
      settings: {
        react: {
          version: 'detect',
        },
      },
      rules: {
        // You can do your customisations here...
        // For example, if you don't want to use the prop-types package,
        // you can turn off that recommended rule with: 'react/prop-types': ['off']
      },
    };
    EOF

## Install and configure Jest to be able to test React

    $ npm i -D jest babel-jest react-test-renderer
    $ sed -i '' 's/"test": "echo \\"Error: no test specified\\" && exit 1"/"test": "jest",/1' package.json

## Install Nodemon to restart node after every file change automatically

    $ npm i -D nodemon
    $ sed -i '' 's/"scripts": {/"scripts": {\'$'\n    "dev-server": "nodemon --exec babel-node src\/server\/server.js --ignore dist\/",/1' package.json

## Setup the directories

    $ mkdir dist && touch dist/main.js
    $ mkdir src && touch src/index.js
    $ mkdir src/server && touch src/server/server.js
    $ mkdir src/components && touch src/components/App.js

## Create a sample app

### src/index.js

    $ cat >> src/index.js << EOF
    import React from 'react';
    import ReactDOM from 'react-dom';

    import App from './components/App';

    ReactDOM.hydrate(
      <App />,
      document.getElementById('mountNode'),
    );
    EOF

### src/components/App.js

    $ cat >> src/components/App.js << EOF
    import React, { useState } from 'react';

    export default function App() {
      const [count, setCount] = useState(0);
      return (
        <div>
          This is a sample stateful and server-side
          rendered React application.
          <br />
          <br />
          Here is a button that will track
          how many times you click it:
          <br />
          <br />
          <button onClick={() => setCount(count + 1)}>{count}</button>
        </div>
      );
    }
    EOF

### src/server/server.js

    $cat >> src/server/server.js << EOF
    import express from 'express';
    import React from 'react';
    import ReactDOMServer from 'react-dom/server';
    import App from '../components/App';

    const server = express();
    server.use(express.static('dist'));

    server.get('/', (req, res) => {
      const initialMarkup = ReactDOMServer.renderToString(<App />);

      res.send(`
        <html>
          <head>
            <title>Sample React App</title>
          </head>
          <body>
            <div id="mountNode">${initialMarkup}</div>
            <script src="/main.js"></script>
          </body>
        </html>
      `)
    });

    server.listen(4242, () => console.log('Server is running...'));
    EOF

## Run the application

To run type the following in two terminals:

    $ npm run dev-server
    // Run the bundle which will create dist/main.js
    $ npm run dev-bundle

Then visit http://localhost:4242/
