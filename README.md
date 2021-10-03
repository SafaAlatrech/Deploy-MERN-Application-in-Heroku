# Deploy-MERN-Application-in-Heroku : 
## Before you start : 
You have a basic knowledge of MERN stack and mongoose.
You already have a MERN application set up (connected to the database) which is running locally. Alternatively you can use the deploy-mern repository to get started. This blog post will be based on the structure of this project.
If you have not done it yet, initialise a git repository inside the root folder of your project.
$ cd your-project
$ git init

## Let’s start!
Downloading and installing Heroku
You can install the Heroku Command Line Interface (CLI) from this link. To check it was installed successfully, you can run the following command:
$ heroku --version
heroku/7.47.11 win32-x64 node-v12.16.2
Once the installation has been completed, you will be able to use the Heroku command from your terminal. But before we continue, create a Heroku account here. Then, you will be able to log in from the terminal:
$ heroku login
This will open a tab to login from the browser. Once you have logged in we will continue by making some modifications.

## Modifying the server.js : 
NOTE: You might see on some occasions -such as in this blog post- that server.js will be used to name the entry point. Nevertheless, it is also common to use index.js to name the entry point instead. The deploy-mern repository uses index.js. Therefore, when we talk about the server.js for the rest of the blog post you might want to refer to the index.js.

THE PORT

You may have defined the PORT to be 5000 as default. But, when the application is deployed with Heroku, this port might not be available, so we will define the PORT as follows:

server.js
const PORT = process.env.PORT || 5000
In this way, when the application is running locally, the server will be hosted at PORT 5000 because process.env.PORT is not defined, but once is deployed, Heroku will run the server in any available PORT.

## MONGODB ATLAS AND THE CONNECTION STRING

As you already have built your MERN application, you may need to consider using MongoDB Atlas. After getting registered and logging in the online platform, you can follow the next steps:

Create a new project from the atlas dashboard.

Create a cluster which will include your database. This will take some minutes. You will need to indicate the cloud provider and the region you are located in.
image

It is important to note, that you may need to whitelist your connection IP address in order to access the cluster. (Network Access >> Add IP Address >> Allow access from anywhere >> Confirm).
image

It's time to connect your application with the database. To do so, click "connect" in the Clusters tab. As it will be the first time connecting the application, you will need to create a user and a password.
image

Now, click "choose a connection method". After selecting the "connect your application" method, you can copy the connection string.

## The string will look like this:

"mongodb+srv://username:<password>@<cluster>/<database>?retryWrites=true&w=majority";
Where <password>, <cluster> and <database> correspond to your own credentials. (Note: The password corresponds to the database user, not your Atlas account. Do not include < or > when filling in the details).

Now, you can add this string to your server.js to complete the connection.

server.js:
mongoose
  .connect(
"mongodb+srv://username:<password>@<cluster>/<database>?retryWrites=true&w=majority";
    {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    }
  )
  .then(() => console.log("MongoDB has been connected"))
  .catch((err) => console.log(err));
  
Nevertheless, you might want to consider defining the string in a .env file, which will be ignored with .gitignore. This means that the .env file will not be pushed to GitHub. To do this, complete the following steps:

Run the following command to install the dotenv dependency, which will load environment variables from a .env file into process.env.

    $ npm install dotenv

Create a .env file in the root folder and define your connection string.
.env:

    MONGODB_CONNECTION_STRING = "mongodb+srv://username:<password>@<cluster>/<database>?retryWrites=true&w=majority",

Create a .gitignore file in the root of your project and include the .env file.
.gitignore:

    # See https://help.github.com/articles/ignoring-files/ for more about ignoring files.
    .env

Now, you can access the variables defined in the .env file from anywhere. So the long string will be substituted and the server.js will look like this.

server.js:
    require("dotenv").config()

    mongoose
     .connect(
         process.env.MONGODB_CONNECTION_STRING,
             {
               useNewUrlParser: true,
               useUnifiedTopology: true,
             }
     )
     .then(() => console.log("MongoDB has been connected"))
     .catch((err) => console.log(err));

## PRODUCTION BUILD : 

Now we can run the following command in the terminal to create a production build, which will serve.
$ cd client
$ npm run build
As a result, a new folder called build will be created inside the client folder. This will include a static folder and an index.html.

In the next step, we will use the path module, which provides utilities for working with file and directory paths.

Now, we will include the following lines in the server.js.

server.js
// Accessing the path module
const path = require("path");

// Step 1:
app.use(express.static(path.resolve(__dirname, "./client/build")));
// Step 2:
app.get("*", function (request, response) {
  response.sendFile(path.resolve(__dirname, "./client/build", "index.html"));
});
Step 1 will import the client build folder to the server.

Step 2 will ensure that the routes defined with React Router are working once the application has been deployed. It handles any requests by redirecting them to index.html.

At this stage, our server.js should look like this:

server.js:
const express = require("express");
const mongoose = require("mongoose");
const bodyParser = require("body-parser");
require("dotenv").config();

const cors = require("cors");

const app = express();
app.use(cors());

//import your models
require("./models/quote");

mongoose
  .connect(
    process.env.MONGODB_CONNECTION_STRING,
    {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    }
  )
  .then(() => console.log("MongoDB has been connected"))
  .catch((err) => console.log(err));

//middleware
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

//import routes
require("./routes/quoteRoute.js")(app);

const PORT = process.env.PORT || 5000;

// Accessing the path module
const path = require("path");

// Step 1:
app.use(express.static(path.resolve(__dirname, "./client/build")));
// Step 2:
app.get("*", function (request, response) {
  response.sendFile(path.resolve(__dirname, "./client/build", "index.html"));
});

app.listen(PORT, () => {
  console.log(`server running on port ${PORT}`);
});

## Modifying the package.json : 
  
Heroku will use the package.json to install all modules listed as dependencies. It is important to note that, when the NODE_ENV environment variable is set to production, npm will not install the modules listed in devDependencies.

Now, add the following lines in your package.json.
{
    ...
    "scripts": {
        ...
        "build": "cd client && npm run build",
        "install-client": "cd client && npm install",
        "heroku-postbuild": "npm run install-client && npm run build",
        "server": "nodemon server.js",
        "develop": "concurrently --kill-others-on-fail \"npm run server\" \"npm run start --prefix client\"",
        "start": "concurrently --kill-others-on-fail \"npm run server\" \"npm run start --prefix client\""
    },
    ...

}
"heroku-postbuild" will run immediately after Heroku has finished the deployment process.
NOTE: You may need to modify "server": "nodemon server.js", depending on where your sever.js is located and the name you have given. In this case, server.js is in the same level as package.json.

## Creating a Procfile
  
This will be the first file that Heroku will run. Create a file in the root of your project and name it Procfile. Inside, copy the following code:
web:npm start
  
## Deploying to Heroku
In this section, we will be working using the terminal. First, go to the root folder and create a new app.
  
$ cd your-project
$ heroku create app-name
  
Creating ⬢ app-name... done
https://app-name.herokuapp.com/ | https://git.heroku.com/app-name.git
Your application will be deployed in the URL displayed. You will need to push any new development with the following commands.
  
$ git add . 
$ git commit -am "commit message"
$ git push heroku main
  
Setting environment variables
Go to the Heroku dashboard online. You will find a list of all the applications you have built. Then, navigate to the settings tab on the top of the page. Scroll down to find the "config vars" section. Click "reveal config vars". You will need to make sure you have the following variables added:

Your mongo connection string. The key will be MONGODB_CONNECTION_STRING in my case, but it might change depending on how did you define this parameter. The value will be your connection string (excluding quotation marks). You can copy it from your .env file directly.
The Node Environment. The key will be NODE_ENV and the value will be production.
The PORT. The key will be PORT and the value, in my case, will be 5000.
Other useful commands
It is also possible to check the application locally before pushing to Heroku by running the following command.
  
$ heroku local
  
Another useful command which will allow you to gain insight into the behaviour of your application and debug any problems:
  
$ heroku logs --tail
  
And to open the application:
  
$ heroku open 
  
And now you have your application hosted and ready to show off!!
