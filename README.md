<img src="https://www.jeremydaly.com/wp-content/uploads/2018/03/lambda-api-logo.svg" />


[![Build Status](https://travis-ci.org/jeremydaly/lambda-api.svg?branch=master)](https://travis-ci.org/jeremydaly/lambda-api)
[![npm](https://img.shields.io/npm/v/lambda-api.svg)](https://www.npmjs.com/package/lambda-api)
[![npm](https://img.shields.io/npm/l/lambda-api.svg)](https://www.npmjs.com/package/lambda-api)

### Lightweight web framework for your serverless applications

Lambda API is a lightweight web framework for use with AWS API Gateway and AWS Lambda using Lambda Proxy integration. This closely mirrors (and is based on) other routers like Express.js, but is significantly stripped down to maximize performance with Lambda's stateless, single run executions.

## Simple Example

```javascript
const API = require('lambda-api') // API library

// Init API instance
const api = new API({ version: 'v1.0', base: 'v1' });

api.get('/test', function(req,res) {
  res.status(200).json({ status: 'ok' })
})

module.exports.handler = (event, context, callback) => {

  // Run the request
  api.run(event,context,callback);

} // end handler
```

## Why Another Web Framework?
Express.js, Fastify, Koa, Restify, and Hapi are just a few of the many amazing web frameworks out there for Node.js. So why build yet another one when there are so many great options already? One word: **DEPENDENCIES**.

These other frameworks are extremely powerful, but that benefit comes with the steep price of requiring several additional Node.js modules. Not only is this a bit of a security issue (see Beware of Third-Party Packages in [Securing Serverless](https://www.jeremydaly.com/securing-serverless-a-newbies-guide/)), but it also adds bloat to your codebase, filling your `node_modules` directory with a ton of extra files. For serverless applications that need to load quickly, all of these extra dependencies slow down execution and use more memory than necessary. Express.js has **30 dependencies**, Fastify has **12**, and Hapi has **17**! These numbers don't even include their dependencies' dependencies.

Lambda API has **ONE** dependency. We use [Bluebird](http://bluebirdjs.com/docs/getting-started.html) promises to serialize asynchronous execution. We use promises because AWS Lambda currently only supports Node v6.10, which doesn't support `async / await`. Bluebird is faster than native promise and it has **no dependencies** either, making it the perfect choice for Lambda API.

Lambda API was written to be extremely lightweight and built specifically for serverless applications using AWS Lambda. It provides support for API routing, serving up HTML pages, issuing redirects, and much more. It has a powerful middleware and error handling system, allowing you to implement everything from custom authentication to complex logging systems. Best of all, it was designed to work with Lambda's Proxy Integration, automatically handling all the interaction with API Gateway for you. It parses **REQUESTS** and formats **RESPONSES** for you, allowing you to focus on your application's core functionality, instead of fiddling with inputs and outputs.

## Lambda Proxy integration
Lambda Proxy Integration is an option in API Gateway that allows the details of an API request to be passed as the `event` parameter of a Lambda function. A typical API Gateway request event with Lambda Proxy Integration enabled looks like this:

```javascript
{
  "resource": "/v1/posts",
  "path": "/v1/posts",
  "httpMethod": "GET",
  "headers": {
    "Authorization": "Bearer ...",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
    "Accept-Encoding": "gzip, deflate",
    "Accept-Language": "en-us",
    "cache-control": "max-age=0",
    "CloudFront-Forwarded-Proto": "https",
    "CloudFront-Is-Desktop-Viewer": "true",
    "CloudFront-Is-Mobile-Viewer": "false",
    "CloudFront-Is-SmartTV-Viewer": "false",
    "CloudFront-Is-Tablet-Viewer": "false",
    "CloudFront-Viewer-Country": "US",
    "Cookie": "...",
    "Host": "...",
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) ...",
    "Via": "2.0 ... (CloudFront)",
    "X-Amz-Cf-Id": "...",
    "X-Amzn-Trace-Id": "...",
    "X-Forwarded-For": "xxx.xxx.xxx.xxx",
    "X-Forwarded-Port": "443",
    "X-Forwarded-Proto": "https"
  },
  "queryStringParameters": {
    "qs1": "q1"
  },
  "stageVariables": null,
  "requestContext": {
    "accountId": "...",
    "resourceId": "...",
    "stage": "prod",
    "requestId": "...",
    "identity": {
      "cognitoIdentityPoolId": null,
      "accountId": null,
      "cognitoIdentityId": null,
      "caller": null,
      "apiKey": null,
      "sourceIp": "xxx.xxx.xxx.xxx",
      "accessKey": null,
      "cognitoAuthenticationType": null,
      "cognitoAuthenticationProvider": null,
      "userArn": null,
      "userAgent": "...",
      "user": null
    },
    "resourcePath": "/v1/posts",
    "httpMethod": "GET",
    "apiId": "..."
  },
  "body": null,
  "isBase64Encoded": false
}
```

The API automatically parses this information to create a normalized `REQUEST` object. The request can then be routed using the APIs methods.

## Configuration

Include the `lambda-api` module into your Lambda handler script and initialize an instance. You can initialize the API with an optional `version` which can be accessed via the `REQUEST` object and a `base` path. The base path can be used to route multiple versions to different instances.

```javascript
const API = require('lambda-api') // API library

// Init API instance with optional version and base path
const api = new API({ version: 'v1.0', base: 'v1' });
```

## Routes and HTTP Methods

Routes are defined by using convenience methods or the `METHOD` method. There are currently five convenience route methods: `get()`, `post()`, `put()`, `delete()` and `options()`. Convenience route methods require two parameters, a *route* and a function that accepts two arguments. A *route* is simply a path such as `/users`. The second parameter must be a function that accepts a `REQUEST` and a `RESPONSE` argument. These arguments can be named whatever you like, but convention dictates `req` and `res`. Examples using convenience route methods:

```javascript
api.get('/users', function(req,res) {
  // do something
})

api.post('/users', function(req,res) {
  // do something
})

api.delete('/users', function(req,res) {
  // do something
})
```
Additional methods are support by calling the `METHOD` method with three arguments. The first argument is the HTTP method, a *route*, and a function that accepts a `REQUEST` and a `RESPONSE` argument.

```javascript
api.METHOD('patch','/users', function(req,res) {
  // do something
})
```

## REQUEST

The `REQUEST` object contains a parsed and normalized request from API Gateway. It contains the following values by default:

- `app`: A reference to an instance of the app
- `version`: The version set at initialization
- `params`: Dynamic path parameters parsed from the path (see [path parameters](#path-parameters))
- `method`: The HTTP method of the request
- `path`: The path passed in by the request
- `query`: Querystring parameters parsed into an object
- `headers`: An object containing the request headers
- `body`: The body of the request.
 - If the `Content-Type` header is `application/json`, it will attempt to parse the request using `JSON.parse()`
 - If the `Content-Type` header is `application/x-www-form-urlencoded`, it will attempt to parse a URL encoded string using `querystring`
 - Otherwise it will be plain text.
- `route`: The matched route of the request
- `requestContext`: The `requestContext` passed from the API Gateway
- `namespace` or `ns`: A reference to modules added to the app's namespace (see [namespaces](#namespaces))

The request object can be used to pass additional information through the processing chain. For example, if you are using a piece of authentication middleware, you can add additional keys to the `REQUEST` object with information about the user. See [middleware](#middleware) for more information.

## RESPONSE

The `RESPONSE` object is used to send a response back to the API Gateway. The `RESPONSE` object contains several methods to manipulate responses. All methods are chainable unless they trigger a response.

### status
The `status` method allows you to set the status code that is returned to API Gateway. By default this will be set to `200` for normal requests or `500` on a thrown error. Additional built-in errors such as `404 Not Found` and `405 Method Not Allowed` may also be returned. The `status()` method accepts a single integer argument.

```javascript
api.get('/users', function(req,res) {
  res.status(401).error('Not Authorized')
})
```

### header
The `header` method allows for you to set additional headers to return to the client. By default, just the `Content-Type` header is sent with `application/json` as the value. Headers can be added or overwritten by calling the `header()` method with two string arguments. The first is the name of the header and then second is the value.

```javascript
api.get('/users', function(req,res) {
  res.header('Content-Type','text/html').send('<div>This is HTML</div>')
})
```

### send
The `send` methods triggers the API to return data to the API Gateway. The `send` method accepts one parameter and sends the contents through as is, e.g. as an object, string, integer, etc. AWS Gateway expects a string, so the data should be converted accordingly.

### json
There is a `json` convenience method for the `send` method that will set the headers to `application\json` as well as perform `JSON.stringify()` on the contents passed to it.

```javascript
api.get('/users', function(req,res) {
  res.json({ message: 'This will be converted automatically' })
})
```

### html
There is also an `html` convenience method for the `send` method that will set the headers to `text/html` and pass through the contents.

```javascript
api.get('/users', function(req,res) {
  res.html('<div>This is HTML</div>')
})
```

### location
The `location` convenience method sets the `Location:` header with the value of a single string argument. The value passed in is not validated but will be encoded before being added to the header. Values that are already encoded can be safely passed in. Note that a valid `3xx` status code must be set to trigger browser redirection. The value can be a relative/absolute path OR a FQDN.

```javascript
api.get('/redirectToHome', function(req,res) {
  res.location('/home').status(302).html('<div>Redirect to Home</div>')
})

api.get('/redirectToGithub', function(req,res) {
  res.location('https://github.com').status(302).html('<div>Redirect to GitHub</div>')
})
```

### redirect
The `redirect` convenience method triggers a redirection and ends the current API execution. This method is similar to the `location()` method, but it automatically sets the status code and calls `send()`. The redirection URL (relative/absolute path OR a FQDN) can be specified as the only parameter or as a second parameter when a valid `3xx` status code is supplied as the first parameter. The status code is set to `302` by default, but can be changed to `300`, `301`, `302`, `303`, `307`, or `308` by adding it as the first parameter.

```javascript
api.get('/redirectToHome', function(req,res) {
  res.redirect('/home')
})

api.get('/redirectToGithub', function(req,res) {
  res.redirect(301,'https://github.com')
})
```

### error
An error can be triggered by calling the `error` method. This will cause the API to stop execution and return the message to the client. Custom error handling can be accomplished using the [Error Handling](#error-handling) feature.

```javascript
api.get('/users', function(req,res) {
  res.error('This is an error')
})
```

## Path Parameters
Path parameters are extracted from the path sent in by API Gateway. Although API Gateway supports path parameters, the API doesn't use these values but insteads extracts them from the actual path. This gives you more flexibility with the API Gateway configuration. Path parameters are defined in routes using a colon `:` as a prefix.

```javascript
api.get('/users/:userId', function(req,res) {
  res.send('User ID: ' + req.params.userId)
})
```

Path parameters act as wildcards that capture the value into the `params` object. The example above would match `/users/123` and `/users/test`. The system always looks for static paths first, so if you defined paths for `/users/test` and `/users/:userId`, exact path matches would take precedence. Path parameters only match the part of the path they are defined on. E.g. `/users/456/test` would not match `/users/:userId`. You would either need to define `/users/:userId/test` as its own path, or create another path with an additional path parameter, e.g. `/users/:userId/:anotherParam`.

A path can contain as many parameters as you want. E.g. `/users/:param1/:param2/:param3`.

## Wildcard Routes
Wildcard routes are supported for methods that match an existing route. E.g. `options` on an existing `get` route. As of now, the best use case is for the OPTIONS method to provide CORS headers. Wildcards only work in the base path. `/users/*`, for example, is not supported. For additional wildcard support, use [Path Parameters](#path-parameters) instead.

```javascript
api.options('/*', function(req,res) {
  // Do something
  res.status(200).send({});
})
```

## Middleware
The API supports middleware to preprocess requests before they execute their matching routes. Middleware is defined using the `use` method and require a function with three parameters for the `REQUEST`, `RESPONSE`, and `next` callback. For example:

```javascript
api.use(function(req,res,next) {
  // do something
  next()
})
```

Middleware can be used to authenticate requests, log API calls, etc. The `REQUEST` and `RESPONSE` objects behave as they do within routes, allowing you to manipulate either object. In the case of authentication, for example, you could verify a request and update the `REQUEST` with an `authorized` flag and continue execution. Or if the request couldn't be authorized, you could respond with an error directly from the middleware. For example:

```javascript
// Auth User
api.use(function(req,res,next) {
  if (req.headers.Authorization === 'some value') {
    req.authorized = true
    next() // continue execution
  } else {
    res.status(401).error('Not Authorized')
  }
})
```

The `next()` callback tells the system to continue executing. If this is not called then the system will hang and eventually timeout unless another request ending call such as `error` is called. You can define as many middleware functions as you want. They will execute serially and synchronously in the order in which they are defined.

## Clean Up
The API has a built-in clean up method called 'finally()' that will execute after all middleware and routes have been completed, but before execution is complete. This can be used to close database connections or to perform other clean up functions. A clean up function can be defined using the `finally` method and requires a function with two parameters for the REQUEST and the RESPONSE as its only argument. For example:

```javascript
api.finally(function(req,res) {
  // close unneeded database connections and perform clean up
})
```

The `RESPONSE` **CANNOT** be manipulated since it has already been generated. Only one `finally()` method can be defined. This uses the Bluebird `finally()` method internally and will execute after properly handled errors as well.

## Error Handling
The API has simple built-in error handling that will log the error using `console.log`. These will be available via CloudWatch Logs. By default, errors will trigger a JSON response with the error message. If you would like to define additional error handling, you can define them using the `use` method similar to middleware. Error handling middleware must be defined as a function with **four** arguments instead of three like normal middleware. An additional `error` parameter must be added as the first parameter. This will contain the error object generated.

```javascript
api.use(function(err,req,res,next) {
  // do something with the error
  next()
})
```

The `next()` callback will cause the script to continue executing and eventually call the standard error handling function. You can short-circuit the default handler by calling a request ending method such as `send`, `html`, or `json`.

## Namespaces
Lambda API allows you to map specific modules to namespaces that can be accessed from the `REQUEST` object. This is helpful when using the pattern in which you create a module that exports middleware, error, or route functions. In the example below, the `data` namespace is added to the API and then accessed by reference within an included module.

The main handler file might look like this:

```javascript
// Use app() function to add 'data' namespace
api.app('data',require('./lib/data.js'))

// Create a get route to load user details
api.get('/users/:userId', require('./lib/users.js'))
```

The users.js module might look like this:

```javascript
module.exports = function(req, res) {
  let userInfo = req.namespace.data.getUser(req.params.userId)
  res.json({ 'userInfo': userInfo })
});
```

By saving references in namespaces, you can access them without needing to require them in every module. Namespaces can be added using the `app()` method of the API. `app()` accepts either two parameters: a string representing the name of the namespace and a function reference *OR* an object with string names as keys and function references as the values. For example:

```javascript
api.app('namespace',require('./lib/ns-functions.js'))

// OR

api.app({
  'namespace1': require('./lib/ns1-functions.js'),
  'namespace2': require('./lib/ns2-functions.js')
})
```

## Promises
The API uses Bluebird promises to manage asynchronous script execution. The API will wait for a request ending call before returning data back to the client. Middleware will wait for the `next()` callback before proceeding to the next step.

**NOTE:** AWS Lambda currently only supports Node v6.10, which doesn't support `async / await`. If you'd like to use `async / await`, you'll need to polyfill.

## CORS Support
CORS can be implemented using the [wildcard routes](#wildcard-routes) feature. A typical implementation would be as follows:

```javascript
api.options('/*', function(req,res) {
  // Add CORS headers
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Methods', 'GET, PUT, POST, DELETE, OPTIONS');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization, Content-Length, X-Requested-With');
  res.status(200).send({});
})
```

Conditional route support could be added via middleware or with conditional logic within the `OPTIONS` route.

## Configuring Routes in API Gateway
Routes must be configured in API Gateway in order to support routing to the Lambda function. The easiest way to support all of your routes without recreating them is to use [API Gateway's Proxy Integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-set-up-simple-proxy.html#api-gateway-proxy-resource?icmpid=docs_apigateway_console).

Simply create one `{proxy+}` route that uses the `ANY` method and all requests will be routed to your Lambda function and processed by the `lambda-api` module.

## Contributions
Contributions, ideas and bug reports are welcome and greatly appreciated. Please add  [issues](https://github.com/jeremydaly/lambda-api/issues) for suggestions and bugs reports.
