Flask is a lightweight web framework for building web applications in Python. It is designed to be simple and easy to use.Flask follows the model-view-controller (MVC) architectural pattern, although it is flexible enough to be used in other architectural styles as well. It provides a way to handle HTTP requests and responses, routing URLs to appropriate functions, rendering templates, and managing sessions.

**from flask import Flask, request, jsonify
from flask_pymongo import PyMongo
from bson.objectid import ObjectId
import socket**

This is import section. Imports the necessary modules and dependencies for the application. The Flask module is used to create the Flask application, while request is used to handle HTTP requests. jsonify is used to convert Python objects into JSON responses. Flask_PyMongo is an extension that integrates MongoDB with Flask, allowing easy interaction with the database. ObjectId is used to convert string identifiers to MongoDB ObjectIds. Finally, the socket module is imported to retrieve the hostname of the current machine.

**app = Flask(__name__)
app.config["MONGO_URI"] = "mongodb://mongo:27017/dev"
mongo = PyMongo(app)
db = mongo.db**

the Flask application is initialized. The Flask(__name__) line creates an instance of the Flask class, with __name__ representing the name of the current module. This instance will be the main entry point for the application.

The next line, app.config["MONGO_URI"], sets the configuration for the MongoDB connection URI. The URI specifies the hostname (mongo) and port (27017) of the MongoDB server, as well as the database name (dev) to connect to.

The PyMongo(app) line initializes the PyMongo extension with the Flask app, establishing a connection to the MongoDB server using the provided configuration. The mongo object represents the MongoDB client.

Finally, db = mongo.db assigns the MongoDB database instance to the db variable, allowing easy access to the database within the application.

By configuring the MongoDB URI and initializing the PyMongo extension, the application is now ready to interact with the MongoDB database.


 @app.route("/") decorates the index() function, which means that whenever a user accesses the root URL ("/") of the Flask application, the index() function will be invoked.

For example, if your Flask application is running on http://localhost:5000, accessing http://localhost:5000/ in a web browser will trigger the execution of the index() function.


**@app.route("/tasks")
def get_all_tasks():
    tasks = db.task.find()
    data = []
    for task in tasks:
        item = {
            "id": str(task["_id"]),
            "task": task["task"]
        }
        data.append(item)
    return jsonify(
        data=data
    )
**


This line specifies that this function will handle requests to the /tasks URL route. So, when a user accesses /tasks in the Flask application, this function will be called.

tasks = db.task.find() retrieves all documents (tasks) from the MongoDB collection named "task" using the find() method.

A list, data, is initialized to store the transformed task data.

The function iterates over each task in the tasks cursor obtained from the database query. For each task, it creates a dictionary, item, with two key-value pairs: "id" (converted to a string using str()) and "task".

The item dictionary representing each task is appended to the data list.

Finally, jsonify(data=data) converts the data list into a JSON response. The response will contain the tasks retrieved from the database, with each task represented by its "id" and "task" properties.


**@app.route("/task", methods=["POST"])
def create_task():
    data = request.get_json(force=True)
    db.task.insert_one({"task": data["task"]})
    return jsonify(
        message="Task saved successfully!"
    )**
    
This line specifies that this function will handle POST requests to the /task URL route. It means that when a user sends a POST request to /task, this function will be called. 


data = request.get_json(force=True) retrieves the JSON payload sent with the POST request using request.get_json(). The force=True argument ensures that the request is parsed as JSON, even if the Content-Type header is not explicitly set to application/json.

db.task.insert_one({"task": data["task"]}) inserts a new document (task) into the MongoDB collection named "task". The task content is extracted from the JSON payload using data["task"].

Finally, jsonify(message="Task saved successfully!") creates a JSON response with a success message. This message indicates that the task was successfully saved in the database.
    
**@app.route("/task/<id>", methods=["PUT"])
def update_task(id):
    data = request.get_json(force=True)["task"]
    response = db.task.update_one({"_id": ObjectId(id)}, {"$set": {"task": data}})
    if response.matched_count:
        message = "Task updated successfully!"
    else:
        message = "No Task found!"
  **

  
  This line specifies that this function will handle PUT requests to the /task/<id> URL route. It uses a variable placeholder <id> to capture the task ID from the URL. So, when a user sends a PUT request to /task/<id>, this function will be called, with the <id> value passed as an argument.
  
  
  data = request.get_json(force=True)["task"] retrieves the JSON payload sent with the PUT request using request.get_json(). The force=True argument ensures that the request is parsed as JSON, even if the Content-Type header is not explicitly set to application/json. It then extracts the task content from the JSON payload using ["task"].

response = db.task.update_one({"_id": ObjectId(id)}, {"$set": {"task": data}}) updates the task in the MongoDB collection named "task" that matches the provided id. It uses the update_one() method to modify the task's content using the $set operator.

The if response.matched_count condition checks if there was a task in the collection that matched the provided id. If matched_count is non-zero, it means a task was found and updated successfully.

If a task was found and updated, the message variable is set to "Task updated successfully!". Otherwise, if no task was found, the message variable is set to "No Task found!".
