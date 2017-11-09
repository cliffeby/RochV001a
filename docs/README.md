# Confessions of a Cut and Paste Developer - Express API Testing
### _About Me_
I am a retired civil engineer studying the MEAN stack as an avocation.  I've found the software development "community" unparallelled in support for all levels of users. When you hit a roadblock, video tutorials, blogs, Stack Overflow, etc. provide a wealth of content.  For me, a quick search, a crtl-c/crlt-v and my problem is solved.  No other profession offers more to it colleagues.  However getting past the HW or sample app has often been a struggle for me.  As I navigate that world with my Roch app, I will document some of the "not so sample" issues and questions that arise.   

This GitHub page is the first of several planned blogs on my efforts to get beyond HW.  I expect that the document will be updated as I learn more about the MEAN Stack.
 
 Cliff Eby - 2017
 

### _Introduction_

In an attempt to learn a little about testing, I tried to write some Mocha tests for my server API in a MEAN stack project.  Once I included JWT authorization to the REST routes, I really struggled to get the test framework working.  After I abandoned the traditional &quot;ng test&quot; approach (more on that later), I started to ask questions &quot;What should I test?&quot;  &quot;Since I control the entire stack, can I adequately test my API on the client side?&quot;  &quot;What level of error reporting should my API generate?&quot;  and most importantly, &quot;Will my API tests ease front end development – What tradeoffs will I encounter?&quot;

A search for &quot;best practices&quot; for API and/or REST testing produced very little.  Articles suggested consistency, comprehensiveness, middleware and documentation, but none addressed it in the context of an authenticated/authorized self-owned MEAN stack.

The following is a guide for the above context.  It contains principles and a framework along with some &quot;How to&quot; suggestions.

### _Context, Caveats and Tools_

#### Context and caveats: #

1. Authenticated/authorized self-owned MEAN stack
2. REST API for CRUD operations
3. I have not yet deployed this server to production.  I suspect that opinions and tests may change once deployed

#### Tools

1. Mongo/Mongoose, Express, Angular, Node/nodemon
2. Angular CLI
3. Node - express sever uses JavaScript – still looking for a Typescript node server implementation that I can understand.
4. Auth0 – for authorization and JWT generation
5. Three JWT libraries - express-jwt, express-jwt-authz, and jwks-rsa
6. POSTMAN for test development and a test runner
7. Newman for command line testing

### _Principles for endpoint design_

#### Design - General and Error reporting

1. Use plural nouns for endpoint.  <span style="background-color:yellow">_WHY: Convention – most developers expect plural endpoints and singular POST, PUT, PATCH, and GET requests are clear with_ _a  /ID appended.  _</span>
2. API responses should use a limited set of http &quot;status-code&quot; responses.  They are:
  - 200 – OK
  - 401 – Unauthorized
  - 404 – Not Found
  - 500 – Server Error
    - Localhost refused to connect - ERR\_CONNECTION\_REFUSED

    <span style="background-color:yellow">_WHY:There are over 70 HTTP status codes. If you use status codes that are not common, you will force developers to search for your intent._</span>

3. API responses should contain specific messages.  <span style="background-color:yellow">_WHY: It&#39;s helpful to know the requested endpoint and result.  There are three ways to send this data:_</span>
  - _Express middleware – e.g: &#39;app.use(function(req, res) {res.status(404).send({url: req.originalUrl + &#39; not found&#39;})});_
  - _Express endpoint:  A res property can return any desired server data - e.g. res.send(500, {status:500, message: &#39;internal error&#39;, type:&#39;internal&#39;});_
  - _Console.log:  Log your meassage to a server terminal window.  **I used this approach so I can easily remove logging from my production server.  Unlike a public API, there is no need for front end users to get these server messages.**_
4. Modify the 200 – OK response from an endpoint or endpoint/ID not found.  <span style="background-color:yellow">_WHY: Misspelled endpoints and wrong IDs are common errors during development.  Make sure they are caught early.  See middleware below._</span>
5. Do not &quot;document&quot; expected error codes and messages in a separate document – e.g. Postman, Swagger,…  <span style="background-color:yellow">_WHY: While often a simple click to create, that document is just another maintenance requirement and is never at your fingertips.  Instead, make sure your server response and testing tool provide the needed info._</span>
6. Unit test for known responses. <span style="background-color:yellow"> _WHY: Make sure that your server response catches all conditions._</span>

#### Design – Middleware #

There are many articles and opinions on the proper use of express-server middleware for catching and reporting errors.  With two exceptions, I struggle to see the benefit of detailed status codes and unique messages for my context.  The exceptions are:

1. a &quot;404 – Not Found&quot;  app.get(&#39;\*&#39;, func…) fall through is provided by the express-generator for unspecified endpoints and paths.  However, when the path is valid and the id for a GET, PUT, or DELETE is not found, the default express server will return a null body and a 200 status code.  I use the code below to report a 404.

2. Similarly, an invalid id (Mongoose defined as not equal to a 16-character hex string) will crash the node server.  Error checking could be on the server or client, but since the id string is not a user input, I chose not to check for a valid hex value.


`Score.findById(req.params.id)`

  `.exec(function(err, score){`
  
  `  if (err) {
      console.log("Error retrieving score";);`
    
    }
    else {
      if ( !score) {
        res.statusCode = 404;
        console.log("Not FOUND - get Score");
      }
      res.json(score);
    }});`


### _Principles for endpoint testing_

#### Testing - Authorization #
- Test for each JWT scope condition. e.g. does absence of read:score produce &quot;Insufficient Scope&quot;
- Confirm messages JWT Malformed, No Authorization Token, and Insufficient Scope are produced

#### Testing – Mock server # 
Comment: None was used for early development.  A Test and Dev server were configured.  Make sure that your tests (both during test script creation and once complete) do not pollute your server data.

#### Testing - ALL POST, PUT, GET, and DELETE actions #
- Is request authorized i.e. Does the JWT contain the appropriate scope
- Is the response time within acceptable limits
- Is the status code correct- 200, 400,…
- Is the response message correct

#### Testing actions #
**POST**
- Does the response body contain all of the requested and expected properties
- Does the response body schema match the expected schema
- Does a selected property response equal the request

**PUT**
- Is the object ID found
- Does the selected property request get updated

**GET - all**
- Is the response body an array

**GET - one**
- Is the object ID found

**DELETE**
- Is the object ID found

### Asssuring Test Coverage #

- Create a compliance grid for your expected workflow and each endpoint.  Use it to make sure that your test cover all of these conditions.

#### My Compliance matrix #
<table >
    <tbody>
        <thead style= "font-size:18px background:black color:white">
        <tr>
            <td width="192" valign="top">
                <p><strong>Behavior</strong></p>
            </td>
            <td width="198" valign="top">
                <p>
                    <strong>Tool/Responsibility</strong>
                </p>
            </td>
            <td width="258" valign="top">
                <p>
                    <strong>Technique</strong>
                </p>
            </td>
        </tr>
        </thead>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>Login</strong>
                </p>
            </td>
            <td width="198" valign="top">
                <p>
                    Auth0
                </p>
            </td>
            <td width="258" valign="top">
                <p>
                    Client-side login uses Auth0 for authentication
                </p>
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>Roles</strong>
                </p>
            </td>
            <td width="198" valign="top">
                <p>
                    Auth0
                </p>
            </td>
            <td width="258" valign="top">
                <p>
Using <u>rules</u> in Auth0, users are granted                    <u>scopes</u> for each endpoint and CRUD action. A role
                    based JWT is issued by Auth0
                </p>
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>JWT security</strong>
                </p>
            </td>
            <td width="198" valign="top">
                <p>
                    Auth0
                </p>
                <p>
                    Server developer
                </p>
            </td>
            <td width="258" valign="top">
                <p>
                    Client secret is needed to modify JWT. Must not be made
                    public
                </p>
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>Authorize Routes</strong>
                </p>
            </td>
            <td width="198" valign="top">
                <p>
                    Developer
                </p>
                <p>
                    Uses
                </p>
                <p>
                    express-jwt, express-jwt-authz, and jwks-rsa
                </p>
            </td>
            <td width="258" valign="top">
                <p>
                    router.route('/scores')
                    <br/>
                    .get(jwtCheck, jwtAuthz(['read:scores']),
                    scoreController.getScores);
                </p>
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>Test user authorization</strong>
                </p>
            </td>
            <td width="198" valign="top">
                <p>
                    POSTMAN and Auth0 Developer to check UnauthorizedError
                    response - JWT Malformed and No Authorization Token
                </p>
            </td>
            <td width="258" valign="top">
                <p>
                    Login to Auth0 via Postman. See Auth0 API documentation for
                    required header and body. <strong>TEST </strong>
                    access_token for valid and invalid JWTs.
                </p>
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>User Identification</strong>
                </p>
            </td>
            <td width="198" valign="top">
                <p>
                    POSTMAN and Auth0
                </p>
                <p>
                    Developer to check UnauthorizedError response -Insufficient
                    Scope
                </p>
            </td>
            <td width="258" valign="top">
                <p>
                    <strong>TEST</strong>
                    id_token for expected user, role and scopes
                </p>
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>Endpoint unknown</strong>
                </p>
            </td>
            <td width="198" valign="top">
                <p>
                    POSTMAN
                </p>
                <p>
                    Developer
                </p>
            </td>
            <td width="258" valign="top">
                <p>
                    <strong>TEST</strong>
                    that unrecognized endpoint is handled
                </p>
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong> </strong>
                </p>
            </td>
            <td width="198" valign="top">
            </td>
            <td width="258" valign="top">
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>Known endpoint - POST</strong>
                </p>
            </td>
            <td width="198" valign="top">
            </td>
            <td width="258" valign="top">
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>a. </strong>
                    <strong>Is endpoint accessible</strong>
                </p>
                <p>
                    <strong> </strong>
                </p>
                <p><br>
                    <strong>b. </strong>
                    <strong>Is response timely</strong>
                </p>
                <p>
                    <strong> </strong>
                </p>
                <p>
                    <strong>c. </strong>
                    <strong>Requested and expected properties</strong>
                </p>
                <p>
                    <strong> </strong>
                </p>
                <p>
                    <strong>d. </strong>
                    <strong>Schema</strong>
                </p>
                <p>
                    <strong> </strong>
                </p>
                <p>
                    <strong>e. </strong>
                    <strong>Data</strong>
                </p>
            </td>
            <td width="198" valign="top">
                <p>
                    POSTMAN
                </p>
                <p>
                    Developer
                </p>
            </td>
            <td width="258" valign="top">
                <p>
                    <strong>TEST</strong>
                    unauthorized, authorized, and role/scope status codes and
                    messages
                </p>
                <p>
                    <strong> </strong>
                </p>
                <p>
                    <strong>TEST</strong>
                    response time
                </p>
                <p>
                    <strong>TEST</strong>
                    response body for properties: required, requested and
                    expected
                </p>
                <p>
                    <strong>TEST</strong>
                    schema for correct types
                </p>
                <p>
                    <strong>TEST</strong>
                    one property for specific data
                </p>
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>Known endpoint - PUT </strong>
                </p>
            </td>
            <td width="198" valign="top">
            </td>
            <td width="258" valign="top">
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>a. </strong>
                    <strong>Is endpoint accessible</strong>
                </p>
                <p>
                    <strong> </strong>
                </p>
                <p><br>
                    <strong>b. </strong>
                    <strong>Is response timely</strong>
                </p>
                <p>
                    <strong> </strong>
                </p>
                <p>
                    <strong>c. </strong>
                    <strong>Data</strong>
                </p>
            </td>
            <td width="198" valign="top">
                <p>
                    POSTMAN
                </p>
                <p>
                    Developer
                </p>
            </td>
            <td width="258" valign="top">
                <p>
                    <strong>TEST</strong>
                    unauthorized, authorized, and role/scope status codes and
                    messages. <strong></strong>
                </p>
                <p>
                    <strong>TEST</strong>
                    response time
                </p>
                <p>
                    <strong>TEST</strong>
                    not null and one property for specific data update
                </p>
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>Known endpoint - GET all</strong>
                </p>
            </td>
            <td width="198" valign="top">
            </td>
            <td width="258" valign="top">
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>a. </strong>
                    <strong>Is endpoint accessible</strong>
                </p>
                <p>
                    <strong> </strong>
                </p>
                <p><br>
                    <strong>b. </strong>
                    <strong>Is response timely</strong>
                </p>
                <p>
                    <strong> </strong>
                </p>
                <p>
                    <strong>c. </strong>
                    <strong>Data</strong>
                </p>
            </td>
            <td width="198" valign="top">
                <p>
                    POSTMAN
                </p>
                <p>
                    Developer
                </p>
            </td>
            <td width="258" valign="top">
                <p>
                    <strong>TEST</strong>
                    unauthorized, authorized, and role/scope status codes and
                    messages
                </p>
                <p>
                    <strong> </strong>
                </p>
                <p>
                    <strong>TEST</strong>
                    response time
                </p>
                <p>
                    <strong>TEST</strong>
                    for array of objects
                </p>
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>Known endpoint - GET one</strong>
                </p>
            </td>
            <td width="198" valign="top">
            </td>
            <td width="258" valign="top">
                <p>
                    <strong> </strong>
                </p>
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>a. </strong>
                    <strong>Is endpoint accessible</strong>
                </p>
                <p>
                    <strong> </strong>
                </p>
                <p>
                    <strong>b. </strong>
                    <strong>Is response timely</strong>
                </p>
                <p>
                    <strong> </strong>
                </p>
                <p>
                    <strong>c. </strong>
                    <strong>Data</strong>
                </p>
            </td>
            <td width="198" valign="top">
                <p>
                    POSTMAN
                </p>
                <p>
                    Developer
                </p>
            </td>
            <td width="258" valign="top">
                <p>
                    <strong>TEST</strong>
                    unauthorized, authorized, and role/scope status codes and
                    messages. <strong></strong>
                </p>
                <p>
                    <strong>TEST</strong>
                    response time
                </p>
                <p>
                    <strong>TEST</strong>
                    body is not null/ID not found
                </p>
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>Known endpoint - DELETE</strong>
                </p>
            </td>
            <td width="198" valign="top">
            </td>
            <td width="258" valign="top">
                <p>
                    <strong> </strong>
                </p>
            </td>
        </tr>
        <tr>
            <td width="192" valign="top">
                <p>
                    <strong>a. </strong>
                    <strong>Is endpoint accessible</strong>
                </p>
                <p>
                    <strong> </strong>
                </p>
                <p><br>
                    <strong>b. </strong>
                    <strong>Is response timely</strong>
                </p>
                <p>
                    <strong> </strong>
                </p>
                <p>
                    <strong>c. </strong>
                    <strong>Data</strong>
                </p>
            </td>
            <td width="198" valign="top">
                <p>
                    POSTMAN
                </p>
                <p>
                    Developer
                </p>
            </td>
            <td width="258" valign="top">
                <p>
                    <strong>TEST</strong>
                    unauthorized, authorized, and role/scope status codes and
                    messages. <strong></strong>
                </p>
                <p>
                    <strong>TEST</strong>
                    response time
                </p>
                <p>
                    <strong>TEST</strong>
                    body is not null/ID not found
                </p>
            </td>
        </tr>
    </tbody>
</table>


### POSTMAN #

I previously dabbled in POSTMAN, but rarely got beyond GET and DELETE.  In trying debug my basic Mocha/JWT tests, I turned to POSTMAN to see header request and response data.  The Auth0 Authentication API and Management API documentation and Lars Bilde&#39;s video [https://www.youtube.com/watch?v=DVMCq8v5b7I](https://www.youtube.com/watch?v=DVMCq8v5b7I) gave me the basics.  Along the way, I tried some POSTMAN tests and then the test runner.  Once I found that you can export the tests to a Newman command line test runner, I abandoned Mocha and dove in.

In general, POSTMAN is giving me all I need.  I like that it&#39;s easy to break out a single test from a collection and run it.  Having the http data response (html or json) readily viewable and a console log made writing the test script easy.  My only frustration was remembering to frequently save and backup my work.  For me, it seemed easy to overwrite my work.

The following describes my testing approach using POSTMAN and Newman:

My test scripts make use of _postman variables_ (desinated by double braces) to allow for test runner iteration by role.  These data are stored in POSTMAN&#39;s Environments (roch1) that are easily manipulated.

To get JWT access\_ and id\_ tokens, Auth0 requires the Headers Keys/Values shown below with Content-Type keys set to _application/x-www-form-urlencoded_.

![loginbody](https://user-images.githubusercontent.com/1431998/32575246-79a6f4c8-c4a1-11e7-8ab3-687e2fd7e815.png)

Prior to the POST, many environmental variables are set and are used by subsequent tests in the collection.

![loginpretest](https://user-images.githubusercontent.com/1431998/32575239-792c0f1a-c4a1-11e7-9b49-c7e0c6c5ed0c.jpg)

Since this is a POST to Auth0, the only test is to assure that it is successful.  Then, JWT access\_ and id\_ tokens are stored in the postman.setGlobalVariable collection.  I chose global variable because these tokens are unreadable and long. It keeps the Environment section readable.

![logintest](https://user-images.githubusercontent.com/1431998/32575240-793af0fc-c4a1-11e7-9bea-97d50456b9cc.jpg)

POSTMAN now has a JWT access\_token that allows API access.  Setting Authorization in the Header to **Bearer** followed by the _access token_, then setting the audience to a _unique string_ defined in Auth0 and changing the Content-type to _application/json_, is all that is needed.  Not shown is that this access\_token includes a scope of create:score that matches the server&#39;s express router requirement.

`router.route("/scores")`     
`.post(jwtCheck, jwtAuthz(["create:score"]), scoreController.postScore)`

![postheader](https://user-images.githubusercontent.com/1431998/32575243-796de098-c4a1-11e7-9153-d204c2f2f72c.jpg)

Next, the TEST scripts are created.  They are basic JavaScript and I use if/else to test for role conditions.  Like any testing library, the syntax takes some time to master, but I like that it is concise and all in one location.

![posttests](https://user-images.githubusercontent.com/1431998/32575245-79988bd6-c4a1-11e7-9773-30c67d699fb2.jpg)

The entire Roch collection test can run in POSTMAN or in Newman.  Using a file named &quot;data&quot; (json or csv), {{variables}} are defined for each iteration.  For example, I change the username and password for each iteration with a pdata.json file:

`[{
  "username": "admin@roch.com",
  "password": "Password"
},`            
 `{
  "username": "rplayer@roch.com",
  "password": "Password"
},`            
 `{
  "username": "rmember@roch.com",
  "password": "Password"
}]`

Exporting all or part of the collection creates a json file that can be used as a Newman input to run at the command line.

C:\Users\cliff\WebstormProjects\RochV001\ngApp&gt;newman run server/tests/roch1.postman\_collection.json -d server/tests/pdata.json  .  Alternatively, you can run the collection&#39;s iterations in the POSTMAN runner.

![newman](https://user-images.githubusercontent.com/1431998/32575242-79531222-c4a1-11e7-801a-7eeee752e555.jpg)

![postrunner](https://user-images.githubusercontent.com/1431998/32575244-798b1f78-c4a1-11e7-9803-a47671d167f1.jpg)

Sources
The exported JSON to use with Newman which can be imported back to POSTMAN is in this repo at imports.json. https://github.com/cliffeby/RochV001a/import.json

[https://derickbailey.com/2014/09/06/proper-error-handling-in-expressjs-route-handlers/](https://derickbailey.com/2014/09/06/proper-error-handling-in-expressjs-route-handlers/)

[https://www.joyent.com/node-js/production/design/errors](https://www.joyent.com/node-js/production/design/errors)

[http://blog.restcase.com/rest-api-error-codes-101/](http://blog.restcase.com/rest-api-error-codes-101/)

[https://kostasbariotis.com/rest-api-error-handling-with-express-js/](https://kostasbariotis.com/rest-api-error-handling-with-express-js/)

[https://webapplog.com/intro-to-express-js-parameters-error-handling-and-other-middleware/](https://webapplog.com/intro-to-express-js-parameters-error-handling-and-other-middleware/)

[http://goldbergyoni.com/checklist-best-practices-of-node-js-error-handling/](http://goldbergyoni.com/checklist-best-practices-of-node-js-error-handling/)

[https://www.owasp.org/index.php/REST\_Security\_Cheat\_Sheet](https://www.owasp.org/index.php/REST_Security_Cheat_Sheet)

[https://blog.mwaysolutions.com/2014/06/05/10-best-practices-for-better-restful-api/](https://blog.mwaysolutions.com/2014/06/05/10-best-practices-for-better-restful-api/)

[https://www.youtube.com/watch?v=DVMCq8v5b7I](https://www.youtube.com/watch?v=DVMCq8v5b7I)

[https://www.youtube.com/watch?v=gqcBptGjHTo&amp](https://www.youtube.com/watch?v=gqcBptGjHTo&amp;t=208s)

