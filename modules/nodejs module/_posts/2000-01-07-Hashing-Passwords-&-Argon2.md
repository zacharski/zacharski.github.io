## Hashing Passwords with Argon2


### Introduction to Hashing Passwords

<iframe width="560" height="315" src="https://www.youtube.com/embed/4Eg7cEdV0AU" frameborder="0" allowfullscreen></iframe>





## Using Argon2 with Node.js


<iframe width="560" height="315" src="https://www.youtube.com/embed/fEV3D0ZuBgM" frameborder="0" allowfullscreen></iframe>

### Basic Instructions.

First, on the command line type:

```
​	npm install argon2
```

You are set!   If you have problems installing Argon2, see the troubleshouting suggestions at the bottom of this page.


Here is the most basic demo:

    const argon2 = require("argon2");

    async function hashit() {
      try {
        const hash = await argon2.hash("password");
        console.log(hash);
       } catch (err) {
        console.log("ERROR " + err);
       }
    }

    hashit();

You see that each time we run this and hash the same password string password, it returns a different hash. That is because it uses a salt that is appended to the returned hash.


Now let's add code to check whether a password matches the hash using `argon2.verify`

      const argon2 = require("argon2");

      async function hashit() {
        let hash;
        try {
          hash = await argon2.hash("password");
          console.log(hash);
        } catch (err) {
          console.log("ERROR " + err);
        }
        try {
          if (await argon2.verify(hash, "password")) {
            // password match
            console.log("matched");
          } else {
            // password did not match
            console.log("not matched");
          }
        } catch (err) {
          // internal failure
          console.log("ERROR", err);
        }
      }

      hashit();

 
 ### Argon2, Postgres and Post requests   

Now let's look at using this in a database app.

So our api will be

>    createUser username password

>    login username password



    const express = require("express");

    const app = express();
    const bodyParser = require("body-parser");
    app.use(bodyParser.json({ type: "application/json" }));
    app.use(bodyParser.urlencoded({ extended: true }));

    app.set("port", 8080);
    const argon2 = require("argon2");
    const Pool = require("pg").Pool;
    const config = {
      host: "localhost",
      user: "parky",
      password: "mItithaWFAnC",
      database: "parky"
    };

    const pool = new Pool(config);

    app.get("/hello", (req, res) => {
      res.json("Hello World!");
    });

    app.post("/login", async (req, res) => {
      console.log(req.body);
      const username = req.body.username;
      const password = req.body.password;
      try {
        const query = "SELECT password from users where username = $1";
        const result = await pool.query(query, [username]);
        if (result.rowCount == 1) {
          console.log(result.rows[0].password);
          if (await argon2.verify(result.rows[0].password, password)) {
            res.json("Log In successful");
          } else {
            res.json("Password incorrect");
          }
        } else {
          res.json("username not found");
        }
      } catch (err) {
        console.log("ERROR " + err);
      }
    });

    app.post("/create", async (req, res) => {
      let hash;
      const username = req.body.username;
      const password = req.body.password;
      try {
        hash = await argon2.hash(password, "abcdefghijklmnop");
        console.log("HASH " + hash);
        const query = "INSERT INTO users (username, password) VALUES ($1, $2)";
        const result = await pool.query(query, [username, hash]);
        //console.log(result);
        if (result.rowCount == 1) {
          res.json("User created");
        } else {
          res.json("User not created");
        }
      } catch (err) {
        console.log("ERROR " + err);
        if (err.message.search("duplicate") != -1) {
          res.json("Username taken");
        }
      }
    });

    app.listen(app.get("port"), () => {
      console.log(`Find the server at: http://localhost:${app.get("port")}/`);
    });



### If you have problems installing Argon2  

###### (pre `npm install argon2`)

#### 1. you need `gcc` on your machine

##### Linux
If you are using Linux chances are you probably already have it installed.  In a terminal execute

	gcc -v
	
If you get a version higher than 5.0 you are good to go. Otherwise 

	sudo apt-get install gcc
	
##### Mac OSX

On a Mac install the Xcode command line tools (preferred) or install via brew

```
   brew install gcc
```

#### 2. install node-gyp globally
Node-gyp allows you to compile node.js modules.

```
	$ npm install -g node-gyp
```

If you get a permissions error you need to do the following.


#### Reinstalling node and npm


##### 1. Delete the currently installed versions of node and npm

	sudo apt-get remove nodejs
	sudo apt-get remove npm

##### 2. Install nvm

	curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
	bash

##### 3. Reinstall npm and nodejs

Go to [nodejs.org](https://nodejs.org/en/) to see what the current version is. Use that version number in the command:

	nvm install <version>

For example, suppose the current version is 10.16.3. Then execute 

	nvm install 10.16.3

That should do it. Now go and install node-gyp as described above.