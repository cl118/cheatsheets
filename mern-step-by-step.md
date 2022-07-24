# MERN Setup Cheatsheet

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
npm i --save-dev nodemon
```
- Express = Node.js framework
- json web token = Authorization
- Mongoose = Object Relational Model (ORM) aka Models/Schemas
- Dotenv loads environment variables from a .env file into process.env
- Argon2 = password hasher
- cors = Cross-Origin Resource Sharing (CORS). Allows server connections from different origin/hosting
- Nodemon = monitors server and restarts on save

Go to package.json and add `"server": "nodemon"` to `"scripts"`

### Server boilerplate
Create `index.js` in `server`

/server/index.js:
```
const express = require (‘express’)
const app = express()

app.get(‘/‘, (req, res) => res.send(‘Hello world’)

const PORT = 5000

app.listen(PORT, () => console.log(`Server started on port ${PORT}`))

npm run server
```

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

### Create models/schemas

1. Create `models` folder in `server`
2. Create `User.js` in `models` folder

/server/models/User.js:
```
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
})

module.exports = mongoose.model('Post', PostSchema)
```

### Set up router/routes
Create `routes` folder in `server`
Create `auth.js` in `routes`

/server/routes/auth.js:
```
const express = require('express')
const router = express.Router()

Create `request.http` and use REST client extension in VS Code to check HTTP requests
