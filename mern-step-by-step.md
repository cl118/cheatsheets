# MERN Setup Cheatsheet

## Server

### Setting up server
Create server folder in project folder cd into it

```
mkdir server
cd server
npm init
```

Say yes to all options

### Install dependencies
```
npm i express jsonwebtoken mongoose dotenv argon2 cors
npm i --save-dev nodemon
```

Go to package.json and add `"server": "nodemon"` to `"scripts"`

### Server boilerplate
Create `index.js` in `server`

index.js:
```
const express = require (‘express’)
const app = express()

app.get(‘/‘, (req, res) => res.send(‘Hello world’)

const PORT = 5000

app.listen(PORT, () => console.log(`Server started on port ${PORT}`))

npm run server
```

### Connect to MongoDB

Set up MongoDB Atlas Cluster
Set MongoDB connections and admin

index.js:
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

Create `models` folder in `server`
Create `User.js` in `models` folder

User.js:
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

module.exports = mongoose.model(‘users’, UserSchema)
```



Create `request.http` and use REST client extension in VS Code to check HTTP requests
