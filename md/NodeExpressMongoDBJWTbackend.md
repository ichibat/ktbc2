# NodeExpressMongoDBJWTbackend

- Starting Express project (on terminal)
    mkdir backend
    cd backend2
    npm init -y
    npm i express mongodb helmet cors mongoose connect-flash passport express-session nodemon bcrypt jsonwebtoken
    touch index.js
- Initial index.js (on VS code)

— variables

    //initial requirement
    const express = require("express");
    const path = require("path");
    const cors = require("cors");
    const helmet = require("helmet");
    const mongoose = require("mongoose");
    const flash = require("connect-flash");
    const passport = require("passport");
    const session = require("express-session");
    const bcrypt = require('bcrypt');
    const jwt = require('jsonwebtoken');
    
    //mongoDB options
    const options = { useNewUrlParser: true, useUnifiedTopology: true }
    //port setting
    const port = process.env.PORT || 3000;
    
    // app for express
    const app = express();
    

— load routes

    const users = require("./routes/api/users");

—load Passport config

    //Passport config
    require("./config/passport")(passport);

— connect to mongoDB

    //connect to mongoDB
    mongoose.connect('mongodb://tims:Hit135Run@ds019946.mlab.com:19946/ktmethod', options);
    const db = mongoose.connection;
    db.on('error', console.error.bind(console, 'DB connection error:'));
    db.once('open', () => console.log('DB connection successful'));

— load models

    require("./models/Patient");
    const Patient = mongoose.model("patients");

— middlewares

    //Use following middlewares
    //Helmet
    app.use(helmet());
    //express.json()
    app.use(express.json())
    app.use(express.urlencoded({ extended: true }));
    //Express-session
    app.use(
      session({
        secret: "secret",
        resave: true,
        saveUninitialized: true
      })
    );
    //Passport
    app.use(passport.initialize());
    app.use(passport.session());
    //Flash
    app.use(flash());
    //Cor
    app.use(cors());

—Hello World! routing

    //Hello World!
    app.get('/', (req, res) => {
      res.send('Hello World!');
    });

— port listening

    // port listening
    app.listen(port, () => console.log(`OK, Server started on port ${port}`));

—creating .gitignore

    node_modules

— adding scripts in package.json

    // don't forget kanma before the script
    "start": "nodemon index.js"

— starting server and check database connection and “Hello world!’ on port 3000.

    npm start

— github upload after creating repository on github

    git init && git add . && git commit -m 'first commit'
    git remote add origin https://github.com/ichibat/backend2.git && git push -u origin master

—creating models directory in the root of server and User.js file.

    //User.js
    const mongoose = require('mongoose');
    const UserSchema = mongoose.Schema({
      name: {
        type: String,
        required: true
      },
      email: {
        type: String,
        required: true
      },
      password: {
        type: String,
        required: true
      },
      date: {
        type: Date,
        default: Date.now
      }
    });
    // User.js is loaded in index.js

— loading model(s) in index.js under database connection

    //loading models
    const User = require('./models/User'); 

— create routes

    mkdir routes && cd routes
    // for example,
    touch user.js

— create ./routes/user.js and include get request to show list of users

    // ./routes/user.js
    const express = require('express');
    const bcrypt = require('bcrypt');
    const router = express.Router();
    const User = require('../../models/User');
    
    
    //bcrypt setting
    const saltRounds = 10;
    
    // JWT setting
    const jwt = require('jsonwebtoken');
    
    // in order to receive json data from client
    router.use(express.json());
    
    //GET request for finding all users
    router.get('/', async (req, res) => {
        const users = await User.find({});
        res.json(users);
    });
    
    module.exports = router;

— include router setting in index.js

    // load router in line of other requires
    const userRouter = require('./routes/user');
    
    // use router under middlewares
    app.use('/user', userRouter);

— check /user route using Postman with GET request.  You will find the list of users.

    http://127.0.0.1:3000/user

— include POST request in ./routes/user.js

    // POST request in ./routes/user.js under router.get
    // This router might have problem in asynchronization
    router.post('/', (req,res)=>{
        const user = new User({
            name: req.body.name,
            email: req.body.email,
            password: req.body.password
        });
        bcrypt.hash(user.password, saltRounds, (err, hash) =>{
            if(err) throw err;
            user.password = hash;
            user.save();
            res.json(user);
        });
    });

— check /user route using Postman with POST request using Body/raw/JSON　and check response.  Also, check mLab.

    http://127.0.0.1:3000/user


    {
        "name": "石神井紗栄子",
        "email": "syakujii@example.com",
        "password": "password"
    }

— get individual data using _id of mongoDB.  Firstly, add GET request in user.js.

    // GET request for individual user under GET request for finding all users.  Note that ':' in ':userID' is removed.
    router.get('/:userID',(req, res)=>{
        User.findById(req.params.userID,(err,user)=>{
            if (err) console.log('error');
            res.send(user);
        });
    });

— check individual data using Postman with GET request using Body/raw/JSON　and check response.  

    http://127.0.0.1:3000/user/5d9ab55987ba7d1a1a0cc771

— Include DELETE request in user.js

    //DELETE request under POST request in user.js
    router.delete('/:userID', async (req, res) => {
      const user = await User.deleteOne({ _id: req.params.userID });
      res.send(user);
    });

— check DELETE request using Postman with Body/raw/JSON　and check response.  

    http://127.0.0.1:3000/user/5d9ab55987ba7d1a1a0cc771

— update data with PATCH request.  In this case find data with _id and update email.

    //PATCH request under DELET request in user.js
    router.patch('/:userID', async (req,res) => {
      console.log(req.body.age);
      const user = await User.updateOne({_id: req.params.userID}, {$set:{email:req.body.email}});
      res.send(user);
    });

— check PATCH request using Postman with Body/raw/JSON　and check response.  

    http://127.0.0.1:3000/user/5d9ab55987ba7d1a1a0cc771


    {
        "email": "pond@example.com"
    }

—create ./push.sh

    git add . && git commit -m 'improved' && git push

—push to github on terminal

    chmod +x ./push.sh
    ./push.sh

—create login router in user.js

    //login authentication with email and password
      router.post('/login',(req,res) => {
        User.findOne({ email : req.body.email }, (err, user) => {
          if (err) {
            return res.status(400).json({"error":err.message});
          }
          if(!user){
            return res.json({"message": "email not found"})
          }
          bcrypt.compare(req.body.password, user.password, (err,result) => {
            if (err) {
              return res.status(400).json({"error":err.message});
            }
            if (!result) {
              return res.json({"message" : "password is not correct"})
            }
            return res.json({"message" : "password is correct"})
          })
        })
      })

— check login using POST request with postman 

    // http://127.0.0.1:3000/user POST request raw JSON
    
    {
        "name": "富山史子",
        "email": "fumiko@example.com",
        "password": "password"
    }

— create token of JWT. remove return res.json({"message" : "password is correct"}) and paste following!

    // remove return res.json({"message" : "password is correct"}) and paste following
    
    const payload = {
    id: user.id,
    name: user.name,
    email: user.email
    }
    
    const token = jwt.sign(payload,'secret')
    return res.json({token})

— Token check using postman

    // POST to http://127.0.0.1:3000/user/login
    
    {
            "email": "hiromasa@example.com",
            "password": "password"
    }

— Check Token contents on https://jwt.io/#debugger

— Create routing for /api/auth/user

    // Under JWT router
    router.get('/api/auth/user/',(req,res) => {
    
    const bearToken = req.headers['authorization']
    const bearer = bearToken.split(' ')
    const token = bearer[1]
    
    jwt.verify(token,'secret',(err,user)=>{
    if(err){
    return res.sendStatus(403)
    }else{
    return res.json({
    user
    });
    }
    })
    });

— JWT test not working

