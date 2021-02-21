---
layout: post
title: "Silver Surfer Project Initialization | Ben Schroth"
date: 2021-02-15 8:05:00 -0500
categories: jekyll update
---

# Silver Surfer Project Setup

**_This blog serves as a description of setting up the Silver Surfer App. Accompanying detailed documentation of the code referenced in this blog can be found in the [Silver Surfer Repo](https://github.com/BennyBlockchain/silver-surfer)_**.

#### Introduction

This blog serves as the documentation of setting up the Silver Surfer Project. It is also written for the Silver Fox team to review the project architecture and initial dependencies.

Initalize a Next.js app with `npx create-next-app silver-surfer`

## Next.js

Silver Surfer is a full stack web application built using the Next.js React framework. Components and pages are built using the React.js. Routing and API's are built using the Next.js.

Example Next.js folder structure:

```
app/ 		# Next.js app
|_public/ 	# Images and other static files
|_pages/ 	# Each js file in this folder is mapped to route (ex. pages/about.js = /page)
	|_api/ 	# API endpoints are defined here
|_components/ 	# React coomponents and layouts
```

## Bootstrap

For styling, layouts, and components Silver Fox decided to use the Bootstrap framework. Because we are using React, I installed the react-bootstrap library which makes the experience of working with Bootstrap better because you can use react components instead of class based components

Import and usage examples of react-bootstrap vs. bootstrap:

```
react-bootstrap

<Container fluid>
	<Row>
		<Col lg={6} md={12}>...</ Col>
		<Col lg={6} md={12}>...</ Col>
		<Col lg={6} md={12}>...</ Col>
	</Row>
</Container>
```

```
bootstrap
<div class="container-fluid">
	<div class="row">
		<div class="col col-lg-6 col-md-12>...</div>
		<div class="col col-lg-6 col-md-12>...</div>
		<div class="col col-lg-6 col-md-12>...</div>
	</div>
</div>
```

\
An example of Bootstrap in a React component in the setup can be found in /components/Query.js

```
import { Modal, Button } from "react-bootstrap";
import styles from "./Query.module.css";

/**
 * Query
 * * Same modal component as /pages/index.js. Used for storybook and blog example.
 */

const Query = ({ courseID, professor, team, project }) => {
  return (
    <>
      <Modal show={true} className={styles.modal} centered>
        <Modal.Header closeButton>
          <Modal.Title>
            <h3>Silver Surfer DB Query</h3>
          </Modal.Title>
        </Modal.Header>
        <Modal.Body>
          <p className="text-muted">
            db: silver-mongo <br />
            collection: CoursePages
          </p>
          <h5>Course</h5>
          <p>{courseID}</p>
          <h5>Professor</h5>
          <p>{professor}</p>
          <h5>Team</h5>
          <p>{team}</p>
          <h5>Project Name</h5>
          <p>{project}</p>
        </Modal.Body>
        <Modal.Footer>
          <Button variant="secondary">Close</Button>
        </Modal.Footer>
      </Modal>
    </>
  );
};

export default Query;

```

## MongoDB

#### Connection

To use MongoDB in Silver Surfer we created a free database cluster through Mongo Atlas. You are given a link to connect the database encoded with the account to the cluster.

With the Next.js initialization done, I moved to connecting and querying the MongoDB database. I created a database named silver-mongo and a temporary collection named CoursePages. MongoDB is a document based database in contrast to relational databases like SQL. Instead of tables, columns, and rows, MongoDB uses collections and documents. Collections resemble tables and documents resemble row and tables. I chose MongoDB because you can make queries to specific data instead of setting up complex schemas in a relational database.

To connect our database to the app I installed the MongoDB Node.js driver `npm i mongodb`. I created a folder called `util` to hold the `connectToDatabase()` function. This keeps the project structure clean and well organized.

```
util/mongodb.js

import { MongoClient } from "mongodb"; // MongoDB Node.js Driver
import { MONGO_URI, DB_NAME } from "../config/mongo.config"; // Hidden uri link for cloud MongoDB instance

/**
 * connectToDatabase() | Connect to silver-mongo utility
 * *  [0] | if no uri or db name is given, return an error.
 * *  [1] | saves the connection after first connection. Returns cached connection on following visits.
 * *  [2] | connect to the silver-mongo db cluster using the MongoDB driver library.
 * *  [3] | select the MongoDB silver-mongo
 * *  [4] | return client instance and silver-mongo database.
 *
 * @param MONGO_URI The link given by Mongo Atlas to connect to Silver Surfer databases
 * @param DB_NAME   Name of database used by Silver Surfer
 */

// [0]
if (!MONGO_URI || !DB_NAME) {
  throw new Error(
    `No MongoDB connection link (${MONGO_URI}) or db name (${DB_NAME}) provided.`
  );
}

// [1]
let cachedClient = null;
let cachedDb = null;

export async function connectToDatabase() {
  // [1]
  if (cachedClient && cachedDb) {
    return { client: cachedClient, db: cachedDb };
  }
  // [2]
  const client = await MongoClient.connect(MONGO_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  });

  // [3]
  const db = await client.db(DB_NAME);

  // [1]
  cachedClient = client;
  cachedDb = db;

  // [4]
  return { client, db };
}

```

#### Query Data

Becuase we will be fetching data in different ways (SSG vs SSR), I won't explain in full detail of how to fetch data in a react component. What I will show is the `mongodb` library functions for querying data from silver-mongo.

These code examples are from the homepage:

```
/pages/index.js

# From util/mongodb.js
const { client, db } = await connectToDatabase();

# Function to check that client is connected
const conn = await client.isConnected();

# Query silver-mongo from CoursePage collection and return document with courseID: "CS492"
const data = await db
    .collection("CoursePage")
    .find({ courseID: "CS492" })
    .toArray();
```

You can return the `data` variable and pass it to any component inside that file. Example from the homepage:

```
/pages/index.js

export default function Home(props) {
  const { conn, query } = props; // props from MongoDB query
	.
	.
	.
}

```

## Storybook

The last thing added to set up our website is Storybook. Storybook is a UI tool used for designing and styling React components. Storybook is a developer dependency that allows you to edit and style components with it's companion web GUI. This does not contribute to the final project, but it will make it easier to design compononents independantly before it is added to the page. Here is an example of the Storybook website GUI:
<video width="750" height="500" controls>

  <source src="https://storybook.js.org/videos/storybook-hero-video-optimized.mp4" type="video/mp4">
</video>

I chose to use Storybook because I think it will be great for our team members that are more comfortable designing components without worrying about data fetching, responsiveness, and placement. This will increase design time and overall productivity when creating the app. Storybook adds a component when it is referenced in the `/stories` folder. After it is referenced for the Storybook website GUI you can pass static data to populate elements with the component I made a demo storybook component and here is how I referenced it to the website GUI.

Start with the main `.storybook` folder. **_This will not be changed_**:

```
main.js // Finds srtories

module.exports = {
  stories: ["../stories/*.stories.js"],
  addons: ["@storybook/addon-essentials", "@storybook/react"],
};

preview.js // Displays w/ HTML and CSS

import "bootstrap/dist/css/bootstrap.min.css";

export const parameters = {
  actions: { argTypesRegex: "^on[A-Z].*" },
};
```

Reference the `/components/Query` **_This is the component on the main page_**:

```
import React from "react";
import Query from "../components/Query/Query"; //import component

// template for .storybook
export default {
  title: "Storybook Example",
  component: Query,
};

// template for passing props to storybook components
const Story = (props) => <Query {...props} />;

// example of passing props to storybook component
export const Example1 = Story.bind({});
NameCard.args = {
  courseID: "CS492",
  professor: "Cindric",
  team: "Silver Foxes",
  project: "Silver Surfer",
};

// example of passing props to storybook component
export const Example2 = Story.bind({});
NameCard.args = {
  courseID: "CS492",
  professor: "Foo",
  team: "Bar",
  project: "Foo Bar",
};
```

These JS files created a Query component and created two variations of the component. Variations are created by passing props the the components. You could also create variations such as themes, this is beneficial because we could have light mode and dark mode ready when the code is finished on the back end.

## Conclusion

This blog served as documentation and inspiration for the structure and dev ops of the Silver Surfer App. Next blog will cover setting up a local dev environment and Git & GitHub practices for branches, commits, pull requests, and naming conventions.
