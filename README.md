# mern-project-tutorial

This tutorial will show you how to build a full-stack MERN application;an employee database, with the most current tools available. Before you begin, make sure that you are familiar with Node.js and React.js basics and have Node and Create React App installed. You will also need access to the MongoDB Atlas database for this tutorial. The full code is available on this GitHub repo.

## What is the MERN Stack?

The MERN stack is a web development framework made up of the stack of **MongoDB**, **Express.js**, **React.js**, and **Nodejs**.  

![MERN stack visualized](https://webimages.mongodb.com/_com_assets/cms/kobuybqq12c9ya16f-mernstack_visualized.png?auto=format%2Ccompress)

When you use the MERN stack, you work with **React** to implement the presentation layer, **Express** and **Node** to make up the middle or application layer, and **MongoDB** to create the database layer.

In this MERN stack tutorial, we will utilize these four technologies to develop a basic application able to record the information of employees and then display it using React.

## Setting Up Project

MERN lets us create full-stack solutions. We will be creating a MERN stack project. For this project, we will create both a back end and a front end. The front end will be implemented with React and the back end will be implemented with MongoDB, Node, and Express. We will call the front end `client` and the back end `server`.

Start by creating an empty directory: `mern`. 
  - This folder will hold all our files after we create a new project. 
  - Then we will create a React app—client—in it.
  
```
mkdir mern
cd mern
npx create-react-app client
```

Then, we create a folder for the back end and name it `server`.

```
mkdir server
```
We will jump into the server folder we created previously and create the server. Then, we will initialize `package.json` using npm init.

```
cd server
npm init -y
```
We will also install the dependencies.
```
npm install mongodb express cors dotenv
```

The command above uses a couple of keywords:

  - **mongodb** command installs MongoDB database driver that allows your Node.js applications to connect to the database and work with data.
  - **express** installs the web framework for Node.js. (It will make our life easier.)
  - **cor**s installs a Node.js package that allows cross origin resource sharing.
  - **dotenv** installs the module that loads environment variables from a `.env` file into `process.env` file. This lets you separate configuration files from the code.

We can check out installed dependencies using the `package.json` file. It should list the packages along with their versions.

After we have ensured that dependencies were installed successfully, we create a file called `server.js` with the following code.

**mern/server/server.js**

```
const express = require("express");
const app = express();
const cors = require("cors");
require("dotenv").config({ path: "./config.env" });
const port = process.env.PORT || 5000;
app.use(cors());
app.use(express.json());
app.use(require("./routes/record"));
// get driver connection
const dbo = require("./db/conn");
 
app.listen(port, () => {
  // perform a database connection when server starts
  dbo.connectToServer(function (err) {
    if (err) console.error(err);
 
  });
  console.log(`Server is running on port: ${port}`);
});
```
Here, we are requiring express and cors to be used. `const port process.env.port` will access the port variable from the `config.env` we required.

## Connecting to MongoDB Atlas

Connect our server to the database. We will use **MongoDB Atlas** as the database. MongoDB Atlas is a cloud-based database service that provides robust data security and reliability. MongoDB Atlas provides a free tier cluster that never expires and lets you access a subset of Atlas features and functionality.

Follow the [Get Started with Atlas](https://docs.atlas.mongodb.com/getting-started/?_ga=2.57207850.1643556021.1663179738-1895383455.1663179738) guide to create an account, deploy your first cluster, and **locate your cluster’s connection string**.

With the connection string, go back to the `server` directory and create a `config.env` file. There, assign the connection string to a new `ATLAS_URI` variable. Once done, your file should look similar to the one below. Replace <username> and the <password> with your database username and password.

**mern/server/config.env**

```
ATLAS_URI=mongodb+srv://<username>:<password>@sandbox.jadwj.mongodb.net/employees?retryWrites=true&w=majority
PORT=5000
```
Create a folder under the `server` directory, `db` and inside it, a file `conn.js`. There we can add the following code to connect to our database.

**mern/server/db/conn.js**

```
const { MongoClient } = require("mongodb");
const Db = process.env.ATLAS_URI;
const client = new MongoClient(Db, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});
 
var _db;
 
module.exports = {
  connectToServer: function (callback) {
    client.connect(function (err, db) {
      // Verify we got a good "db" object
      if (db)
      {
        _db = db.db("employees");
        console.log("Successfully connected to MongoDB."); 
      }
      return callback(err);
         });
  },
 
  getDb: function () {
    return _db;
  },
};
```

## Server API Endpoints

Now it's time for the **Server API endpoint**. Start by creating a `routes` folder and adding `record.js` in it. Navigate back to your `server` directory and create the new directory and file:

```
cd ../server
mkdir routes
touch routes/record.js
```

The `routes/record.js` file will also have the following lines of code in it.

**mern/server/routes/record.js**

```
const express = require("express");
 
// recordRoutes is an instance of the express router.
// We use it to define our routes.
// The router will be added as a middleware and will take control of requests starting with path /record.
const recordRoutes = express.Router();
 
// This will help us connect to the database
const dbo = require("../db/conn");
 
// This help convert the id from string to ObjectId for the _id.
const ObjectId = require("mongodb").ObjectId;
 
 
// This section will help you get a list of all the records.
recordRoutes.route("/record").get(function (req, res) {
 let db_connect = dbo.getDb("employees");
 db_connect
   .collection("records")
   .find({})
   .toArray(function (err, result) {
     if (err) throw err;
     res.json(result);
   });
});
 
// This section will help you get a single record by id
recordRoutes.route("/record/:id").get(function (req, res) {
 let db_connect = dbo.getDb();
 let myquery = { _id: ObjectId(req.params.id) };
 db_connect
   .collection("records")
   .findOne(myquery, function (err, result) {
     if (err) throw err;
     res.json(result);
   });
});
 
// This section will help you create a new record.
recordRoutes.route("/record/add").post(function (req, response) {
 let db_connect = dbo.getDb();
 let myobj = {
   name: req.body.name,
   position: req.body.position,
   level: req.body.level,
 };
 db_connect.collection("records").insertOne(myobj, function (err, res) {
   if (err) throw err;
   response.json(res);
 });
});
 
// This section will help you update a record by id.
recordRoutes.route("/update/:id").post(function (req, response) {
 let db_connect = dbo.getDb();
 let myquery = { _id: ObjectId(req.params.id) };
 let newvalues = {
   $set: {
     name: req.body.name,
     position: req.body.position,
     level: req.body.level,
   },
 };
 db_connect
   .collection("records")
   .updateOne(myquery, newvalues, function (err, res) {
     if (err) throw err;
     console.log("1 document updated");
     response.json(res);
   });
});
 
// This section will help you delete a record
recordRoutes.route("/:id").delete((req, response) => {
 let db_connect = dbo.getDb();
 let myquery = { _id: ObjectId(req.params.id) };
 db_connect.collection("records").deleteOne(myquery, function (err, obj) {
   if (err) throw err;
   console.log("1 document deleted");
   response.json(obj);
 });
});
 
module.exports = recordRoutes;
```

If you run the application at this point, you will get the following message in your terminal as the connection establishes.

```
node server.js

Server is running on port: 5000
Successfully connected to MongoDB.
```
Now, we will start working on the front end.













