---
layout: post
title: "Building an API with Firebase"
tags: [Firebase, API, Js]
comments: true
---


In this post I will be building out an API with [Google’s Firebase](https://firebase.google.com/). I will be building out the `back-end` with `Firebase Cloud Functions` and `ExpressJS`.

Before I begin, I recommend you have the following setup:

1. A terminal setup on either a Windows, Linux, or Mac (OSX) computer
2. Node v13.1.0 installed
3. NVM installed [with the instructions here](https://github.com/nvm-sh/nvm)
4. A Google Account
5. [Postman](https://www.getpostman.com/) installed
6. [Firebase CLI](https://firebase.google.com/docs/cli) installed with a run of the terminal command `npm install -g firebase-tools`

I’m going to be referring to the code available at this [GitHub repo](https://github.com/andrewevans0102/how-to-build-a-firebase-api). The GitHub repo also contains a `Postman Collection` – I recommend importing that collection and using that to test your project. Please note that the app id in the URL paths is specific to my deployed project. You’ll need to change the app id to match a project you’ll be creating in the Firebase Console. If you don’t understand that yet, it’s okay – I’ll be discussing this more after the initial project is setup.

# THE BASICS

To start, I wanted to go over some basic concepts about what APIs are and how the technologies work. This is completely introductory, so feel free to skip this section if you are already familiar.

API stands for Application Programming Interface and refers to the method that computer systems use to communicate with one another. If you Google it, Google defines APIs as:

> a set of functions and procedures allowing the creation of applications that access the features or data of an operating system, application, or other service.

Basically, you build out an API so that your system can communicate with whatever you’re building. API’s can include basic REST endpoints for a website, or even the methods that define a software library you’ve built. Which brings up the next important thing, RESTful Services.

RESTful services refer to [Representational State Transfer](https://en.wikipedia.org/wiki/Representational_state_transfer), and take advantage of the HTTP protocol to pass data (via an API). The HTTP protocol is what we use everyday in websites and internet applications. RESTful services use different HTTP verbs (or methods) to pass data between different systems. Typical HTTP verbs that you will use include the following:

* GET = retrieving data
* POST = creating or updating data
* PUT = updating data
* DELETE = deleting data

I also mentioned `endpoints` earlier, that is just what I’ll be referring to when I mention a website or service I need to hit. An endpoint is just a fancy way to describe an address that my system will be hitting with an HTTP request. For more on the ins and outs of HTTP requests, [please refer to the wikipedia page](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Message_format).

# FIREBASE

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-22-at-3.59.49-pm.png" />
    <i>Firebase homepage</i>
</p>

I’m using Google’s Firebase platform for what I’m going to be building with this post. Firebase is a very powerful platform that developers can use to build applications quickly. Firebase provides common services that include:

* [Hosting](https://firebase.google.com/docs/hosting)
* [Realtime Database](https://firebase.google.com/docs/database)
* [NoSQL Database](https://firebase.google.com/docs/firestore)
* [Functions (similar to AWS lambdas)](https://firebase.google.com/docs/functions)
* [File Storage](https://firebase.google.com/docs/storage)
* much more!

In order to use Firebase, you simply need a Google account. This is why I asked you to have a Google account setup in the intro section.

In the rest of this post, I’m going to walk through setting up the backend for an API built with Firebase. If you’d like to dive more into Firebase, [please consult my post here](https://rhythmandbinary.com/2018/04/08/firebase/).

# INITIAL SETUP

To start, go to [the Firebase Console by clicking the link here](https://console.firebase.google.com/u/0/). You should see something like the following:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-22-at-7.29.16-am.png" />
    <i>Firebase dashboard</i>
</p>

Click the `add project` button and give your project a name. I recommend accepting the analytics steps as it helps you and Google. You can refer to the analytics information later on in the console (after your project is created).

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-22-at-7.30.17-am.png" />
</p>

Once your project is created, open it in the console and click “database” on the left hand navigation to see the following:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-22-at-7.32.37-am.png" />
</p>

Click `Create database` under `Cloud Firestore` to create your initial database. Select `test mode` to enable all reads and writes. You can build out rules to specify security on your database to lock it down further later. Please consult [the firebase documentation here for more on locking down your database instances](https://firebase.google.com/docs/firestore/security/get-started). You’ll also be asked to set a location; whatever it defaults should be fine, but you can also try to pick a datacenter closer to you. [Checkout this page for more on datacenter locations](https://firebase.google.com/docs/projects/locations).

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-22-at-7.32.47-am.png" />
</p>

Now that we’ve got the basic project pieces setup in the console, let’s switch over to your computer to setup the code.

# WRITING CODE

In this step, we’ll assume you already have the Firebase CLI on your machine. I recommend following the [instructions here to get that setup](https://firebase.google.com/docs/cli) if you haven’t done so already.

Next, go to your terminal and create a folder for your project with `mkdir my-project`

Next, cd into that folder and run firebase init and you should see something like the following:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-22-at-7.45.06-am.png" />
</p>

Select “Functions” in the options menu. Then select the Firebase App that you created before in the list shown on the next terminal output.

The next set of options are fairly straightforward.

* Select __JavaScript__
* Select __yes__ for linting
* select __yes__ to install dependencies
* and you’re good to go!

Next, cd into the created `functions` folder. When you run the init command, it will generate a `functions` folder for you to work with. The `functions` folder should have the following files:

* index.js
* node_modules folder
* package-lock.json
* package.json

Go ahead in your terminal editor of choice (highly recommend [VSCode](https://code.visualstudio.com/)) and look at `index.js` first.

# SERVERLESS APIS AND YOUR FIRST ENDPOINT

`Firebase Functions` enables you to use the ExpressJS library to host a __Serverless API__. __Serverless__ is just a term for a system that runs without physical servers. This is a bit of a misnomer because it technically does run on a server, however, you’re letting the provider handle the hosting aspect. Traditional APIs would require you to setup a server either in the cloud or on prem that can host your application. This means that developers would have to be in charge of OS patches and alerts, etc. In the world of Serverless, you don’t __have to worry about anything but your code__. This is one of the coolest parts of Firebase!

So in order to use ExpressJS with your project, erase the index.js file and paste the following lines there:

```js
const functions = require('firebase-functions');
const admin = require('firebase-admin');
const express = require('express');
const cors = require('cors');
const app = express();
app.use(cors({ origin: true }));

app.get('/hello-world', (req, res) => {
  return res.status(200).send('Hello World!');
});

exports.app = functions.https.onRequest(app);
```

The first lines that use require are pulling in the dependencies. We don’t yet have express or cors so lets install them in the terminal with the following two commands:

```bash
npm i express
npm i cors
```

The initial lines that have the require value are importing the libraries we’re going to be using. Here are the libraries in more detail:

* `firebase-functions` is an [npm module](https://www.npmjs.com/package/firebase-functions) that enables you to create functions
* `firebase-admin` is [the firebase admin SDK](https://firebase.google.com/docs/admin/setup) that enables your functions to control all of your backend Firebase services
* `express` is the [ExpressJS](https://expressjs.com/) library that lets you create a server instance
* `cors` is an [npm module](https://www.npmjs.com/package/cors) that allows your functions to run somewhere separate from your client. The app.use is just enabling CORS for your express server instance.

The section of the code with app.get creates a `hello world` endpoint. We’re using [express routing](https://expressjs.com/en/guide/routing.html) here. There are many ways to do this. I’m explicitly defining the routes just for the sake of ease. In an enterprise environment, you would likely use the express router and the code would probably look a little less verbose. For a more detailed dive into express routing, I [recommend checking out the tutorial here](https://scotch.io/tutorials/learn-to-use-the-new-router-in-expressjs-4).

For our purposes, the `app.get` is just making an __HTTP GET__ call and capturing the request in req and response in res. When the endpoint is called, it will return a `Hello World!` string with a HTTP status code of 200. Status Codes in HTTP are used to determine responses. There are many, but for the purposes of this tutorial 200 is success and 500 will return __error__.

The section of code with `exports.app = functions.https.onRequest(app);` exposes your express application so that it can be accessed. If you don’t have the exports section, your application won’t start correctly.

With packages installed and initial code setup, we can go ahead and run our project with `npm run serve`. This will server your functions locally, and should result in the output as follows:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-22-at-7.57.32-am.png" />
</p>

Now if you notice, I did get a warning about my node version. Ignore that for now, I will come back to that when we start actually accessing the database.

Also notice that the terminal output the localhost address of your API that is running locally. You’re going to call that directly from Postman as this is the `address` of your API. When running on `localhost`, the URLs for your app will look like the following:

```bash
[<------domain---->]  / [<-app id---> / [<-zone-->] / app / [<-endpoint->]
http://localhost:5000 / fir-api-9a206 / us-central1 / app / create
```

When we go to deploy, the only difference in the URL will be that http://localhost:500 will be replaced with zone + app id + “cloudfunctions.net” similar to the following:

```bash
[<------zone + app id + cloudfunctions.net-------->] / app / [<--endpoint-->]
https://us-central1-fir-api-9a206.cloudfunctions.net / app /   hello-world
```

You’ll need to update the Postman collection I mentioned in the intro with your app id values. In order to see your app id, go to the Firebase Console and click “project settings” like you see in the following screenshot:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-24-at-7.10.02-am.png" />
</p>

Once there, the project id (and some other settings) will be listed. Here I’ve circled it in this screenshot:
<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-24-at-7.10.46-am.png" />
</p>

Open up the postman collection and edit the `hello-world localhost` request under the `localhost` folder.

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-24-at-7.12.41-am.png" />
</p>

> If you’re still having issues with Postman, please consult the [instructions here](https://learning.getpostman.com/docs/postman/collections/creating_collections/).

As I mentioned, change the id value to match your project. The address that was listed in the terminal when you ran `npm run serve` should have this information as well. When your done, run the request from postman and you should see the following:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-22-at-8.07.51-am.png" />
</p>

Now that we have our initial endpoint setup, lets go ahead and add database calls.

# DATABASE CALLS

For the API we’re building, it’s just operations for a list of items. We’re just going to setup `Create`, `Read`, `Update`, and `Delete` (or `CRUD`) functions for this list of items.

In Firebase, you have two options for database use. You can use a traditional database, or you can use the `cloud firestore`. For this tutorial, we’re going to use cloud firestore because it’s easier to work with and more versatile. `Cloud firestore`is a [NoSQL database](https://en.wikipedia.org/wiki/NoSQL) which just means that your data is stored as `documents` within `collections`. This is fairly similar to how data is stored in `rows` in `tables` in a SQL database. NoSQL databases typically perform better and are easier to scale due to the nature of their data access and storage.

In the setup section, we added a `cloud firestore` instance to our project. We’re now going to access this. In order to interact with the `cloud firestore` using the [admin SDK](https://firebase.google.com/docs/admin/setup) locally, you’ll need to access it via a service account. Service accounts have keys that they use, you run these permissions by downloading a key file. To do this do the following:

Go to the Firebase Console, and open your application. Then click the little gear box and `users and permissions` as you see here:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-22-at-8.12.40-am.png" />
</p>

Then click the `service accounts` tab and you should see something like the following:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-22-at-8.15.02-am.png" />
</p>

If you notice, at the bottom of the screen they provide you with some code to run in your project. The only thing you’ll need is the permissions file to reference. Go ahead and click “Generate new private key” to be able to install that in your project. Then store that next to the index.js file in your functions folder we were just working with. You can name it whatever you want, I named mine `permissions.json` and then add the following to the top of your `index.js` file:

```js
var serviceAccount = require("./permissions.json");
admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: "https://fir-api-9a206..firebaseio.com"
});
const db = admin.firestore();
```

What these lines do is (1) load in your permissions file and then (2) use it to initialize your application. I also created a variable db to represent our firestore instance. This isn’t necessary, but makes the code cleaner later on.

With those values loaded up, let’s add a create endpoint to our application. Below the `hello-world` endpoint add the following:

```js
// create
app.post('/api/create', (req, res) => {
    (async () => {
        try {
          await db.collection('items').doc('/' + req.body.id + '/')
              .create({item: req.body.item});
          return res.status(200).send();
        } catch (error) {
          console.log(error);
          return res.status(500).send(error);
        }
      })();
  });
```

This code is creating an endpoint `/api/create-item` that you make a `POST` call to. When the POST call is made, it adds the `item` from the body to a collection in the database called `items` with an id of the value you passed in called “id” here. Collections in a NoSQL database are just a `collector` of `documents`. You could just as easily do this with a specific document. I tend to like collections when using firestore because they are easy to understand. You can also think of collections similarly to `tables` in a SQL Database.

>If you also notice, I’ve prefixed the endpoint with `/api` here. This is not required, but is typically convent with any API you create.

>The `id` value I am also specifying with my requests here. Firebase will do this for you, but I thought it was easier to understand if we explicitly define this in the requests.

So with this code added, lets go ahead and stop the server in the terminal (if you haven’t already) and restart it with a new `npm run serve`.

In the postman collection pull up the `create localhost` POST request. Modify the app id value as you did before, and attempt the request.

When you run it, you should see the following error:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-22-at-8.32.35-am.png" />
</p>

The Firebase admin SDK requires node version `8.13.0` or `10.10`. This is why I asked [to install nvm in the first section](https://github.com/nvm-sh/nvm). This is very easy to fix. nvm enables you to quickly switch node versions in your local terminal. In your terminal, run nvm use `10.10` and you should see the following:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-22-at-8.34.24-am.png" />
</p>

Now, go ahead and start your server again with `npm run serve` and hit the endpoint on postman. You should see a `200` success. In the terminal you can also see that the API was hit locally.

>If you run into an error about onRequestWithOpts or something similar, do an npm i firebase-tools. There was an error that recently came up with the CLI, version 7.1.0 fixed this. When I was writing this post I found my Firebase CLI version needed to be updated to fix this. Checkout [this issue for more](https://github.com/firebase/firebase-tools/issues/1480).

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-24-at-7.45.32-am.png" />
</p>

One additional cool feature is that you can directly see your data in cloud firestore. If you hop over to the firebase console, click on the `database` link on the left hand navigation and you should see something like the following picture:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-24-at-7.47.40-am.png" />
</p>

# BUILDING OUT THE DB ENDPOINTS

So now that we have the `create` endpoint setup, let’s go ahead and add the rest of the operations. We’re going to add the following:

* `/read-item/:item_id` = read a specific item (by ID)
* `/read-items` = read all items (total collection)
* `/update-item/:item_id` = update an item
* `/delete-item/:item_id` = delete an item

For the most part, all of the endpoints will be similar. The exception being when I pass query params with the item_id value. This all follows basic routing that you can see in the [ExpressJS documentation](https://expressjs.com/en/guide/routing.html). Additionally, for actually interacting with the Firebase Admin SDK, I recommend checking out the [Firestore API reference here](https://googleapis.dev/nodejs/firestore/latest/index.html).

> For a look at what these endpoints should look like (when finished) [checkout the index.js file on the GitHub repo here](https://github.com/andrewevans0102/how-to-build-a-firebase-api/blob/master/functions/index.js).

Here’s the code for the rest of the endpoints:

```js
// read item
app.get('/api/read/:item_id', (req, res) => {
    (async () => {
        try {
            const document = db.collection('items').doc(req.params.item_id);
            let item = await document.get();
            let response = item.data();
            return res.status(200).send(response);
        } catch (error) {
            console.log(error);
            return res.status(500).send(error);
        }
        })();
    });

// read all
app.get('/api/read', (req, res) => {
    (async () => {
        try {
            let query = db.collection('items');
            let response = [];
            await query.get().then(querySnapshot => {
            let docs = querySnapshot.docs;
            for (let doc of docs) {
                const selectedItem = {
                    id: doc.id,
                    item: doc.data().item
                };
                response.push(selectedItem);
            }
            });
            return res.status(200).send(response);
        } catch (error) {
            console.log(error);
            return res.status(500).send(error);
        }
        })();
    });

// update
app.put('/api/update/:item_id', (req, res) => {
(async () => {
    try {
        const document = db.collection('items').doc(req.params.item_id);
        await document.update({
            item: req.body.item
        });
        return res.status(200).send();
    } catch (error) {
        console.log(error);
        return res.status(500).send(error);
    }
    })();
});

// delete
app.delete('/api/delete/:item_id', (req, res) => {
(async () => {
    try {
        const document = db.collection('items').doc(req.params.item_id);
        await document.delete();
        return res.status(200).send();
    } catch (error) {
        console.log(error);
        return res.status(500).send(error);
    }
    })();
});
```

# DEPLOYING

So now we have a fully functional CRUD API. We’re ready to deploy! If you’re developing an enterprise application (or one that you will be maintaining) it is typical to build out a Continious Integration Continuous Deployment (CICD) pipeleine. This is basically a set of steps that are automated for delivering your application into production.

> There is a lot of documentation avasilable on CICD best practices. I recommend checking out my Angular-In-Depth blog post on [deploying an app with Firebase and CircleCI](https://blog.angularindepth.com/deploying-an-angular-site-to-firebase-with-circleci-ed881cb6a2fa).

With our API, we’re just interested in the actual deployment step. The Firebase CLI takes care of this for you with just a firebase deploy.

When you ran the firebase init command to initially build your project, the Firebase CLI already set the deploy step up as an NPM script. We can now deploy that created project with npm run deploy from the functions folder.

> npm scripts are very powerful. Most modern JavaScript applications make use of them in one way or another. I recommend checking out the [npm documentation for more](https://docs.npmjs.com/misc/scripts).

When you run `npm run deploy` from the `functions` folder, you should see something like the following output:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-22-at-3.38.33-pm.png" />
</p>

The line of the terminal output Function URL provides the endpoint of your functions that are deployed. Go back to postman collection, and checkout the deployed folder for a set of requests to hit your deployed API.

# CONNECTING A FRONTEND

So now that we’ve built out the API, I wanted to showcase what it would look like if you were to use it in a client application.

When discussing APIs, typically you will have a producer and a consumer. The producer is the API itself (or at least what provides the endpoints). The consumer is anything that uses those endpoints. Typically application developers will build a client application using JavaScript frameworks like [Angular](https://angular.io/), [react](https://reactjs.org/), [emberJS](https://emberjs.com/), [vue](https://vuejs.org/) and others.

Client applications are ran in the browser, and enable JavaScript code to be interpreted on the fly. This is particularly useful because many times developers just need somewhere to statically host their JavaScript `bundle.` This takes advantage of the JavaScript language as well as many advances that have happened with browsers today.

If you remember from the first sections, we’re using the code that I have put out on [the the GitHub repo here](https://github.com/andrewevans0102/how-to-build-a-firebase-api). This GitHub project has both our backend API code and a frontend Angular application that I built to interact with the API. The basic application really just consists of one main page and is considered a Single Page Application (SPA). The application just does basic CRUD operations on list items.

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-24-at-3.44.01-pm.png" />
</p>

While you are running the application, you can also see it in action in the browser’s console. If you open the console (right click “inspect” if you’re using Chrome) then you should see the following:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-24-at-3.50.40-pm.png" />
</p>

To use the client I built, first cd into the frontend folder of my [GitHub project](https://github.com/andrewevans0102/how-to-build-a-firebase-api). To make the application run against the endpoints of the API you deployed, you just need to open the `/frontend/src/environments/environment.ts` file.

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-23-at-9.23.59-pm.png" />
</p>

In this file, you’ll see a set of endpoints. To use the client I built, first cd into the frontend folder of my GitHub project. To make the application run against the endpoints of the API you deployed, you just need to change these values to match the URLs from your project respectively.

If you remember, once you deployed your API, the Firebase CLI output a domain to the terminal. The hosted endpoint nomenclature is super intuitive and is as follows:

```bash
[<------zone + app id + cloudfunctions.net-------->] / app / [<--endpoint-->]
https://us-central1-fir-api-9a206.cloudfunctions.net / app /   hello-world
```

Go ahead and replace the values for the endpoints in this environments file with the one’s from your project. Remember to always end each endpoint with the associated endpoint path like `/api/create` or `/api/read` respectively.

Once you’ve replaced the values, cd into the frontend directory and run a standard npm install to install the dependencies. Once the dependencies finish installing, use the Angular CLI command to start the app with ng serve.

>If you encounter errors with the CLI, check out [the Angular CLI documentation here](https://cli.angular.io/).

After running ng serve you should see something like the following in the terminal:

<p align="center">
    <img src="https://atevans85.files.wordpress.com/2019/07/screen-shot-2019-07-23-at-8.11.17-pm.png" />
</p>

This message just means the CLI has built the application (with [webpack](https://webpack.js.org/)) and it is currently running on port 4200. If you open your browser and go to `localhost:4200` you should see the app running.

If you look at the project’s app component, you’ll see that the actions are just doing `JavaScript` fetch calls to the various endpoints from the API we created:

```js
  async selectAll() {
    try {
      console.log(environment.readAll);
      console.log('calling read all endpoint');

      this.exampleItems = [];
      const output = await fetch(environment.readAll);
      const outputJSON = await output.json();
      this.exampleItems = outputJSON;
      console.log('Success');
      console.log(outputJSON);
    } catch (error) {
      console.log(error);
    }
  }

  // really this is create but the flow is that
  // click the "create item" button which appends a blank value to the array, then click save to actually create it permanently
  async saveItem(item: any) {
    try {
      console.log(environment.create);
      console.log('calling create item endpoint with: ' + item.item);

      const requestBody = {
        id: item.id,
        item: item.item
      };

      const createResponse =
        await fetch(environment.create, {
          method: 'POST',
          body: JSON.stringify(requestBody),
          headers:{
            'Content-Type': 'application/json'
          }
        });
      console.log('Success');
      console.log(createResponse.status);

      // call select all to update the table
      this.selectAll();
    } catch (error) {
      console.log(error);
    }
  }

  async updateItem(item: any) {
    try {
      console.log(environment.update);
      console.log('calling update endpoint with id ' + item.id + ' and value "' + item.item);

      const requestBody = {
        item: item.item
      };

      const updateResponse =
        await fetch(environment.update + item.id, {
          method: 'PUT',
          body: JSON.stringify(requestBody),
          headers:{
            'Content-Type': 'application/json'
          }
        });
      console.log('Success');
      console.log(updateResponse.status);

      // call select all to update the table
      this.selectAll();
    } catch (error) {
      console.log(error);
    }
  }

  async deleteItem(item: any) {
    try {
      console.log(environment.delete);
      console.log('calling delete endpoint with id ' + item.id);

      const deleteResponse =
        await fetch(environment.delete + item.id, {
          method: 'DELETE',
          headers:{
            'Content-Type': 'application/json'
          }
        });

      console.log('Success');
      console.log(deleteResponse.status);

      // call select all to update the table
      this.selectAll();
    } catch (error) {
      console.log(error);
    }
  }
```

Since the main point of this post was about creating the API, I won’t go into the details of how this Angular application works. I highly recommend you checkout the [Angular Documentation](https://angular.io/) as well as looking at the tutorials available. If you do some googling, you’ll find a lot of great `getting started` posts on Angular basics. I also highly recommend the blog [Angular-In-Depth](https://blog.angularindepth.com/) for more advanced learning.

# CLOSING THOUGHTS

This image has an empty alt attribute; its file name is duplo-1981724_1920.jpg
Congratulations! You’ve just deployed an API with Firebase. There are a lot of additional things that you an do with this API, but this shows you the basics. The client application that I showcased gets you started with ways to consume your API. I highly recommend reviewing the ExpressJS site tutorials for more in-depth ways to route calls and use middleware. I also highly recommend checking out my additional blog posts on Firebase:

* [Firebase](https://rhythmandbinary.com/2018/04/08/firebase/)
* [Why Firebase Cloud Functions are Awesome](https://blog.angularindepth.com/why-firebase-cloud-functions-are-awesome-f4faeab630f7)
* [How the AngularFire Library makes Firebase feel like Magic](https://blog.angularindepth.com/how-the-angular-fire-library-makes-firebase-feel-like-magic-1fda375966bb)
* [Why Building with a JAMstack is Awesome](https://blog.angularindepth.com/why-building-with-a-jamstack-is-awesome-49618fd21198)

I hope my post here has helped you to get started with building APIs with Firebase. Feel free to leave comments and thanks for reading!