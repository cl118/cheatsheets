# MERN Step by Step Example
This step by step cheat sheet translates Henry Web Dev's MERN stack tutorial on YouTube from Vietnamese to English, updates some out of date syntax, and sums up some of the extra notable information. Feel free to take this and add/remove what works for you. This was incredibly helpful for me while learning the MERN stack so I want to pass it along. 

This guide assumes the reader knows basics and tasks needed to be completed outside of coding will not be covered. These tasks will be numbered to differentiate them from the explanation of tasks to be coded. 

## Server

### Setting up server
Create server folder in project folder cd into it
Initialize to defaults

```
mkdir server
cd server
npm init -y
```

### Install dependencies
```
npm i express jsonwebtoken mongoose dotenv argon2 cors
npm i -D nodemon
```
- Express = Node.js framework
- json web token = Authorization
- Mongoose = Object Relational Model (ORM) aka Models/Schemas
- Dotenv loads environment variables from a .env file into process.env
- Argon2 = password hasher
- cors = Cross-Origin Resource Sharing (CORS). Allows server connections from different origin/hosting
- Nodemon = monitors server and restarts on save

Go to package.json and add `"server": "nodemon index"` to `"scripts"`

### Server boilerplate
Create `index.js` in `server`

/server/index.js:
```
const express = require(‘express’)
const app = express()

app.get(‘/‘, (req, res) => res.send(‘Hello world’)

const PORT = 5000

app.listen(PORT, () => console.log(`Server started on port ${PORT}`))
```
Type `npm run server` in terminal to start up server.

### Connect to MongoDB

1. Set up MongoDB Atlas Cluster
2. Set MongoDB connections and admin

/server/index.js:
```
const express = require (‘express’)
const app = express()

/* New code below this line */
const mongoose = require('mongoose')

const connectDB = async() => {
  try {
    await mongoose.connect('<MONGODB SRV URI>')
    
    console.log('MongoDB Connected')
  } catch (error) {
    console.log(error.message)
    process.exit(1)
  }
}

connectDB()

/* New code above this line */

app.get(‘/‘, (req, res) => res.send(‘Hello world’)
```


### Create .env file
Create `.env` file in `server`
.env:
```
DB_USERNAME=<MONGODB_DATABASE_USERNAME>
DB_PASSWORD=<MONGODB_DATABASE_PASSWORD>
```
Now that we've tested the connection to MongoDB, we'll move the username and password into a .env file to protect our information.
Add `require('dotenv').config()` to top of `/server/index.js` and replace the username and password in the MongoDB URI with the variables created using object literals.

Example:
```
`mongodb+srv://${process.env.DB_USERNAME}:${process.env.DB_PASSWORD}@cluster0.ksh2g.mongodb.net/<COLLECTION_NAME>?retryWrites=true&w=majority`
```

### Create models/schemas

Create `models` folder in `server`
Create `User.js` in `models` folder

/server/models/User.js:
```
const mongoose = require('mongoose')
const Schema = mongoose.Schema

const UserSchema = new Schema({
  username: {
    type: String,
    require: true,
    unique: true
  },
  password: {
    type: String,
    require: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
})

module.exports = mongoose.model(‘User’, UserSchema)
```
> When you call mongoose.model() on a schema, Mongoose compiles a model for you.

> The first argument is the singular name of the collection your model is for. Mongoose automatically looks for the plural, lowercased version of your model name. Thus, for the example above, the model User is for the users collection in the database.

> Note: The .model() function makes a copy of schema. Make sure that you've added everything you want to schema, including hooks, before calling .model()!

*https://mongoosejs.com/docs/models.html*

Create `Post.js` in `models` folder
/server/models/Post.js:
```
const mongoose = require('mongoose')
const Schema = mongoose.Schema

const PostSchema = new Schema ({
  title: {
    type: String,
    required: true
  },
  description: {
    type: String
  },
  url: {
    type: String
  },
  status: {
    type: String,
    enum: ['TO LEARN', 'LEARNING', 'LEARNED']
  },
  user: {
    type: Schema.Types.ObjectId,
    ref: 'User'
  }
})

module.exports = mongoose.model('Post', PostSchema)
```

### Set up authRoutes boilerplate
Create `routes` folder in `server`
Create `auth.js` in `routes`

/server/routes/auth.js:
```
const express = require('express')
const router = express.Router()

const User = require('../models/User')

router.get('/', (req, res) => res.send('USER ROUTE'))

module.exports = router
```

### Add router to server/index.js
Add `const authRouter = require('./routes/auth')` to `server/index.js`

Remove `app.get(‘/‘, (req, res) => res.send(‘Hello world’)` now that we know it works.

Replace with: 
```
app.use(express.json())
app.use('/api/auth', authRouter)
```
This will allow the server to read json format and route all authentication related routes to the authRouter.


### Check API route with REST client extension

Create `request.http` in `server` and use REST client extension in VS Code to check HTTP requests

/server/request.http:
```
GET http://localhost:5000/api/auth
```
Click "Send Request" and you should receive a response of "USER ROUTE" as was defined in `authRoutes`


## Set up authentication routes
Add to /server/routes/auth.js:
```
const argon2 = require('argon2');
const jwt = require('jsonwebtoken');
```

Add to /server/.env:
```
ACCESS_TOKEN_SECRET=<You can put a bunch of gibberish here as a token secret i.e. 'alkshgglkasjhaakjgh9872ak'>
```

### Create registration route
Add to /server/routes/auth.js:
```
// @route POST api/auth/register
// @desc Register user
// @access Public
router.post('/register', async (req, res) => {
  const { username, password } = req.body;

  // Simple Validation
  if (!username || !password)
    return res
      .status(400)
      .json({ success: false, message: 'Missing username and/or password' });

  try {
    // Check for existing user
    const user = await User.findOne({ username });
    if (user)
      return res
        .status(400)
        .json({ success: false, message: 'Username already taken' });

    // All good
    const hashedPassword = await argon2.hash(password);
    const newUser = new User({ username, password: hashedPassword });
    await newUser.save();

    // Return token
    const accessToken = jwt.sign(
      { userId: newUser._id },
      process.env.ACCESS_TOKEN_SECRET
    );

    res.json({
      success: true,
      message: 'User created successfully',
      accessToken,
    });
  } catch (error) {
    console.log(error);
    res.status(500).json({ success: false, message: 'Internal Server Error' });
  }
});
```

### Check register route with `request.http`
Delete previous content in `request.http`.

request.http:
```
POST http://localhost:5000/api/auth/register
Content-Type: application/json

{
    "username": "user",
    "password": "password"
}
```
Check response for success confirmation and check MongoDB for created user(s)


### Create login route
Add to /server/routes/auth.js:
```
// @route POST api/auth/login
// @desc Login user
// @access Public
router.post('/login', async (req, res) => {
  const { username, password } = req.body;

  // Simple Validation
  if (!username || !password)
    return res
      .status(400)
      .json({ success: false, message: 'Missing username and/or password' });

  try {
    // Check for existing user
    const user = await User.findOne({ username });
    if (!user)
      return res
        .status(400)
        .json({ success: false, message: 'Incorrect username or password' });

    // Username found
    const passwordValid = await argon2.verify(user.password, password);
    if (!passwordValid)
      return res
        .status(400)
        .json({ success: false, message: 'Incorrect username or password' });

    // All good
    // Return token
    const accessToken = jwt.sign(
      { userId: user._id },
      process.env.ACCESS_TOKEN_SECRET
    );

    res.json({
      success: true,
      message: 'User logged in successfully',
      accessToken,
    });
  } catch (error) {
    console.log(error);
    res.status(500).json({ success: false, message: 'Internal Server Error' });
  }
});
```
While in development, you can specifically put "incorrect username" or "incorrect username" for testing purposes, but remember to change it to "incorrect username or password" for both responses in production for increased security purposes.

### Check login route with request.http
Add `###` below previous post request to separate requests
Add to request.http:
```
POST http://localhost:5000/api/auth/login
Content-Type: application/json

{
    "username": "user",
    "password": "password"
}
```
Try changing username/password to incorrect values to check responses as well.


## Set up post routes
Create `post.js` in `/server/routes/`.

/server/routes/post.js:

CONTINUE 56 MINS
