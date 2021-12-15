---
author: michal
title: "Automating data migration testing"
date: 2020-04-12T11:12:28+01:00
categories:
  - Technology
tags:
  - databases
  - data migration
  - testing
  - automation
---

Data migrations are notoriously difficult to test. They take a long time to run on large datasets. They often involve heavy, inflexible database engines. And they're only meant to run once, so people think it's throw-away code, and therefore not worth writing tests for. Not so.

<!--more-->

With a few modern tools, we can test migrations just like we do our production code. Let me show you one approach.

## Bird's-eye view

We'll be moving data from a [PostgreSQL](https://www.postgresql.org/) database to [MongoDB](https://www.mongodb.com/) using [Node.js](https://nodejs.org/). Obviously, you can substitute any of these. Anything that runs in Docker, like PostgreSQL, or in-memory, like MongoDB, will fit the approach. Also, any programming language is fine, if you're comfortable with it. I chose Node.js for the plasticity it offers, which is useful to have in data transformations.

We'll split the migration into several functions, organized around data scopes (like companies, users, whatever you have in the database), each function receiving an instance of the PostgreSQL and MongoDB client, i.e. `migrateUsers(postgresClient, mongoClient)`. This way, our test code can provide connections to the test databases, and production code to production ones.

Each migration function will have its accompanying test suite, i.e. `migrate-users.js` will have `migrate-users.specs.js`.

```
         /-> migrateCompanies()  <-- migrate-companies.spec.js
index.js --> migrateUsers()      <-- migrate-users.spec.js
         \-> migrateFizzBuzzes() <-- migrate-fizzBuss.spec.js
```

The test suites will also share a file we'll call `test-init.js` containing code that'll launch both test databases and connect clients, then shut them down after tests finish.

The article comes with an [accompanying GitHub repository](https://github.com/Pragmatists/data-migration-testing), containing all the code we're discussing here. I'll solely be quoting excerpts to illustrate key concepts, and you may prefer to follow them by studying the full source code.

## Install the tools

Install [Docker Engine](https://docs.docker.com/engine/install/) with [Docker Compose](https://docs.docker.com/compose/install/) for your platform. Make sure you can run the `docker` command without requiring user rights elevation (i.e. without `sudo`). On Linux, this is typically achieved by adding your current user to the docker group:

``` Bash
sudo usermod -aG docker $USER
```

after which you'll need to log out and back in again.

You'll also need [Node.js](https://nodejs.org/).

Start a new `npm` project by going through the `npm init` steps, then install the dependencies we'll be using:

``` Bash
npm install -E mongodb pg
npm install -ED jest mongodb-memory-server shelljs
```

We'll use [jest](https://jestjs.io/) to run the tests, but again, go with whatever suits your needs.

## Configure the tools

We'll be running PostgreSQL in Docker, so let's first configure that. Create a [file called `docker-compose.yml`](https://docs.docker.com/compose/compose-file/) and paste in the following:

``` YAML
version: '3'
services:
  postgresql:
    image: postgres:12   #1
    container_name: postgresql-migration-test
    environment:
      POSTGRES_PASSWORD: 'password'   #2
    ports:
      - "5432"   #3
```

This will configure a Docker container with the [latest PostgreSQL 12.x](https://hub.docker.com/_/postgres) (1), default user and `password` for password (2). (Never use that outside of testing, obviously.) The above also says that PostgreSQL's standard port `5432` will be exposed to a *random* port on the host (3). Thanks to that, you can safely launch the container on a host machine where PostgreSQL is already running, thus avoiding test failures due to port conflicts. We'll have a way to find the right port to connect to in tests.

Now, if you run the command `docker-compose up` in the same directory, the container with the database should start and display PostgreSQL's standard output. If it did, great; shut it down with <kbd>Ctrl+c</kbd>, followed by `docker-compose down` and we'll configure the test suite to do the starting and stopping later.

Move on to your `package.json` and add the tasks we'll be using into the `scripts` section:

``` JSON
"scripts": {
  "start": "node src/index.js",
  "test": "jest --runInBand"   //1
},
```

With that, `npm start` will run your migration and npm test will run the tests.

Notice the `--runInBand` parameter for jest (1). That tells `jest` to run every test suite *in sequence*, whereas by default these run in parallel. This is important, because all test suites share the same database instances and running them simultaneously would cause data conflicts. Don't worry about speed, though---we're not dealing with a lot of data here, so tests will still be fast.

`jest` itself also needs a bit of configuration, so add a file called `jest.config.js` with the following content:

``` JS
module.exports = {
    testEnvironment: "jest-environment-node",
    moduleFileExtensions: ["js"],
    testRegex: ".spec.js$",
}
```

This simply tells `jest` to treat any file ending with `.spec.js` as a test suite to run, and to use its `node` environment (as opposed to the default `jsdom` useful for code meant to run in the browser).

Ready. Let's start testing.

## Set up the test environment

We'll start with creating the `src/test-init.js` file, which will deal with database startup and teardown, and will be included by every test suite. It's a crude method, I know, and `jest` [claims to have better ways](https://jestjs.io/docs/en/configuration#testenvironment-string), but I'll exclude them from this text for clarity.

``` JS
const shell = require("shelljs");
const {MongoMemoryServer} = require("mongodb-memory-server");
const {getPostgresClient} = require("./postgres-client-builder");
const {afterAll, afterEach, beforeAll} = require("@jest/globals");
const {getMongoClient} = require("./mongo-client-builder");

let clients = {};   //1
let mongod;

const dockerComposeFile = __dirname + '/../docker-compose.yml';

beforeAll(async () => {
    console.log('Starting in memory Mongo server');
    mongod = new MongoMemoryServer({autoStart: false});   //2
    await mongod.start();

    const mongoUri = await mongod.getUri();
    clients.mongo = await getMongoClient(mongoUri);   //3
    clients.mongoDb = clients.mongo.db();

    console.log('Starting Docker container with PostgreSQL');
    shell.exec(`docker-compose -f ${dockerComposeFile} up -d`);  //4
    await new Promise(resolve => setTimeout(resolve, 1000));   //5

    const postgresHost = shell.exec(`docker-compose -f ${dockerComposeFile} port postgresql 5432`).stdout;   //6
    const postgresUri = `postgresql://postgres:password@${postgresHost}`;

    clients.postgres = await getPostgresClient(postgresUri);   //7
});

afterAll(async () => {
    await clients.mongo.close();   //8
    await clients.postgres.end();

    await mongod.stop();
    console.log('Mongo server stopped')
    shell.exec(`docker-compose -f ${dockerComposeFile} down`);   //9
    console.log('PostgreSQL Docker container stopped');
});

module.exports = {
    clients,
}
```

Let's go through this step by step. First, we're declaring and exporting the variable `clients` (1) which will allow us to pass database clients to the test suites.

`beforeAll` is all about starting the databases. The first part starts MongoDB and follows standard instructions from the [`mongodb-memory-server` package](https://www.npmjs.com/package/mongodb-memory-server) (2). There's a call to a custom function `getMongoClient()` (3) here that I left out which simply [builds a `MongoClient` instance from the `mongodb` package](https://mongodb.github.io/node-mongodb-native/3.6/quick-start/quick-start/#connect-to-mongodb).

Then it gets interesting. `shell` allows us to run shell commands, and we're simply calling `docker-compose up` (4) to start the PostgreSQL container we configured earlier. That's followed by a one-second wait (5) to let the database start accepting connections on the port. Next, we need to see what port on the host machine was assigned to the container. That's what `docker-compose port` does (6) and it also outputs the local IP address, i.e. `0.0.0.0:12345` so we can store that directly in postgresUri and use another custom function `getPostgresClient()` (7) to [build the PostgreSQL client instance](https://node-postgres.com/#getting-started).

You'll notice I use two extra options for `docker-compose`: `-f` points to the `docker-compose.yml` file which is in the parent directory of `src/test-init.js` and `-d` runs the containers as daemons, so they're not holding up the tests.

`afterAll` reverses the process. First, we disconnect the clients from the databases (8). Then we stop MongoDB and finally shut down the PostgreSQL Docker container with `docker-compose down` (9).

The `src/test-init.js` file needs one more function that will clean both databases after each test.

``` JS
afterEach(async () => {
    const tables = await clients.postgres.query(
        `SELECT table_name FROM information_schema.tables WHERE table_schema ='public';`
    );
    await Promise.all(tables.rows.map(row => clients.postgres.query(`DROP TABLE ${row.table_name}`)));

    const collections = await clients.mongoDb.collections();
    await Promise.all(collections.map(collection => clients.mongoDb.dropCollection(collection.collectionName)));
});
```

We're simply dropping all tables from PostgreSQL and collections from MongoDB, leaving a clean slate for the next test. You can truncate the tables instead, so you won't need to recreate them before each test. Either way is fine.

The code in `src/test-init.js` is *the* magic sauce of the solution. It'll ensure that there are two, real databases running for your migration code to operate on, make sure they're clean for each test case, and shut them down after all the tests finish running.

## Create the test suite

We're ready to start writing test code. Our actual migration code will be split into functions for each data type. We could be migrating user accounts, so we'll have a file called `src/migrate-users.js` and inside:

``` JS
async function migrateUsers(postgresClient, mongoClient) {
    // Migration logic goes here
}
```

We'll be calling this function in our test suite at `src/migrate-users.spec.js`.

Start with a `beforeEach()` call to create the necessary tables (because we're dropping them; if you opted to truncate them, you can skip this step):

``` JS
beforeEach(async () => {
    await clients.postgres.query(`CREATE TABLE users (id integer, username varchar(20))`);
});
```

We're not adding any indexes or constraints, as we'll be dealing with small amounts of strictly controlled data. That said, your specific use case may warrant it.

Now, we'll write the actual test case:

``` JS
it('migrates one user', async () => {
    await clients.postgres.query(`
        INSERT INTO users (id, username) 
        VALUES (1, 'john_doe')
        `);

    await migrateUsers(clients.postgres, clients.mongo);

    const users = await clients.mongoDb.collection('users').find().toArray();
    expect(users).toHaveLength(1);
    expect(users[0].username).toEqual('john_doe');
});
```

It looks simple and it is. We're inserting a user into the source database (PostgreSQL), then calling the `migrateUsers()` function, passing in the database clients connected to our test databases, then fetching all users from the target database (MongoDB), verifying that exactly one was migrated and that its `username` value is what we're expecting.

Try running the test, either with an `npm test` or from within your favorite IDE.

You can continue building out your test suite, adding more data, conditions and cases. Some of the items you should test for, are:

* data transformations---if you're mapping or changing data,
* missing or incorrect data---make sure your code expects and handles these,
* data scope---check that you're only migrating the records you want,
* joins---if you're putting together data from several tables, what if there is no match? What if there are multiple matches?
* timestamps & time zones---when you're moving these between tables, make sure they still describe the same point in time.

## Run the actual migration

You can test-drive and build your whole migration code with the above setup, but eventually you'll want to run it on actual data. I suggest you *try that early* on a *copy* of the database you'll be migrating, because that'll uncover conditions you haven't considered, and will inspire more test cases.

You'll have several migration functions for various data scopes, so what you'll need now is the `src/index.js` file to call them in sequence:

``` JS
run().then().catch(console.error);

async function run() {
    const postgresClient = await buildPostgresClient();
    const mongoClient = await buildMongoClient();

    console.log('Starting migration');

    await migrateCompanies(postgresClient, mongoClient);
    await migrateUsers(postgresClient, mongoClient);
    await migrateFizzBuzzes(postgresClient, mongoClient);

    console.log(`Migration finished`);
}
```

And you can run the actual migration with `npm start`.

## Build out the test suite

Your test suite will quickly grow in scope and complexity, so here are a few more ideas to help you on your way:

* `docker-compose.yml` can be extended with more services, if you need them: additional databases, external services, etc. They'll all launch together via the same `docker-compose up` command.
* You'll be creating the same tables in several test suites, so write a shared function for that, i.e. `createUsersTable()` and call it inside your `beforeEach()` block.
* You will also be inserting a lot of test data, often focusing on one or more fields. It's useful to have generic, shared functions for inserting test records, with the ability to override specific fields, like the one below. Then you can call it in your tests with `await givenUser({username: "whatever"})`.

    ``` JS
    async function givenUser(overrides = {}) {
        const user = Object.assign({
            id: '1',
            username: 'john_doe',
        }, overrides);

        await clients.postgres.query(`
            INSERT INTO users (id, username)
            VALUES (${user.id}, '${user.username}')
            `);

        return user;
    }
    ```

* You may need to make your test record IDs unique. Sometimes that's easy, like with MongoDB, where calling `new ObjectId()` generates a unique value, or using an integer sequence; sometimes you can use packages like [uuid](https://www.npmjs.com/package/uuid); at other times, you'll have to write a simple function that generates an ID in the format you require. Here's one I used, when I needed unique 18-character IDs that were also sortable in the order they were generated:

    ``` JS
    let lastSuffix = 0;
    
    function uniqueId() {
        return new Date().toISOString().replace(/\D/g, '') + (lastSuffix++ % 10);
    }
    ```

## Check out the sample repository

All of the code quoted above is available in a GitHub repository at [@Pragmatists/data-migration-testing](https://github.com/Pragmatists/data-migration-testing). You can pull it, run `npm install` and, with a bit of luck, it should run out of the box. You can also use it as a base for your own data migration suite. There's no license attached---it's public domain.

&nbsp;

Thanks to Zbigniew Artemiuk for a review and feedback. Reposted with permission from [Pragmatists](https://pragmatists.com/).
