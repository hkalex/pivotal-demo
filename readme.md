```
npm init -y
npm install express
```


Create file `app.js`
```
var express = require('express')
var app = express()

var env = process.env;
 
app.get('/', function (req, res) {
  console.log('Get a new request ' + (new Date()).toISOString());
  res.set('Content-Type', 'application/json');
  res.send(JSON.stringify(env));
})

app.listen(process.env.PORT || 3000)
```

```
cf login
cf push alex-nodejs -c "node app.js" -m 256M -b nodejs_buildpack
cf logs alex-nodejs
cf scale alex-nodejs -i 2 -m 128M
```

Adding service
```
cf marketplace
cf marketplace -s cleardb
cf create-service cleardb spark alex-db
cf bind-service alex-nodejs alex-db
cf env alex-nodejs
```

Connect to DB
```
npm install mysql --save
```

Change Code
```
var express = require('express')
var mysql = require('mysql')
var app = express()

var env = process.env;

var dbConfig = JSON.parse(env.VCAP_SERVICES).cleardb[0].credentials;

app.get('/', function (req, res) {
  console.log('Get a new request ' + (new Date()).toISOString());
  var connection = mysql.createConnection({
    host     : dbConfig.hostname,
    user     : dbConfig.username,
    password : dbConfig.password,
    database : dbConfig.name
  });

  connection.connect();

  connection.query('SELECT 1 + 1 AS solution', function (error, results, fields) {
    if (error) throw error;
    console.log('The solution is: ', results[0].solution);
    res.send(JSON.stringify(results[0].solution));
  });

  connection.end();
})

app.listen(process.env.PORT || 3000)

```




Deleting the application
```
cf delete alex-nodejs
```


