# express-mongodb-auth
## Overview
The express-mongodb-auth project provides a foundational setup for building an authentication system using Express.js and MongoDB. Essential concepts like defining schemas and models, implementing secure password hashing using bcrypt, and creating API routes for user registration are included. 
## Setup
### Project setup:
- In your terminal make sure you `cd` into the directory that you want your project to go.
    - Make a new directory for your project:
        ```bash
        mkdir express-mongodb-auth
    - Go into that directory:
        ```bash
        cd express-mongodb-auth
    - Initialize `Node.js`:
        ```bash
        npm init -y
    - Install dependencies for the project:
        ```bash
        npm install express mongoose dotenv 
    - Open your new project in VSCode:
        ```bash
        code .v
### Create the Server:
- Create a file named `index.js`.
    - Copy this server code into `index.js`:
        ```js
        // index.js
        const express = require('express');
        const mongoose = require('mongoose');
        const app = express();
        const port = 3000;

        // Middleware to parse JSON bodies
        app.use(express.json());

        // Connect to MongoDB
        mongoose
        .connect('your-mongodb-connection-string-here', {
            useNewUrlParser: true,
            useUnifiedTopology: true,
        })
        .then(() => {
            console.log('Connected to MongoDB');
        })
        .catch((error) => {
            console.error('Error connecting to MongoDB:', error);
        });

        app.get('/', (req, res) => {
            res.send('Hello, World!');
        });

        app.listen(port, () => {
            console.log(`Server is running at http://localhost:${port}`);
        });


- In order to keep our `connection string` secure, we will need to store it in a `.env` file. 
    - Create a file named `.env`.
    - Define a variable for your `connection string`. NOTE: Your variable should be in all caps and no spaces between the variable and `=` or after.
        Example: `ATLAS_URL=mongodb+srv://<username>:<password>@cluster0.x3zgp.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0`
    - In order for us to use the environment variable in our server, we will need to import an configure dotenv:
        ```js
        require('dotenv').config()
    - We can add a console.log to confirm it is working, but remove after confirming:
        ```js
        console.log(process.env)
    - To use the environment variable in the server code, you can store it in a new variable to use it where you originally put your `connection string`:
        ```js
        const atlasUrl = process.env.ATLAS_URL
    - Replace your `'your-mongodb-connection-string-here'` with the new variable. 
- Test the Connection:
    - Start the server by running `node index.js`
    - In the terminal it should log `Connected to MongoDB`.
    - Navigate to `localhost/3000` in your browser to confirm that the page displays `Hello World!`. 
## Schema and Model
### Schema:
- A schema is a structure that defines the shape of the documents within a MongoDB collection. It acts as a blueprint for the data by specifying fields, their data types, and any constraints or validations.
- **Example Schema:**
    ```js
    //Define the user schema
    const userSchema = mongoose.Schema({
        name: { type: String, required: true },
        email: { type: String, required: true, unique: true },
        password: { type: String, required: true }
    });
### Model:
- A model is a wrapper around a schema and provides an interface to interact with the database collection. With a model, you can create, read, update, and delete (CRUD) documents in the corresponding collection.
- **Example Model:**
    ```js
    //Create the User model
    const User = mongoose.model('User', userSchema);
### Define the Schema and Model
- Create a directory named `models`.
- Inside that directory, create a file named `User.js`.
- In `User.js` define a schema with the following fields: `name, email, password`.
- After the schema, create the model and export the user module:
    ```js
    //Create the User model
    const User = mongoose.model('User', userSchema);

    //Export the model
    module.exports = User;
### Password Hashing
Implementing password hashing using `bcrypt` before saving a user's password involves securely converting the user's plain-text password into a hashed format. 
- `bcrypt`: is a library designed for hashing passwords securely. It makes brute-force attacks(trying every possible password) more time-consuming. 
- `Salt`: Bcrypt automatically generates a salt (random data added to the password) and incorporates it into the hash. This makes each hash unique, even if two users have the same password.
- `Hashing Workflow`:
    * The `plain-text` password is combined with a `salt`.
    * The `bcrypt` algorithm generates a hash based on the password and salt. 
    * Only the `hash`(not the original password) is stored in the database. 
- Example Password Hashing using `bcrypt`:
    ```js
    //Hash bcyrpt
    userSchema.pre('save', async function (next) {
        const salt = await bcrypt.genSalt(10);
        const plainTextPassword = this.password;
        const encryptedPassword = await bcrypt.hash(plainTextPassword, salt);
        this.password = encryptedPassword;
        next();
    });
- In order to use `bcrypt` we need to install it:
    ```bash
    npm install bcryptjs
- Import it in `User.js` where our schema is located.
    ```js
    const bcrypt = require("bcryptjs");
- Add the `password hashing` code under the schema code.
### Register Route
- Update `index.js` to include a registration route:
    ```js
    //Register route
    app.post('/register', async (req, res) => {
        // Extract the request body containing the user data
        const data = req.body;
        // Create a new User instance with the provided data
        const user = new User({
            name: data.name,
            email: data.email,
            password: data.password
        });

        try {
            // Save the user data to the database
            const savedUser = await user.save();
            console.log(savedUser);
            // Send the saved user data as a JSON response
            res.json(savedUser);
        } catch (e) {
            console.error(e);
            res.status(500).json({ error: 'Failed to save user' });
        }
    })
## Testing
- Start Your Server:
- If `nodemon` is installed:
    ```bash
    npm run dev
- If not:
    ```bash
    node index.js
- Test with Postman:
- Register a User:
    - Method: POST
    - Endpoint: http://localhost:3000/register
    - Response:
        ```json
        {
            "name": "Jennifer",
            "email": "jennifettarleton@gmail.com",
            "password": "$2a$10$Zb3BwzjesDmL9HZrZNKW6eQ5.8xZEH49xfcDtqFpYloWnBAf6DRee",
            "_id": "675aff39e60f0acb0444c417",
            "__v": 0
        }
    - Screenshot:
    ![Register User](<registerUserPostman.png>)
- Make sure your new user is in MongoDB database with the hash password:
![Mongo](<mongoScreenshot.png>)

## Initializing a Github Repository
- In your bash terminal, add a `.gitignore`:
    ```bash
    touch .gitignore
- Include `node_modules` and `.env`:
    ```bash
    echo "node_modules/" >> .gitignore
    echo ".env" >> .gitignore
- Create a new repository on Github, without a README.md or .gitignore.
- Back in bash initialize a empty repo:
    ```bash
    git init
- Add files to be staged for commit:
    ```bash
    git add .
- Initial commit:
    ```bash
    git commit -m "initial commit"
- Add a main branch:
    ```bash
    git branch -M main
- Add your new repository:
    ```bash
    git remote add origin https://github.com/username/reponame.git
- Push to Github
    ```bash
    git push -u origin main

## Conclusion
This project demonstrates how to build a robust authentication system with Express.js and MongoDB. With this setup, you're ready to expand the project by adding features such as login functionality, token-based authentication, or user management. It's a great starting point for building secure web applications.
## Acknowledgments

- `bcrypt` - <https://en.wikipedia.org/wiki/Bcrypt?form=MG0AV3>
- NPM bcryptjs - <https://www.npmjs.com/package/bcryptjs>