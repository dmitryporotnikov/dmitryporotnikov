---
id: 708
title: 'Setting Up a Flask Web Application with Gunicorn on WSL'
summary: 'Setting Up a Flask Web Application with Gunicorn on WSL'
date: '2023-10-18T09:00:45+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=708'
permalink: /2023/10/18/setting-up-a-flask-web-application-with-gunicorn-on-wsl/
categories:
    - Linux
---

### Setting Up a Flask Web Application with Gunicorn on WSL

Flask is one of the most popular Python web frameworks, known for its simplicity and ease of use. Gunicorn (short for Green Unicorn) acts as a Web Server Gateway Interface HTTP server, serving as a link between your Flask app and the outside world. When developing Flask applications on a Windows system, Windows Subsystem for Linux can offer a more Linux-like environment that’s often easier to work with.

Here are the steps needed to create a flask app served by Gunicorn on WSL:

#### Step 1: Install Required Packages

Before we begin, it’s essential to make sure that Python and pip (Python’s package installer) are installed on your WSL terminal. To do so, open your WSL terminal and execute the following commands:

```bash
sudo apt update
sudo apt install python3 python3-pip
sudo apt install python3.10-venv
```

This will update your package list and install Python 3, pip, and the Python 3.10 virtual environment package.

#### Step 2: Create a Virtual Environment

While it’s optional to use a Python virtual environment, it’s a good practice to isolate each project’s dependencies. To create a new virtual environment, run:

```bash
python3 -m venv myenv
```

To activate the environment:

```bash
source myenv/bin/activate
```

##### Step 3: Install Flask and Gunicorn

With the virtual environment activated (or globally, if you prefer), you can now install Flask and Gunicorn:

```bash
pip install Flask gunicorn
```

##### Step 4: Create Your Flask App

Create a new Python file named `app.py` and insert the following code:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

This is a minimal Flask application that will display “Hello, World!” when accessed.

##### Step 5: Test Your App

Before deploying the app, test it locally using Flask’s built-in server:

```bash
export FLASK_APP=app.py flask run
```

Open a web browser and navigate to `http://127.0.0.1:5000/`. If everything is set up correctly, you should see “Hello, World!” displayed.

##### Step 6: Run with Gunicorn

To serve your Flask app using Gunicorn, execute the following command:

```bash
gunicorn app:app
```

By default, Gunicorn will serve your app on `http://127.0.0.1:8000/`. You can specify a different port by using the `-b` flag:

```bash
gunicorn -b 0.0.0.0:8000 app:app
```
