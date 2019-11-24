* Boxpik: Docker/Chai tests
  - CodeMentor example:
https://www.codementor.io/jeanpauldelimat/api-end-to-end-testing-with-docker-10ng9l43bc?utm_content=posts&utm_source=sendgrid&utm_medium=email&utm_term=post-10ng9l43bc&utm_campaign=newsletter20191113
    - Has NodeJS/PostgreSQL REST API back-end
      <= We'll ignore this, and use Boxpik instead...
    - Creates Docker image for Unit test
    - Creates another Docker image for the REST API
    - ALSO: Dockerize, Redis swarm, mocking
    <= NOTE: apparently "Dockerizes" the whole shebang.
             We don't *WANT* to Dockerize anytning but the tests.
             Q: Do we even *NEED* Docker for that?  Can't we just run the Mocha/Chai tests *anywhere* (with NPM/NodeJS)?

  - Docker tutorials:
    - Dockerize: Dockerizing an application is the process of converting an application to run within a Docker container.
    - https://docs.docker.com/get-started/
      <= Installs "Docker Desktop", which in turn installs Kubernetes (windows/OSX only)

  - Mocha/Chai tutorials:
https://raygun.com/blog/mocha-vs-jasmine-chai-sinon-cucumber/
https://www.codementor.io/codementorteam/javascript-testing-framework-comparison-jasmine-vs-mocha-8s9k27za3
https://scotch.io/tutorials/test-a-node-restful-api-with-mocha-and-chai
https://www.sitepoint.com/unit-test-javascript-mocha-chai/
https://scotch.io/tutorials/how-to-test-nodejs-apps-using-mocha-chai-and-sinonjs

===================================================================================================

* Test a Node RESTful API with Mocha and Chai, Samuele Zaza
    https://scotch.io/tutorials/test-a-node-restful-api-with-mocha-and-chai
    https://github.com/samuxyz/bookstore

  - Table of Contents:
    -----------------
    1. Introduction
    2. Mocha: Testing Environment
    3. Chai: Assertion Library
    4. Project setup
    5. The server
    6. A Naive Test
    7. A Better Test
    8. Conclusion
    9. Bonus Mockgoose

   - Prerequisites:
     - Node.js: basic understanding of node.js.
     - Postman: for making fast HTTP requests to the API.
     - ES6: Familiarity with  ES6 syntax.
     - MongoDB installed and running

---------------------------------------------------------------------------------------------------
1. Introduction
   - REST Backend: "bookstore"
     - app\routes\book.js:
       let mongoose = require('mongoose');
       let Book = require('../models/book');
         getBooks()
         postBook()
         getBook()
         deleteBook()
         updateBook()

     - app\models\book.js:
       let mongoose = require('mongoose');
       let Schema = mongoose.Schema;
       let BookSchema = new Schema({
         title: { type: String, required: true },
         author: { type: String, required: true },
         year: { type: Number, required: true },
         pages: { type: Number, required: true, min: 1 },
         createdAt: { type: Date, default: Date.now }, 
         ...
    <= Mongoose: NodeJS library for MongoDB

---------------------------------------------------------------------------------------------------
2. Mocha: Testing Environment
   - Previous tutorials in series:
     - Building a RESTful API using Node and Express 4 (also MongoDB, Mongoose and Postman);
         https://scotch.io/tutorials/build-a-restful-api-using-node-and-express-4

     - Authenticate a Node ES6 API with JSON Web Tokens:
         https://scotch.io/tutorials/authenticate-a-node-es6-api-with-json-web-tokens

   - In this tutorial, we'll use Mocha and Chai to test CRUD operations against a "bookstore" (REST AP)

   - Mocha features:
     - simple async support, including promises.
     - async test timeout support.
     - before, after, before each, after each hooks (very useful to clean the environment where each test!).
     - use any assertion library you want, Chai in our tutorial.

---------------------------------------------------------------------------------------------------
3. Chai: Assertion Library
   - Chai shines on the freedom of choosing the interface we prefer: "should", "expect", "assert".
     <= they are *ALL* available.

   - Chai HTTP addon allows Chai library to easily use assertions on HTTP requests.
---------------------------------------------------------------------------------------------------
4. Project setup

   bookstore-master\
   |
   +---app\
   |   +---models\
   |   |   +-book.js
   |   \---routes\
   |       +-book.js
                      <= bookstore REST API
   +---config\
   |   +---default.json
   |   |---dev.json
   |   \---test.json  <= default.json, dev.json: ```javascript { "DBHost": "YOUR_DB_URI" } ```
                         test.json: ```javascript { "DBHost": "YOUR_TEST_DB_URI" } ```
   +---test\
   |   +---book.js
                      <= Our tests
   +---package.json
   \---server.js
       <= NPM scripts/dependencies, NodeJS "server" app

   - package.json:
     -------------
{
  "name": "bookstore",
  "version": "1.0.0",
  "description": "A bookstore API",
  "main": "server.js",
  "author": "Sam",
  "license": "ISC",
  "dependencies": {
    "body-parser": "^1.19.0",  // OLD: "^1.15.1",
    "config": "^3.2.4",        // OLD: "^1.20.1",
    "express": "^4.17.1",      // OLD: "^4.13.4",
    "mongoose": "5.7.12",      // OLD: "^4.4.15",
    "morgan": "^1.9.1"         // OLD: "^1.7.0"
  },
  "devDependencies": {
    "chai": "^4.2.0",          // OLD: "^3.5.0",
    "chai-http": "^4.3.0",     // OLD: "^2.0.1",
    "mocha": "^6.2.2"          // OLD: "^2.4.5"
  },
  "scripts": {
    "start": "SET NODE_ENV=dev && node server.js",
    "test": "mocha --timeout 10000"
  }
}
   <= REFERENCE: https://www.npmjs.com/search?q=mocha

  - cd bookstore-master
    npm --version: 6.11.3; npm update -g npm => 6.13.1
    node --version: v11.6.0: OK as-is
    ng --version: 7.2.1; npm install -g @angular/cli@latest => 8.3.19
    <= Update NPM, NodeJS (Angular not related to this tutorial...)
    npm update
    <= Install npm_modules listed in package.json

    code .
    <= Start VSCode
---------------------------------------------------------------------------------------------------
5. The server
   
  - config/{dev, default}.json:
      { "DBHost": "mongodb://localhost:27017/bookstore" }
    config/test.json:
      { "DBHost": "mongodb://localhost:27017/bookstore-test" }

    NOTES: 
    - Running Node on Linux (ubuntu18) - there's where MongoDB happens to be installed
    - To connect remotely to MongoDB from a Windows host, need to edit /etc/mongod.conf:
        sudo vi /etc/mongod.conf =>
# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0  <-- Change this to 0.0.0.0
...

   - Minimal "bookstore" app: 

    - package.json:
      "scripts": {
        "start": "export NODE_ENV=dev && node server.js",
      <= "npm start" invokes server.js

    - server.js:
      <= invokes functionality in app/models/book.js (model) and app/routes/book.js (REST endpoints)_

    - Both server.js and app/routes/book.js access MongoDB:
      - server.js
          let mongoose = require('mongoose');

          ...
          mongoose.connect(config.DBHost, options);
          let db = mongoose.connection;
          db.on('error', console.error.bind(console, 'connection error:'));
          <= We actually seem to get PAST this point...

    - mongo
      > db => test
      > show dbs =>
HelloMongoDB  0.000GB
admin         0.000GB
config        0.000GB
local         0.000GB
mydb          0.000GB
test          0.000GB
  <= No "bookstore" DBs...

---------------------------------------------------------------------------------------------------
6. A Naive Test

   - Postman (windows) > 
     - GET, http://ubuntu18:8081/book, [Send]
       HTTP Result: HTTP 200, HTTP return: application/json;charset= utf8: []
       <= No books present...

     - POST, http://ubuntu18:8081/book, body:
{
  "title": "Lord of the Rings",
  "author": "J.R.R. Tolkein",
  "year": 1954,
  "pages": 1170
}
      <= Result: HTTP 200, message 

    - POST, http://ubuntu18:8081/book, body:
      - REQUEST:
    {
        "title": "Dr. Sleep",
        "author": "Stephen King",
        "year": 2017,
        "pages": 300
    }
      - RESPONSE: HTTP 200, 
{
    "message": "Book successfully added!",
    "book": {
        "_id": "5dd88382b27316ce6be90227",
        "title": "Dr. Sleep",
        "author": "Stephen King",
        "year": 2017,
        "pages": 300,
        "createdAt": "2019-11-23T00:55:30.118Z"
    }
}
    - GET, http://ubuntu18:8081/book
      - RESPONSE: HTTP 200, 
[
    {
        "_id": "5dd880adb27316ce6be90226",
        "title": "Lord of the Rings",
        "author": "J.R.R. Tolkein",
        "year": 1954,
        "pages": 1170,
        "createdAt": "2019-11-23T00:43:25.512Z"
    }
    ...
]

    - mongo, show dbs
> show dbs
HelloMongoDB  0.000GB
admin         0.000GB
bookstore     0.000GB  <-- Bingo!
config        0.000GB
local         0.000GB
mydb          0.000GB
test          0.000GB

  Postman > PUT >  http://ubuntu18:8081/book/5dd880adb27316ce6be90226
      - REQUEST:
{
	"message": "Updating Title!",
    "book": {
        "_id": "5dd880adb27316ce6be90226",
        "title": "The Two Towers",
        "author": "J.R.R. Tolkein",
        "year": 1954,
        "pages": 1170
    }
}
      - RESPONSE: HTTP 200, 
{
    "message": "Book updated!",
    "book": {
        "_id": "5dd880adb27316ce6be90226",
        "title": "Lord of the Rings",
        "author": "J.R.R. Tolkein",
        "year": 1954,
        "pages": 1170,
        "createdAt": "2019-11-23T00:43:25.512Z"
    }
}
       <= Yay!  Everything "seems to work" (in Postman).

    - mongo, 
        show dbs
        <= See "bookstore"
        use bookstore
        show collections => books
         db.books.find() =>
{ "_id" : ObjectId("5dd880adb27316ce6be90226"), "title" : "Lord of the Rings", "author" : "J.R.R. Tolkein", "year" : 1954, "pages" : 1170, "createdAt" : ISODate("2019-11-23T00:43:25.512Z") }
{ "_id" : ObjectId("5dd88382b27316ce6be90227"), "title" : "Dr. Sleep", "author" : "Stephen King", "year" : 2017, "pages" : 300, "createdAt" : ISODate("2019-11-23T00:55:30.118Z") }
       <= We also see all expected records in Mongo!

     Q: Good enough to ship?
     A: NOOOOOOO!!!!
     NEXT: Enter *automated tests* (with Mocha and Chai)

---------------------------------------------------------------------------------------------------
7. A Better Test

  - test/book.js:
//During the test the env variable is set to test
process.env.NODE_ENV = 'test';
  <= package.json@start sets NODE_ENV to "dev" before launching server

    - Imports/dependencies:
let mongoose = require("mongoose");
let Book = require('../app/models/book');
let chai = require('chai');
let chaiHttp = require('chai-http');
let server = require('../server');
let should = chai.should();
  <= We can assert "should", "expect", and/or "assert".
     Let's use "should"...

    - The modules we're using ("server", etc) all did a "module.exports":
      - server.js:
module.exports = app; // for testing
      - app/models.book.js:
module.exports = mongoose.model('book', BookSchema);
      - app/routes/book.js:
module.exports = { getBooks, postBook, getBook, deleteBook, updateBook };

    - beforeEach: *DELETE* all existing books to initialize before each test
      <= Note: we're running the tests against a *SEPARATE* DB: "bookstore-test" (test), instead of "bookstore" (prod)

    - cd bookstore-master
      npm test=>
sh: 1: mocha: not found
npm ERR! Test failed.  See above for more details.
      <= Mocha, Chai were *NOT* auto-installed from package.json/npm update

    - npm install --save mocha
      <= mocha@6.2.2
      npm install --save chai
      <= chai@4.2.0
      npm install --save chai-http
      <= chai-http@4.3.0
      npm test =>
  1) Uncaught error outside test suite
  Books
    /GET book
(node:54190) DeprecationWarning: collection.remove is deprecated. Use deleteOne, deleteMany, or bulkWrite instead.
      ✓ it should GET all the books
    /POST book
      ✓ it should not POST a book without pages field (41ms)
      ✓ it should POST a book  (69ms)
    /GET/:id book
      ✓ it should GET a book by the given id
    /PUT/:id book
      ✓ it should UPDATE a book given the id
    /DELETE/:id book
      ✓ it should DELETE a book given the id
  6 passing (237ms)
  1 failing
  1) Uncaught error outside test suite:
     Uncaught Error: listen EADDRINUSE: address already in use :::8081
     <= Original "npm start" dev server still running
  <<  Re-ran tests: all six passed  >>

---------------------------------------------------------------------------------------------------
8. Conclusion

   - In this tutorial we faced the problem of testing our routes to provide our users a stable experience.

   - We went through all the steps of creating a RESTful API, doing a naive test with POSTMAN.

   - It is good habit to always spend some time making tests to assure a server as reliable as possible but unfortunately it is often underestimated.

---------------------------------------------------------------------------------------------------
9. Bonus Mockgoose
   - Instead of using two separate databases (bookstore and bookstore-test), we could have used MockGoose:
       https://github.com/mockgoose/mockgoose

===================================================================================================
* Other Mocha links:
  - https://samwize.com/2014/02/08/a-guide-to-mochas-describe-it-and-setup-hooks/
    - Minimal (Mocha-only) "Hello world":
      1. cd my-test-dir; npm install mocha
         <= NOTE: this will *FAIL* if "package.json" is present, but not "complete"..
      2. mkdir test; vi test/test.js:
var assert = require('assert');
describe('Array', () => {
  describe('#indexOf()', () => {
    it('should return -1 when the value is not present', () => {
      assert.equal([1, 2, 3].indexOf(4), -1);
    });
  });
});
         <= NOTE: Mocha will this will *FAIL* if "package.json" is present, but not "complete"..
      2. vi package.json:
"scripts": {
  "test": "mocha"
}
         <= NOTES:
            1. The npm target is "test"
            2. Target "test" will execute command "node mocha" 
            3. Mocha will run any/all *.js files it finds under "test/" (not just "test.js")
      4. npm test
> @ test /home/paulsm/temp
> mocha
  Array
    #indexOf()
      ✓ should return -1 when the value is not present
  1 passing (6ms)

    - To test async code:
      - Add a "done()" callback
        ... OR ...
      - Return a Promise

    - Hooks:
      - { before(), after(), beforeEach(), afterEach() }

    - this.skip();
      <= "skip()" preferred to commenting tests out...
         Will skip all it, beforeEach/afterEach, and describe blocks within the suite.

    - Mocha's predecessor was "expresso":
        https://github.com/visionmedia/expresso
        <= Last commit was in 2016...

  - https://mochajs.org/: https://gist.github.com/samwize/8877226 > mocha-guide-to-testing.js >
    // # Mocha Guide to Testing
    // Objective is to explain describe(), it(), and before()/etc hooks
    
    // 1. `describe()` is merely for grouping, which you can nest as deep
    // 2. `it()` is a test case
    // 3. `before()`, `beforeEach()`, `after()`, `afterEach()` are hooks to run
    //       before/after first/each it() or describe().
    //       Which means, `before()` is run before first it()/describe()
    // 4. should.js is the preferred assertion library
    //       var should = require('should');
          <= Samuele Zaza's example above uses Chai "should": let should = chai.should();
===================================================================================================
