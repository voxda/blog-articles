# Custom Prometheus Metrics and Easy Synthetic Monitoring!

In this article we will explore how to create custom prometheus metrics and easy synthetic monitoring using nodejs. We'll take advantage of the nodejs event loop and its asynch nature of processing. This is one of those time where we will allow our server side code to run without `await` statement within our API calls.  

**Prometheus** has several metrics types explained within their [documentation](https://prometheus.io/docs/concepts/metric_types/). Take some time to explore how simple prometheus is and it will open a world of capability to you.   

## GitHub Sample Code
The code for this article will be on GitHub at [synthetic-monitor](https://github.com/voxda/synthetic-monitor.git). Synthetic Monitor is a simple nodejs application that allows you to create custom logic around your monitoring urls and generates custom prometheus metrics using the [prom-client](https://www.npmjs.com/package/prom-client) from npm. 

## Tutorial

### Step 1: setup your project
1. Create a folder called synthetic-monitor
2. In a terminal change directory `cd synthetic-monitor`
3. Create a file `app.js` at the root folder synthetic-monitor.
4. Perform `npm init`. Accept all of the defaults when prompted until you are back at the command prompt.  

We are now ready to add some code!

### Step 2: Create our endpoints

We'll use [express.js](https://expressjs.com/) as our http server to expose our custom metrics endpoint. Express is a popuplar nodejs server but there are other options you could use.  

#### Import NPM Packages
Open the `app.js` we created earlier and add the following packages to it.  

```javascript
const express = require('express');
const prometheus = require('prom-client');
const Health = require('./services/health');

const app = express();
```

1. `prom-client` allows us to create custom metrics.
2. `express` is our http server
3. `Health` is a custom class will be creating within this tutorial
4. `const app = express();` creates an isntance of the express server that we will configure and add endpoints to. 

#### Add metrics endpoint

Add an endpoint called `/metrics`.  Within this method we will register our prometheus metrics collector. This endpoint will return our raw prometheus metrics that we will create soon. 

```javascript
app.get('/metrics', async (req, res) => {
    try {
        const metrics = await prometheus.register.metrics();
        res.set('Content-Type', prometheus.register.contentType);
        res.end(metrics);
    } catch (error) {
        res.status(500).json({ error: 'Internal server error' });
    }
});
```

#### Add Health Check class
Add a reference to our `Health` class that calls the `check()` method. I've added a `console.log()` statement for now to see when the method is called. We'll implement the details of the `Health` class in subsequent steps.  

**NOTE:** this code will not run until we implement the Health class.   

```javascript
Health.check();
console.log('health check started');
```

### Add express server startup
`app.listen()` allows us to start the express server on port 3001.  This port can be anything you want that doesnt conflict with other apps you might have running.  

```javascript
// Start the express app
app.listen(3001, () => {
    console.log('Server listening on port 3001');
});
```

For now, save this file and we come back to it later in the tutorial.  

The completed file should look like this:
```javascript
const express = require('express');
const prometheus = require('prom-client');
const Health = require('./services/health');

const app = express();

app.get('/metrics', async (req, res) => {
    try {
        const metrics = await prometheus.register.metrics();
        res.set('Content-Type', prometheus.register.contentType);
        res.end(metrics);
    } catch (error) {
        res.status(500).json({ error: 'Internal server error' });
    }
});

Health.check();
console.log('health check started');

// Start the express app
app.listen(3001, () => {
    console.log('Server listening on port 3001');
});
```

### Step 3: Health.js our custom health monitoring application
The `health.js` will contain a class that we create to perform custom health monitoring and expose custom prometheus metrcs.  This is where all the magic will happen.  

#### Import packages
You will need the following npm packages to be imported into `health.js` file.  

```javascript
const prometheus = require('prom-client');
const https = require('https');
const axios = require('axios').create({
    httpsAgent: new https.Agent({
        rejectUnauthorized: false
    })
})
```

1. `prom-client` allows us to create custom metrics. 
2. `https` lets us to make http call against tls enabled endpoints. 
3. `axios` is a common library used to make http calls.  There are many options in this space but axios is one of the most robust options. 

Notice I added a `httpsAgent` with a `rejectUnauthorized` statement. This is needed if you are calling https endpoints and don't care about tls cert errors. This is only needed if you are calling endpoints that are protected behind private certs. Public certificates should not be an issue.  

#### Setup Health class

I prefer to create javascript classes when writing nodejs. Ideally this would be written in TypeScript but for now we are keeping it simple.  

Let's start by create the Health class with a constructor.  The constructor will always return an instance of the Health class if one already exists. This ensures we have a single Health object instance (singleton pattern)

```javascript
class Health {
    constructor(){
        if(Health.instance) {
            return Health.instance;
        }
    }
}
```

#### Register custom metrics and create a Gauge
Need more info here
```javascript
constructor(){
        if(Health.instance) {
            return Health.instance;
        }

        prometheus.collectDefaultMetrics({ register: new prometheus.Registry(), timeout: 5000 });
        prometheus.register.setDefaultLabels({app: 'synthetic-monitor'});

        this.services = require('../resources/endpoints.js');

        this.EndPointGauge = new prometheus.Gauge({
            name: 'up_down',
            help: 'Shows if an endpoint is up or down',
            labelNames: ['name', 'type', 'environment', 'region']
        })
    }
```