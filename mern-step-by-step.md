# MERN Step by Step Example
This walkthrough summarizes and translates Henry Web Dev's MERN stack tutorial on YouTube from Vietnamese to English and updates out of date syntax. Feel free to take this and add/remove what works for you. This was incredibly helpful for me while learning the MERN stack so I want to pass it along. This walkthrough is not as comprehensive as the video and some shortcuts were taken. If you'd like to see him code it live, you can [watch it](https://www.youtube.com/watch?v=rgFd17fyM4A) and reference the code here. This can be especially helpful when encountering errors with old syntax.

*This guide assumes the reader knows the fundamentals and tasks needed to be completed outside of coding will not be covered. These tasks will be numbered to differentiate them from the explanation of tasks to be coded.*

## Server

### Setting up server
Create server folder in project folder cd into it
Initialize to defaults

```javascript
mkdir server
cd server
npm init -y
```

### Install dependencies
```javascript
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
```javascript
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
```javascript
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
```javascript
DB_USERNAME=<MONGODB_DATABASE_USERNAME>
DB_PASSWORD=<MONGODB_DATABASE_PASSWORD>
```
Now that we've tested the connection to MongoDB, we'll move the username and password into a .env file to protect our information.
Add `require('dotenv').config()` to top of `/server/index.js` and replace the username and password in the MongoDB URI with the variables created using object literals.

Example:
```javascript
`mongodb+srv://${process.env.DB_USERNAME}:${process.env.DB_PASSWORD}@cluster0.ksh2g.mongodb.net/<COLLECTION_NAME>?retryWrites=true&w=majority`
```

### Create models/schemas

Create `models` folder in `server`
Create `User.js` in `models` folder

/server/models/User.js:
```javascript
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
```javascript
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
```javascript
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
```javascript
app.use(express.json())
app.use('/api/auth', authRouter)
```
This will allow the server to read json format and route all authentication related routes to the authRouter.


### Check API route with REST client extension

Create `request.http` in `server` and use REST client extension in VS Code to check HTTP requests

/server/request.http:
```javascript
GET http://localhost:5000/api/auth
```
Click "Send Request" and you should receive a response of "USER ROUTE" as was defined in `authRoutes`


## Set up authentication routes
Add to /server/routes/auth.js:
```javascript
const argon2 = require('argon2');
const jwt = require('jsonwebtoken');
```

Add to /server/.env:
```javascript
ACCESS_TOKEN_SECRET=<You can put a bunch of gibberish here as a token secret i.e. 'alkshgglkasjhaakjgh9872ak'>
```

### Create registration route
Add to /server/routes/auth.js:
```javascript
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
```javascript
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
```javascript
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
```javascript
POST http://localhost:5000/api/auth/login
Content-Type: application/json

{
    "username": "user",
    "password": "password"
}
```
Try changing username/password to incorrect values to check responses as well.


## Set up authentication middleware
Create folder `middleware` in `/server`
Create `auth.js` file in `/server/middleware`

/server/middleware/auth.js:
```javascript
const jwt = require('jsonwebtoken');

const verifyToken = (req, res, next) => {
  const authHeader = req.header('Authorization');
  const token = authHeader && authHeader.split(' ')[1];

  if (!token)
    return res
      .status(401)
      .json({ success: false, message: 'Access token not found' });

  try {
    const decoded = jwt.verify(token, process.env.ACCESS_TOKEN_SECRET);

    req.userId = decoded.userId;
    next();
  } catch (error) {
    console.log(error);
    return res.status(403).json({ success: false, message: 'Invalid token' });
  }
};

module.exports = verifyToken;
```
This will ensure user has a valid access token and provide user id information for posts


## Set up post routes
Create `post.js` in `/server/routes/`.

### Create POST request
The following code will declare the necessary modules and code a POST request to create new posts

/server/routes/post.js:
```javascript
const express = require('express');
const router = express.Router();
const verifyToken = require('../middleware/auth');

const Post = require('../models/Post');

// @route POST api/posts
// @desc Create post
// @access Private
router.post('/', verifyToken, async (req, res) => {
  const { title, description, url, status } = req.body;

  // Simple validation
  if (!title)
    return res
      .status(400)
      .json({ success: false, message: 'Title is required' });

  try {
    const newPost = new Post({
      title,
      description,
      url: url.startsWith('https://') ? url : `https://${url}`,
      status: status || 'TO LEARN',
      user: req.userId,
    });

    await newPost.save();

    res.json({
      success: true,
      message: 'Post created successfully',
      post: newPost,
    });
  } catch (error) {
    console.log(error);
    res.status(500).json({ success: false, message: 'Internal Server Error' });
  }
});
```

### Check POST request with request.http
Add `###` again below previous request to separate requests

Add following code to request.http:
```javascript
POST http://localhost:5000/api/posts
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiI2MmQ0ZWM3MGJlNTQ5YTJmMjdjNjhmNTEiLCJpYXQiOjE2NTgxMjE0MTR9.WRZpibPt1tmCN_wJyqKEXnKW-2boYOg1s-V8sutiCWc


{
    "title": "React",
    "description": "React",
    "url": "react.com",
    "status": "LEARNING"
}
```
In the above code example, the long string after `Authorization: Bearer` is the access token assigned to a specific user. To submit a valid POST, you'll need to copy the "accessToken" from the login request response and paste it into the post request after `Authorization Bearer`. You can purposefully submit an invalid token to see the response of "Invalid token" to check.

Try creating several posts with different information and verify in your MongoDB for new posts in your appropriate collection.


### Create GET posts request
Next, we will code a GET request to get all posts by the current logged in user. Adding `.populate('user', ['username',])` will add the username to the response so you don't need to keep referencing the user ID in MongoDB.

Add to /server/routes/post.js:
```javascript
// @route GET api/posts
// @desc Get posts
// @access Private
router.get('/', verifyToken, async (req, res) => {
  try {
    const posts = await Post.find({ user: req.userId }).populate('user', [
      'username',
    ]);
    res.json({ success: true, posts });
  } catch (error) {
    console.log(error);
    res.status(500).json({ success: false, message: 'Internal Server Error' });
  }
});
```

### Check GET request with request.http
Once again add `###` below previous requests to separate requests

Add following code to request.http with valid access token:
```javascript
GET http://localhost:5000/api/posts
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiI2MmQ0ZTEyNWZmYjE0YWZjNmQxZDAyMWIiLCJpYXQiOjE2NTgxMjg0OTF9.EMb0sBYPggXaa3wosNtWqeOxM_CR6Xj4RGIU1YomUKg
```
The response should be an array of all posts created by the current user


### Create PUT request
After GET and POST, we will make a PUT request to enable us to update necessary information. This is similar to the POST request earlier, but we need to add `:id` of post we are updating. Because we are updating the information, some of the fields are not required, so we will need to add `|| ''` to the description and URL fields in case they are left blank. The status will default to 'TO LEARN' and the title is required so it does not need this.

Add to /server/routes/post.js:
```javascript
// @route PUT api/posts
// @desc Update post
// @access Private
router.put('/:id', verifyToken, async (req, res) => {
  const { title, description, url, status } = req.body;

  // Simple validation
  if (!title)
    return res
      .status(400)
      .json({ success: false, message: 'Title is required' });

  try {
    let updatedPost = {
      title,
      description: description || '',
      url: (url.startsWith('https://') ? url : `https://${url}`) || '',
      status: status || 'TO LEARN',
    };

    const postUpdateCondition = { _id: req.params.id, user: req.userId };

    updatedPost = await Post.findOneAndUpdate(
      postUpdateCondition,
      updatedPost,
      { new: true }
    );
    
    // User not authorized to update post or post not found
    if (!updatedPost)
      return res.status(401).json({
        success: false,
        message: 'Post not found or user not authorized',
      });

    res.json({ success: true, message: 'Post updated', post: updatedPost });
  } catch (error) {
    console.log(error);
    res.status(500).json({ success: false, message: 'Internal Server Error' });
  }
});
```

### Check PUT request with request.http
Once again we'll add `###` below the previous posts to start a new request.

Add following code to request.http:
```javascript
PUT http://localhost:5000/api/posts/<POST_ID>
Content-Type: application/json
Authorization: Bearer <accessToken>

{
    "title": "Next.js",
    "description": "Next.js is AMAZING!",
    "url": "nextjs.org",
    "status": "LEARNING"
}
```
In the code above, copy the post ID of a post you want to update from your GET request and paste it into the PUT url. Update the information and see if it works! If it's successful, check MongoDB for updated information.


### Create DELETE request
Last but not least, we will need a way to delete our posts.

Add to /server/routes/post.js:
```javascript
// @route DELETE api/posts
// @desc Delete post
// @access Private
router.delete('/:id', verifyToken, async (req, res) => {
  try {
    const postDeleteCondition = { _id: req.params.id, user: req.userId };
    const deletedPost = await Post.findOneAndDelete(postDeleteCondition);

    // User not authorized or post not found
    if (!deletedPost)
      return res.status(401).json({
        success: false,
        message: 'Post not found or user not authorized',
      });

    res.json({
      success: true,
      message: 'Post deleted successfully',
      post: deletedPost,
    });
  } catch (error) {
    console.log(error);
    res.status(500).json({ success: false, message: 'Internal Server Error' });
  }
});
```

### Check DELETE request with request.http
You know the drill... Add `###` to `request.http` to start a new request.

request.http:
```javascript
DELETE http://localhost:5000/api/posts/<POST_ID>
Authorization: Bearer <accessToken>
```
Just like the PUT request, you will need a post ID that you want to delete. Paste it into the <POST_ID> and send your request. The response should be the post you deleted and you can confirm by doing a GET request of posts and see that it is no longer there.
