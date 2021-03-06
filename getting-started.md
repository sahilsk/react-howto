Setting up the folder
-

    mkdir advanced-react
    cd advanced-react
    mkdir lib
    touch lib/server.js


Initialize the repo

    yarn init 
 
 Add eslint (developmnet dependency)
 
     yarn add --dev eslint
     yarn eslint -- --init
  
  
  Add `.eslintrc.js`
  
      yarn add --dev eslint-plugin-react babel-eslint
  
  
      module.exports = {
          "parser": 'babel-eslint',
          "env": {
            "browser": true,
            "commonjs": true,
            "es6": true,
            "node": true,
            "jest": true,
          },
          "extends": ["eslint:recommended", "plugin:react/recommended"],
          "parserOptions": {
            "ecmaFeatures": {
              "experimentalObjectRestSpread": true,
              "jsx": true
            },
            "sourceType": "module"
          },
          "plugins": [ "react" ],
          "rules": {
            "react/prop-types": ["off"],
            "indent": ["error", 2],
            "linebreak-style": ["error","unix"],
            "quotes": ["error","single"],
            "semi": ["error","always"],
            "no-console": ["warn", { "allow": ["info", "error"] }]
          }
        };

Test eslint configuration

    vim server.js
   console.log('Hello')
   
  
 
 ## Setup web server configuration
 
 Two main production dependencies
 
    yarn add express ejs
   
 
vim lib/config.js
 
    module.exports = {
      'port': process.env.PORT || 8080
    }
    
 server.js
 
    const express =   require('express');
    const config = require('./config');
    const app = express();
    app.set('view engine', 'ejs');

    app.use(express.static('public'));

    app.get('/', (req, res) =>  {
      res.render('index', {'answer': 42 });
    });

    app.listen(config.port, function listenHandler() {
      console.info(`Running on ${config.port}...`);
    });
    
    
Add templating
    
        mkdir views
        
server.js

      ....
      app.use('view engine', 'ejs');
      ...
      app.get('/', (req, res) =>  {
        res.render('index', {answer: 42 });
      })
      ...
 
touch views/index.ejs

       <h2> Hello express and ejs -- <%= answer %> </h2>
   

fire 'localhost:8080'


Add pm2
--

Why pm2?
Zero downtime restart and cluster module support

    yarn add pm2
    

vim package.json

    '''
    'script': {
      "dev": "pm2 start lib/server.js --watch"
      }
     '''
 
 Start the process now
 
    yarn dev

Start logs

    yarn pm2 logs
    
> Focus on Green lines. Blue lines are logs from pm2.


Add babel to add jsx, class, import support
---

Add babel to understand jsx and 'import' statements and class properties(stage-2)

vim lib/server.js

    import express from 'express';
    import config from './config';

vim package.js


    "babel":{
      "presets": ["react", "env", "stage-2"]
     }


Add in production so that you can run commands on production servers
(depends on prod packaging/deployment flow)

    yarn add babel-cli babel-preset-react babel-preset-env babel-preset-stage-2
    
Add babel node 

    ...
    "dependencies
    
    "scripts": {
      "dev": "pm2 start lib/server.js --watch --interpreter babel-node"
      ...
on windows
    "scripts": {
      "dev": "nodemon --exec babel-node .\lib\server.js"
      ...

      

start again

    yarn dev
    

Adding React
---

Create all react app in `lib` directory

        mkdir lib/components
        

vim lib/components/Index.js

For 'import' to work you need babel polyfills.

        import  React from 'react';
        import ReactDOM from 'react-dom';

        const App = () => {
          return (
            <h2> Hello React </h2>
          );
        };
        ReactDOM.render(<App/>, document.getElementById('root'));


vim index.ejs

        ...
        
        <body>
        ..
        <div id='root'>
        Loading...
        </div>
        ...
        <script src="./dist/bundle.js" charset="utf-8"> </script>


bundle.js << Enter Webpack


Webpack setup
---

        yarn add react react-dom webpack


Where to start and place bundle.js?

Visit webpack.js.org


vim webpack.config.js

       const path = require('path');

        const config = {
          entry: './libs/components/Index.js',
          output: {
            path: path.resolve(__dirname, 'public'),
            filename: 'bundle.js'
          },
          module: {
            rules: [
              { test: /\.js$/, use: 'babel-loader' }
            ]
          }
        };
        module.exports = config;


Add babel-loader

        yarn add babel-loader
        
        
Edit package.json

        "scripts": {
        ...
            "webpack": "webpack -wd"   
            }
         ...

`w` watch files
`d`: development mde


start again and it should generate bundle.js

        yarn webpack

Improve performance: Ignore node_modules in the webpack configuration
        
        
            rules: [
              { test: /\.js$/,exclude: /node_modules/, use: 'babel-loader' }
            ]
        

Class components instead of function components
--

``` js
class App extends React.Componet {
    state = {
        answer: 42
        };
    asyncFunc = () =>{
        return Promise.resolve(37);
     };
     
     async componentDidMount() {
        this.setState({
            anwer await this.asyncFunc()
        })
     }
     
     render(){
        return (
            <h2> Hello Class Components -- {this.state.answer} </h2>
          );
      }
}
```

Need webpack polyfill for `async-await` to work

    
    yarn add babel-polyfill

vim webpack.config.js

        ...
        entry: ['babel-polyfill', './lib/components/Index.js'] 
        ...

Working with Data
---

mkdir ./state-api







For production
---

Separating vendor file: include 'CommonsChunkPlugin'  and require 'webpack'

vim webpack.config.js

        const path = require('path');
        const webpack = require('webpack');

        const config = {
          resolve: {
            modules: [path.resolve('./lib'), path.resolve('./node_modules')]
          },
          // entry: './libs/components/Index.js',
          entry: {
            vendor: [
              'babel-polyfill',
              'react',
              'react-dom',
              'prop-types',
              'axios',
              'lodash.debounce',
              'lodash.pickby'
            ],
            app: ['./lib/renderers/dom.js']
          },
          output: {
            path: path.resolve(__dirname, 'public'),
            filename: '[name].js'
          },
          module: {
            rules: [{
              test: /\.js$/,
              exclude: /node_modules/,
              use: {
                loader: 'babel-loader',
                options: {
                  presets: ['react', 'env', 'stage-2']
                }
              }
            }]
          },
          plugins: [
            new webpack.optimize.CommonsChunkPlugin({
              name: 'vendor'
            })
          ]
        };

        module.exports = config;


vim index.ejs

         ...
        <script src="./dist/vendor.js?version=1" charset="utf-8"> </script>
             ...
        <script src="./dist/app.js?version=1" charset="utf-8"> </script>



Now minify the bundle files

vim package.json

        
        ..
        "scripts": {...
        "build-webpack": "webpack -p",
        ...

Run this in production to generate minified bundled files

    yarn build-webpack
    


Using babel-node in prodution is bad idea

vim package.json

`copy-file`: copy non-js files also

Also, use cluster feature of pm2 to run in production.

        ...
        "scripts": {
        ...
         "build-node": "babel lib -d build --copy-files",
         "start-prod": "NODE_ENV=production NODE_PATH=./build pm2 start build/server.js -i max --name appProd"
         ..
         },

Also, insead of using `stage2`, we can be specific about our plugin list.

         "babel": {
            "presets": [
              "react",
              ["env", { "targets": { "node": "current" } }]
            ],
            "plugins": [
              "transform-class-properties",
              "transform-object-rest-spread"
            ]
          },
        ...

> async/await are supported by node natively. So, babel not required for that.

    $ yarn babel-node



vim webpack.config.js

Tell webapck to what to build for browser, not everything.

    ...
      module: {
        rules: [{
          test: /\.js$/,
          exclude: /node_modules/,
          use: {
            loader: 'babel-loader',
            options: {
              presets: ['react', 'env', 'stage-2']
            }
          }
        }]
      },
     ...


