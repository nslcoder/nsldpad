---
title: How to seed a MongoDB database  
date: "2022-01-29"  
description: "A short guide on how to populate a MongoDB database with initial data"  
tags: ["MongoDB", "Mongoose", "Database", "Node.js"]
---

During development, you will need to check if your code, especially CRUD operations, are working or not. To do so, you will need to populate your database with initial data. That is what database seeding is.

In this post, I will show you how you can seed a MongoDB database. It's easy and won't take long.

## Initial Setup

Create a project folder and initialize it with `npm init`. You know the drill: either provide your own values or keep on pressing the enter button to use the default values. This is what my `package.json` looks like:

![package.json](C:\Users\User\Desktop\mongodbseeder\package.json content.png)

Now install `mongoose`. This is the only third-party module we need. And, then create the `seeder.js` file.

## Schema and Model

We will need a schema to define the shape of a MongoDB document and then we will create a model from the schema.

Create a folder titled `models` and inside it create a file `Person.js`. In the file, add the following code:

![Schema and model](C:\Users\User\Desktop\mongodbseeder\person schema & model.png)

As you can see in the image, the schema has four mandatory fields: first name, last name, email and country. They are all of the `String` type. The `timestamps` option is there to create `createdAt` and `updatedAt` fields for each document. We then create a model using `mongoose.model()` and export it.

## Mock Data

We will get our mock data for seeding from this amazing tool called [Mockaroo](https://www.mockaroo.com/). Download a JSON file of 100 rows and bring it to your project root. It's easy to play with the options in the Mockaroo website and get the data, so I won't be showing that in this post.

The image below shows partial JSON data that will go into our database. 

![Mock data](C:\Users\User\Desktop\mongodbseeder\mock data json.png)

## Seeder

Finally, we will work on `seeder.js`, the main file of our little program. Ensure your file has the following code.

![seeder.js](C:\Users\User\Desktop\mongodbseeder\seeder.png)

In the file, there is an async function named `seeder`. It at first connects to the MongoDB database `mockDB` in your local machine. If the database of the same name already exists, this method simply connects to it. If there is no database of such name, it will create a new one. 

The `seeder` function then reads `MOCK_DATA.json` and stores all its contents in the `data` variable. It's JSON data so it needs to be parsed and converted to valid JavaScript value. That is done by `JSON.parse()`.

Then, the function calls the `create` method on the `Person` model. The `parsedData` variable holds a single array with 100 objects. Each object corresponds to a single document. Pass it to `create` and 100 documents will be created in the database. If you check the Mongoose documentation, you will see that to create multiple documents you need to wrap all of them in a single array. Since `parsedData` is already an array, we don't need to do anything other than passing it to the `create` method.

Once the seeding is complete, the `seeder` function will print the message to the console and exit from the process.

If you check it via MongoDB Compass or mongo shell, you will see the `mockDB` database with a single collection that has 100 documents.

 If you want all the code for this, please get it from this [repo](https://github.com/nslcoder/mongodbseeder).

