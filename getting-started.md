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
   

Test eslint configuration

    vim server.js
   console.log('Hello')
   
  
 
 ## Setup web server configuration
 
 Two main production dependencies
 
    yarn add express ejs
   
 
 config.js
 
    module.exports = {
      'port': process.env.PORT || 8080
    }
 server.js
 
    const epxress =   require('express');
    const config = require('./config')
    const app = express();
    app.use(express.static('public'));
    
    app.listen(config.port, function listenHandler(){
      console.info(`Running on ${config.port}`);
    );
    
    
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


Add jsx suppot
---

Add babel to understand jsx


vim package.js


    "babel":{
      "presets": ["react", "env", "stage-2"]
     }


Add in production so that you can run commands on production servers
(depends on prod packaging/deployment flow)

    yarn add  babel-cli babel-preset-react babel-preset-env babel-preset-stage2
    
Add babel node 

    ...
    "dependencies
    
    "scripts": {
      "dev": "pm2 start lib/server.js --watch --interpreter babel-node"
      ...
      

start again

    yarn dev