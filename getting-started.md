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
              { test: /\.js$/,excloude: /node_modules/, use: 'babel-loader' }
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

vim webpack.config

        ...
        entry: ['babel-polyfill', './lib/components/Index.js']
        ...

