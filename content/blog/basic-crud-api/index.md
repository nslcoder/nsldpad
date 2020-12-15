---
title: Building a Basic CRUD API in Node, Express and MongoDB with Mongoose  
date: "2020-12-15"  
description: "Check outhow a simple RESTful CRUD API is built using Node, Express and MongoDB with Mongoose"  
tags: ["Node.js", "Express", "MongoDB", "Mongoose", "API", "REST"]
---

I've been learning Node and Express since June 2020. I've made a few projects using these two but all the code I've written is line-for-line what I learned in the tutorials. To step up, I realized, I need to try to make something on my own. If not, take a tutorial project as a base and make significant changes to it by writing my own code, as in do the same thing in different ways or add new functionalities. And that's what I did here. I watched and read tons of tutorials and guides on building a CRUD API, tried to understand the whys and the whats and progressed from one line of code to another until I completed what I had set out to do, and while doing so, encountered my own problems and solved them one after another. 

## What I Learned
While building this basic CRUD API which follows the REST architecture, I learned:

- How to use the Dotenv module and load environment variables from a `.env` file
- How to connect to MongoDB using Mongoose, a object data modeling (ODM) tool
- How to create schema and model
- How to create, read, update and delete documents in MongoDB using Mongoose
- What  the async/await pattern is

## How I Built It
Please follow along as I show you how I made this CRUD API.

### Initial Setup
I initialized `npm` on the root folder named `basic-crud-api`.

```
npm init -y
```

This command creates a `package.json` file which gives a summary of the application and helps to manage dependencies. The `y` flag is used to specify yes to all the default settings.

Then, I installed the dependencies needed to create the CRUD API.

```
npm i express dotenv mongoose
```

This step creates a `node_modules` directory and a `package-lock.json` file in the root folder. The former stores the installed packages. The latter watches over the exact version of packages and locks the version including minor and patch releases.

**Express** is a web framework for Node.js that makes it easier for us to create web applications. It is built on top of the core `http` module.

**Dotenv** is a module that loads environment variables from a `.env` file into `process.env`. 

**Mongoose** is a object data modeling (ODM) tool which makes it easier to work with MongoDB. It is built atop Node's MongoDB driver. It allows us to create schemas and enforce them in the schemaless database that is MongoDB.

### Main File: Creating a Server and Connecting to MongoDB
In the root folder, I created a file named `app.js`. This is the file from where the execution starts. It's the entry point of our application.

In the file, I required the necessary modules. To load environment variables into the `process.env` object, especially for MongodDB database URL which I don't want to share, I created a `.env` file. I used the built-in Express JSON parser middleware, `express.json()`. I then connected to a MongoDB database using the `mongoose.connect` method. It returns a promise and that allows us to chain `then()` to handle a successful connection and `catch()` to catch and log any errors. We don't have to create a database before connecting to it. The `mongoose.connect` method will make one for us if we have specified the URL which is its first parameter. Then, I added the `listen` method on the `app`. It takes the port where the server will listen and a callback function. Without this method, we can't start the server.

After creating the routes and the controllers, I came back to the main file, required the `postRoutes` file and then used the `app.use()` to specify the routing functions for the path `/apis`. When the base of the requested route matches `/apis`, `postRoutes` will be executed.

```
const express = require("express");
const mongoose = require("mongoose");
require("dotenv").config();

const postRoutes = require("./routes/postroutes");

const app = express();

const port = process.env.PORT || 5000;
const dbUrl = process.env.DB_URL;

app.use(express.json()); 

// Connecting to MongoDB
mongoose.connect(dbUrl, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useFindAndModify: false,
    useCreateIndex: true
})
    .then(() => console.log("MongoDB connected!"))
    .catch(err => console.log(err));

app.get("/", (req, res) => {
    res.send({ message: "Namaste from the server!"})
})

app.use("/apis", postRoutes);

app.listen(port, () => {
    console.log(`The server is running at port ${port}.`);
});
```

### Schema and Model
MongoDB is a schemaless database. But with Mongoose we can create a schema for our MongoDB documents and validate them before adding to the database.

I created the `models` folder at the project root and inside it added a file named `post.js`. We obviously need Mongoose, so I required that. Mongoose provides the schema constructor. Using the constructor, I created `PostSchema` that has `title` and `body`. Both are `String` and required. The constructor also takes an options object. I added it with one option `timestamps` Setting it to `true` ensures the schema will have `createdAt` and `updatedAt` properties.

We need models too. They are constructors that take a schema and create documents based on it. They also read, update and delete documents in the database. I used the `mongoose.model` method to create a model. The method's first parameter is the model's name which is singular and with the first letter capitalized and its second parameter is the schema. Finally I exported the model to be used in the controllers.

```
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

const PostSchema = new Schema({
    title: {
        type: String,
        required: true
    },
    body: {
        type: String,
        required: true
    }
}, { timestamps: true });

const PostModel = mongoose.model("Post", PostSchema);

module.exports = PostModel;
```

### Controllers: Create, Read, Update and Delete Functions
In the `controllers` folder, I created a file named `crud.js` which holds all the CRUD functions. At the top of the file, I required the post model into the `Post` variable. The model offers us various methods we can use to manipulate the database. 

There are 5 functions in this file. All of them use the async/await pattern which is specified by the Mongoose docs. In brief, `async` turns a function asynchronous and makes the function return a promise, and `await`, which can only be used inside the `async` function, pauses the function execution in the line until the promise settles. The functions also use the `try...catch` statement for error handling. The application tries a bunch of statements in the `try` block, and if there are any errors in this block, the `catch` block will catch them and send a response. If an error isn't found, the app skips the `catch` block.

The first function is called `createPost`. As the name suggests, it creates a post. In the `try` block, whether `req.body` exists or not is checked. If it doesn't exist, a JSON response is sent with a message to provide a post. The `return` statement ensures the execution doesn't move forward if the `if` condition is matched. The `req.body` is technically an object and objects don't have the `length` property unlike arrays. So, I used `Object.keys` to turn `req.body` into an array. If the new array isn't empty, the execution moves forward. In the next line, `Post.create()` takes a document to create and saves it in the database. `req.body` supplies the document. Once the post is successfully created, `res.send` sends a JSON response. And, if the app encounters errors in the `try` block, the `catch` block will send the error object.

The second function is called `getPosts`. Inside the function, `Post.find()` finds the posts in the collection matching the filter criteria. If an empty object is used as the argument, the method returns all the documents in the collection. The method's returned query is an object which is held in the variable `posts`. So, like above, I turned the object into an array and checked its length. If it is empty, the execution stop right here and sends a JSON response. If it isn't empty, the response contains all the documents.

The third function is called `getPost`. It finds a document by an id and returns the matched document. The `Post.findById` method gets its argument id from `req.params.id`. In `req.params.id`, `id` is a named route parameter, which I will specify in the `postroutes.js` file, and `req.params`is an object where route parameters are properties. If the method doesn't find anything, then the variable `post` will be empty and the app will stop executing the function. If the document exists, `res.send` will send it to the client.

The fourth function is called `updatePost`. The function finds a document by its id and then updates new changes into it. `req.body` is checked if it is empty or not by turning it into an array with `Object.keys`. If it's empty, a JSON response with a message to provide new updates will be sent. If not, the execution moves forward. In the next line, `Post.findByIdAndUpdate` finds the document by `req.params.id` and updates it with the new changes contained in `req.body`. If the document with the specified id doesn't exist, the app will send a JSON response indicating the document doesn't exist. If the document is successfully updated, the app will send the "Post is updated" message.

The final function is called `deletePost`. Inside the function, `Post.findByIdAndDelete` finds a document by `req.params.id` and deletes it from the database. The variable `deletedPost` contains the deleted document. If it is empty, it means the document wasn't in the database. If the document was there and it is successfully deleted, the app will send the "Post is updated" message.

Once I created all the functions, I exported it with `module.exports`.

```
const Post = require("../models/post");

// Create a new post
const createPost = async (req, res) => {
    try {
        if(!Object.keys(req.body).length) return res.send({ message: "Please provide a post." })
        const post = await Post.create(req.body);
        res.send({ message: "Post is created."});
    } catch(err) {
        res.send(err);
    };
};

// Get all the posts
const getPosts = async (req, res) => {
    try {
        const posts = await Post.find({});
        if(!Object.keys(posts).length) return res.send({ message: "There are no posts." });
        res.send(posts);
    } catch(err) {
        res.send(err);
    };
};

// Get a post by its id
const getPost = async (req, res) => {
    try {
        const post = await Post.findById(req.params.id);
        if(!post) return res.send({ message: "Post doesn't exist." });
        res.send(post);
    } catch(err) {
        res.send(err);
    };
};

// Update a post 
const updatePost = async (req, res) => {
    try {
        if(!Object.keys(req.body).length) return res.send({ message: "Please provide new updates." });
        const updatedPost = await Post.findByIdAndUpdate(req.params.id, req.body);
        if(!updatedPost) return res.send({ message: "Post doesn't exist." });
        res.send({ message: "Post is updated." });
    } catch(err) {
        res.send(err);
    };
};

// Delete a post
const deletePost = async (req, res) => {
    try {
        const deletedPost = await Post.findByIdAndDelete(req.params.id);
        if(!deletedPost) return res.send({ message: "Post doesn't exist." });
        res.send({ message: "Post is deleted." });
    } catch(err) {
        res.send(err);
    };
};

module.exports = {
    createPost: createPost,
    getPosts: getPosts,
    getPost: getPost,
    updatePost: updatePost,
    deletePost: deletePost
};
```

### Routing
Express provide the router object which we can use to set up various routes for our application. The router object is basically an isolated instance of middleware and route which only provides middleware and routing functions.

I created the `routes` folder with a file named `postroutes.js`. In the file, I required `express` and created `router` with `express.Router()`. Then I imported the CRUD functions from the controllers using destructing assignment. 

I set up routes using the `router` object and HTTP verbs. `POST` is for creating a document. `GET` is for getting one or all documents. `PUT` is for updating a document. And, `DELETE` is for deleting a document. I've used the `id`route parameter whose value is supplied from the `_id` field of the documents in the database. `router` needs to be exported and `module.exports` does the job.

```
const express = require("express");
const router = express.Router();

const { createPost, getPosts, getPost, updatePost, deletePost } = require("../controllers/crud");

router.post("/", createPost);
router.get("/", getPosts);
router.get("/:id", getPost);
router.put("/:id", updatePost);
router.delete("/:id", deletePost);

module.exports = router;
```

### Testing
The CRUD API is completed. All that is needed is testing whether it works or not. I used Postman to test the routes and they all work perfectly.

You can find the entire code at this [GitHub repo](https://github.com/nslcoder/basic-crud-api).