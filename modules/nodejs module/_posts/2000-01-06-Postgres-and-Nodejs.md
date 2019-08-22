## Using Postgres with Node.js

<iframe width="560" height="315" src="https://www.youtube.com/embed/wNCt8Q6hCFc" frameborder="0" allowfullscreen></iframe>


#### Files used in the video

* [campgrounds.sql](https://bit.ly/2KTgqaR)
* [NationalParks.postman_collection.json](https://raw.githubusercontent.com/zacharski/devCourse/master/postman/NationalParks.postman_collection.json)



## SQL Injection


<iframe width="560" height="315" src="https://www.youtube.com/embed/Ty7RIBrqalc" frameborder="0" allowfullscreen></iframe>


## Notes for the Postgres Node video

### 1. The basic template and introducing the connection pool.

	const express = require("express");
	const bodyParser = require("body-parser");
	const app = express();

	app.set("port", 3000);
	app.use(bodyParser.json({ type: "application/json" }));
	app.use(bodyParser.urlencoded({ extended: true }));

	const Pool = require("pg").Pool;
	const config = {
		host: "localhost",
		user: "parky",
		password: "mItithaWFAnC",
		database: "parky"
	};

	const pool = new Pool(config);

	app.get("/hello", (req, res) => {
		res.json("Hello World");
	});

	app.listen(app.get("port"), () => {
		console.log(`Find the server at http://localhost:${app.get("port")}`);
	});



### 2. Handling the /info get request

	app.get("/info", async (req, res) => {
		// req.query.q
		// ? SELECT * FROM campgrounds WHERE name = ...
		try {
			const template = "SELECT * FROM campgrounds WHERE name = $1";
			const response = await pool.query(template, [req.query.q]);
			if (response.rowCount == 0) {
				res.json({ status: "not found", searchTerm: req.query.q });
			} else {
				res.json({ status: "ok", results: response.rows[0] });
			}
		} catch (err) {
			console.error("Error running query " + err);
			res.json({ status: "error" });
		}
	});


### 3. Handling the /near get request

	app.get("/near", async (req, res) => {
		try {
			const template = "SELECT name FROM campgrounds WHERE closest_town = $1";
			const response = await pool.query(template, [req.query.city]);
			if (response.rowCount == 0) {
				res.json({ status: "not found", searchTerm: req.query.city });
			} else {
				const camps = response.rows.map(function(item) {
					return item.name;
				});
				res.json({
					status: "ok",
					results: { city: req.query.city, campgrounds: camps }
				});
			}
		} catch (err) {
			res.json({ status: "error" });
		}
	});



### 4. Handling the /add post request

	app.post("/add", async (req, res) => {
		const name = req.body.name;
		const town = req.body.city;
		const desc = req.body.description;
		const toilets = req.body.toilets;

		try {
			const template =
				"INSERT INTO campgrounds (name, closest_town, description, restrooms) VALUES ($1, $2, $3, $4)";
			const response = await pool.query(template, [
				name,
				town,
				desc,
				toilets
			]);
			res.json({ status: "ok", results: { city: town, campground: name } });
		} catch (err) {
			res.json({ status: "not added: duplicate entry", campground: name });
		}
	});
