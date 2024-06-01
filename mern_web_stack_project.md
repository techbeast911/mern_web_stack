### MERN WEB STACK

The MERN web stack consists of MongoDB, Express.js, React, and Node.js. MongoDB is a NoSQL database for storing data. 
Express.js is a web application framework for Node.js, handling the server-side logic. 
React is a front-end library for building user interfaces. 
Node.js is a JavaScript runtime that executes code outside a web browser, enabling server-side scripting. 
Together, they allow for full-stack development using JavaScript throughout.

#### In this project we will create a simple to-do list app

### Background configuration
update ubuntu

   sudo apt update

upgrade ubuntu

   sudo apt upgrade


lets get the location of Node.js software from ubuntu repositories

    curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -

install node.js
    sudo apt-get install nodejs -y

this command above installs both node.js and npm(a node.js package manager like apt for ubuntu and pip for python)

verify node version
    
    node -v

verify node installation with command below

    npm -v



### Application code setup

make a new directory for your To-do project

   mkdir Todo

now navigate to our todo directory

   cd Todo

now we use npm init to initialize our project , so that a new file package.json will be created,
this file typically contains informations about our file and dependencies our project needs to run.  stopped at npm init

### Install Expressjs

Express.js is a minimalist web application framework for Node.js, designed for building web applications and APIs. 
It simplifies the process of handling HTTP requests and responses, enabling efficient routing, middleware integration, and template rendering. 
With its flexible and unopinionated design, Express.js provides developers with a robust set of features to create server-side applications, 
while allowing them to choose additional libraries and tools as needed.

    npm install express

create a file index.js with the command below
     touch index.js


install dotenv module

     npm install dotenv


use vim to open index.js file with

   vim index.js

type the code below into the opened file and save it.

    const express = require('express');
    require('dotenv').config();
    const app = express();
    const port = process.env.PORT || 5000;
    app.use((req, res, next) => {
    res.header("Access-Control-Allow-Origin", "\*");
    res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With,
    Content-Type, Accept");
    next();
    });
    app.use((req, res, next) => {
    res.send('Welcome to Express');
    });
    app.listen(port, () => {
    console.log(`Server running on port ${port}`)
    });


Now its time to start our server to see if it works,open your terminal in same directory as index.js and type:

    node index.js

now we create a custom tcp security rule in aws to open port 5000

To open port 5000 on an AWS EC2 instance, you need to modify the security group associated with your instance to allow inbound traffic on that port.
Here are the steps to do that:

    Log into the AWS Management Console:
        Navigate to the EC2 Dashboard.

    Select Your EC2 Instance:
        Find and select the instance you want to configure.

    View Security Groups:
        In the instance details, locate the Security Groups section. Click on the security group linked to your instance.

    Edit Inbound Rules:
        In the security group details, go to the Inbound rules tab.
        Click on Edit inbound rules.

    Add a New Rule:
        Click Add rule.
        Set the Type to Custom TCP Rule.
        Set the Port range to 5000.
        Set the Source to 0.0.0.0/0 if you want to allow traffic from any IP address. For more restricted access, specify a particular IP address or range.

    Save the Rule:
        Click Save rules to apply the changes


### Routes
there are 3 actions that our to do application needs to be able to do

1)create a new task
2)display list of all tasks
3)delete a completed task

each task will be assigned to a particular endpoint, and will use different standard http request methods:
   
   POST,GET,DELETE

For each task we need to create routes that will define various end points that the to-do app will depend on

, so lets create a folder routes:

    mkdir routes

navigate into the routes directory

   cd routes

create a file api.js with the command below

    touch api.js


open the file with the command below

   vim api.js

paste this code

    const express = require('express');
    const router = express.Router();

    router.get('/todos', (req, res, next) => {
        // Your GET /todos logic here
    });

    router.post('/todos', (req, res, next) => {
        // Your POST /todos logic here
    });

    router.delete('/todos/:id', (req, res, next) => {
        // Your DELETE /todos/:id logic here
    });

    module.exports = router;




### Models 

Since the app is going to make use of mondodb which is a nosql database we need to create a model;
A model is what makes javascript applications interactive . We will also use the models to define the database schema,this is very important
so we will be able to define the fields stored in each Mondodb document. A schema is a blueprint of how the database will be constructed
,including other data fields that may not be required to be stored in database. These are known as virtual properties.


change directory back to the Todo folder with

  cd ..

install mongoose

  npm install mongoose

create a new folder with

  mkdir models 

change directory into models

  cd models


create a file todo.js

   touch todo.js

open todo.js with vim

  vim todo.js

paste the code in the opened file

  const mongoose = require('mongoose');
  const Schema = mongoose.Schema;
  //create schema for todo
  const TodoSchema = new Schema({
  action: {
  type: String,
  required: [true, 'The todo text field is required']
  }
  })
  //create model for todo
  const Todo = mongoose.model('todo', TodoSchema);
  module.exports = Todo;



now we need to update our routes from the file api.js to make use of the new model, delete the code there and paste this new one below

    const express = require('express');
    const router = express.Router();
    const Todo = require('../models/todo');

    // GET all todos, exposing only the id and action fields
    router.get('/todos', (req, res, next) => {
        Todo.find({}, 'action')
            .then(data => res.json(data))
            .catch(next);
    });

    // POST a new todo
    router.post('/todos', (req, res, next) => {
        if (req.body.action) {
            Todo.create(req.body)
                .then(data => res.json(data))
                .catch(next);
        } else {
            res.status(400).json({
                error: "The input field is empty"
            });
        }
    });

    // DELETE a todo by id
    router.delete('/todos/:id', (req, res, next) => {
        Todo.findOneAndDelete({ "_id": req.params.id })
            .then(data => res.json(data))
            .catch(next);
    });

    module.exports = router;


mongodb database

create a file in your todo directory and name it .env
   touch .end
   vi.env

Add the connection string access to the database in it, just as below;


    DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>? 
    retryWrites=true&w=majority'
   

now we need to update the index.jsto reflect the use of the .env so that node.js can connect to the database
update the file with the update below,
  vim index.js

now paste code below


    const express = require('express');
    const bodyParser = require('body-parser');
    const mongoose = require('mongoose');
    const routes = require('./routes/api');
    const path = require('path');
    require('dotenv').config();

    const app = express();
    const port = process.env.PORT || 5000;

    // Connect to the database
    mongoose.connect(process.env.DB, {
        useNewUrlParser: true,
        useUnifiedTopology: true
    })
    .then(() => console.log('Database connected successfully'))
    .catch(err => console.log(err));

    // Since mongoose promise is deprecated, we override it with Node's promise
    mongoose.Promise = global.Promise;

    // Middleware to set CORS headers
    app.use((req, res, next) => {
        res.header("Access-Control-Allow-Origin", "*");
        res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
        next();
    });

    // Middleware to parse JSON bodies
    app.use(bodyParser.json());

    // Use the routes defined in the 'api' router
    app.use('/api', routes);

    // Error handling middleware
    app.use((err, req, res, next) => {
        console.log(err);
        next();
    });

    // Start the server
    app.listen(port, () => {
        console.log(`Server running on port ${port}`);
    });

run 
  node index.js

you will see database connected successfully


### TESTING BACKEND CODE WITHOUT FRONTEND USING RESTFUL API

in this project we will use postman to test our api,now we will create a post request to the api todo http://publicip or dns:5000/api/todos
make sure to header key (content-type as application/json)

post request





#### GET REQUEST
A GET request is used to retrieve data from a server at a specified resource. In Postman, making a GET request is straightforward. Here’s how you can do it:
Steps to Make a GET Request in Postman

    Open Postman:
    Launch the Postman application on your computer.

    Create a New Request:
        Click on the "New" button (usually at the top left).
        Select "Request".
        Enter a name for your request and optionally a description, then save it in a collection or create a new collection.

    Set the Request Type and URL:
        Use the dropdown menu next to the URL input field to set the request type to GET.
        Enter the URL you want to make the GET request to. For example:

        http

        http://<your-ec2-instance>:5000/api/todos

    Add Headers (if necessary):
        Click on the "Headers" tab (below the URL input field).
        Add any necessary headers by entering the header name in the "Key" column and the corresponding value in the "Value" column. For most simple GET requests, headers are not required unless your API expects them.

    Send the Request:
        Click the "Send" button to make the GET request.
        Postman will display the response from the server in the lower half of the screen.

Example: Making a GET Request

    Open Postman and Create a New Request:
        Click on "New", select "Request", and enter a name like "Get Todos".

    Set Request Type and URL:
        Set the request type to GET.
        Enter the URL: http://<your-ec2-instance>:5000/api/todos.

    Add Headers (if necessary):
        Click on the "Headers" tab if your API requires specific headers. For a basic GET request to a typical REST API, you usually don't need additional headers.

    Send the Request:
        Click the "Send" button.
        Review the response displayed in the lower half of the Postman interface.

Interpreting the Response

    Status Code: Indicates the result of the HTTP request. Common status codes include:
        200 OK: The request was successful, and the server responded with the requested data.
        404 Not Found: The server could not find the requested resource.
        500 Internal Server Error: The server encountered an error processing the request.

    Response Body: Contains the data returned by the server. For example, if you’re requesting a list of todos, you might see a JSON array of todo items.



#### DELETE REQUEST

A DELETE request is used to delete a resource from a server. In Postman, making a DELETE request is straightforward. Here’s how you can do it:
Steps to Make a DELETE Request in Postman

    Open Postman:
    Launch the Postman application on your computer.

    Create a New Request:
        Click on the "New" button (usually at the top left).
        Select "Request".
        Enter a name for your request and optionally a description, then save it in a collection or create a new collection.

    Set the Request Type and URL:
        Use the dropdown menu next to the URL input field to set the request type to DELETE.
        Enter the URL of the resource you want to delete. For example:

        http

        http://<your-ec2-instance>:5000/api/todos/<id>

        Replace <your-ec2-instance> with your EC2 instance's public IP address or domain name.
        Replace <id> with the ID of the specific resource you want to delete.

    Add Headers (if necessary):
        Click on the "Headers" tab (below the URL input field).
        Add any necessary headers by entering the header name in the "Key" column and the corresponding value in the "Value" column. For most DELETE requests, headers are not required unless your API expects them.

    Send the Request:
        Click the "Send" button to make the DELETE request.
        Postman will display the response from the server in the lower half of the screen.

Example: Making a DELETE Request

    Open Postman and Create a New Request:
        Click on "New", select "Request", and enter a name like "Delete Todo".

    Set Request Type and URL:
        Set the request type to DELETE.
        Enter the URL: http://<your-ec2-instance>:5000/api/todos/<id> (replace <id> with the actual ID of the todo you want to delete).

    Add Headers (if necessary):
        Click on the "Headers" tab if your API requires specific headers. For a basic DELETE request to a typical REST API, you usually don't need additional headers.

    Send the Request:
        Click the "Send" button.
        Review the response displayed in the lower half of the Postman interface.





### FRONT-END CREATION

since we are done with our back_end and api, we need to create a user interface for a web client(browser) to interact with via api.
to start out on our frontend of the To-do app,we will use the create-react-app command to scaffold our app.

in the same directory as my backend code which is the todo directory run:

   npx create-react-app client

this will create a new folder in your todo directory called client, where you will save all your react code

install nodemon,it is used to run and monitor the server,if there is any change in the server code, nodemon will detect it and automatically reload the server

   npm install nodemon --save-dev

in the Todo folder open the package.json and replace the scripts part with code below


    {
    "name": "todo",
    "version": "1.0.0",
    "main": "index.js",
    "scripts": {
        "start": "node index.js",
        "start-watch": "nodemon index.js",
        "dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
    },
    "author": "",
    "license": "ISC",
    "description": "",
    "dependencies": {
        "dotenv": "^16.4.5",
        "express": "^4.19.2",
        "mongoose": "^8.4.0"
    },
    "devDependencies": {
        "nodemon": "^3.1.1"
    }
    }
    


#### CONFIGURE PROXY IN PACKAGE.JSON

1)change directory to client

   cd client

2)open the package.json

   vi package.json

add the key value pair of proxy in the package.json

  {
  "name": "client",
  "version": "0.1.0",
  "private": true,
  "proxy": "http://localhost:5000"
 }

  ![alt text](image-1.png)


ensure you are in your todo directory, 

 npm run dev

in order to be able to access it from your browser, you have to open tcp port 3000 


### CREATING REACT COMPONENTS

For our todo app there will be 2 stateful components and one stateless component.
from your Todo directory run
  cd client

move to src directory
   cd src

inside src folder create another folder called components
    mkdir components

move into the components directory with 
    cd components

in components directory create 3 files
   touch Input.js ListTodo.js Todo.js

open Input.js file
    vi Input.js

paste in the following code 

    import React, { Component } from 'react';
    import axios from 'axios';

    class Input extends Component {
    state = {
        action: ""
    }

    addTodo = () => {
        const task = { action: this.state.action };
        if (task.action && task.action.length > 0) {
        axios.post('/api/todos', task)
            .then(res => {
            if (res.data) {
                this.props.getTodos();
                this.setState({ action: "" });
            }
            })
            .catch(err => console.log(err));
        } else {
        console.log('input field required');
        }
    }

    handleChange = (e) => {
        this.setState({
        action: e.target.value
        });
    }

    render() {
        let { action } = this.state;
        return (
        <div>
            <input type="text" onChange={this.handleChange} value={action} />
            <button onClick={this.addTodo}>add todo</button>
        </div>
        );
    }
    }

    export default Input;

to make use of axis which is a promise base http client for the browser and node.js you need to cd into your client from terminal
and run yarn add axios or npm install axios.

move to src folder
 cd..

move to the clients folder
  cd ..

install axios
  npm install axios

open your listTodo.js
  vi ListTodo.js

paste the following code in it

    import React from 'react';

    const ListTodo = ({ todos, deleteTodo }) => {
    return (
        <ul>
        {
            todos && todos.length > 0 ? (
            todos.map(todo => {
                return (
                <li key={todo._id} onClick={() => deleteTodo(todo._id)}>
                    {todo.action}
                </li>
                )
            })
            ) : (
            <li>No todo(s) left</li>
            )
        }
        </ul>
    );
    }

    export default ListTodo;

then in your Todo.js paste

    import React, { Component } from 'react';
    import axios from 'axios';
    import Input from './Input';
    import ListTodo from './ListTodo';

    class Todo extends Component {
    state = {
        todos: []
    }

    componentDidMount() {
        this.getTodos();
    }

    getTodos = () => {
        axios.get('/api/todos')
        .then(res => {
            if (res.data) {
            this.setState({
                todos: res.data
            })
            }
        })
        .catch(err => console.log(err))
    }

    deleteTodo = (id) => {
        axios.delete(`/api/todos/${id}`)
        .then(res => {
            if (res.data) {
            this.getTodos()
            }
        })
        .catch(err => console.log(err))
    }

    render() {
        let { todos } = this.state;
        return (
        <div>
            <h1>My Todo(s)</h1>
            <Input getTodos={this.getTodos} />
            <ListTodo todos={todos} deleteTodo={this.deleteTodo} />
        </div>
        )
    }
    }

    export default Todo;

we need to make little adjustments to your react code.Delete the logo and adjust our App.js

move to the src folder
  cd ..

make sure you are in the src folder then run

 vi App.js

paste the code below into it

    import React from 'react';
    import Todo from './components/Todo';
    import './App.css';

    const App = () => {
    return (
        <div className="App">
        <Todo />
        </div>
    );
    }

    export default App;


in the src directory open App.css and paste the code below

    .App {
    text-align: center;
    font-size: calc(10px + 2vmin);
    width: 60%;
    margin-left: auto;
    margin-right: auto;
    }

    input {
    height: 40px;
    width: 50%;
    border: none;
    border-bottom: 2px #101113 solid;
    background: none;
    font-size: 1.5rem;
    color: #787a80;
    }

    input:focus {
    outline: none;
    }

    button {
    width: 25%;
    height: 45px;
    border: none;
    margin-left: 10px;
    font-size: 25px;
    background: #101113;
    border-radius: 5px;
    color: #787a80;
    cursor: pointer;
    }

    button:focus {
    outline: none;
    }

    ul {
    list-style: none;
    text-align: left;
    padding: 15px;
    background: #171a1f;
    border-radius: 5px;
    }

    li {
    padding: 15px;
    font-size: 1.5rem;
    margin-bottom: 15px;
    background: #282c34;
    border-radius: 5px;
    overflow-wrap: break-word;
    cursor: pointer;
    }

    @media only screen and (min-width: 300px) {
    .App {
        width: 80%;
    }

    input {
        width: 100%;
    }

    button {
        width: 100%;
        margin-top: 15px;
        margin-left: 0;
    }
    }

    @media only screen and (min-width: 640px) {
    .App {
        width: 60%;
    }

    input {
        width: 50%;
    }

    button {
        width: 30%;
        margin-left: 10px;
        margin-top: 0;
    }
    }

in the src directory open the index.css

    body {
    margin: 0;
    padding: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto",
    "Oxygen",
    "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
    sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    box-sizing: border-box;
    background-color: #282c34;
    color: #787a80;
    }
    code {
    font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
    monospace;
    }


go to the Todo directory
 cd ../..

run 

  npm run dev

our todo app is now ready