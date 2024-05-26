# Django-React Notes App

## Project Setup

### Create Project Directory

1. Create a project directory and navigate into it:
    ```bash
    mkdir django-react-notes
    cd django-react-notes
    ```

### Set Up Virtual Environment

2. Install `pipenv` and create a virtual environment:
    ```bash
    pip install pipenv
    pipenv shell
    ```

### Install Django

3. Install Django using `pipenv`:
    ```bash
    pipenv install django
    ```

## Backend Setup (Django)

### Create Django Project and App

4. Start a new Django project:
    ```bash
    django-admin startproject backend
    cd backend
    ```

5. Create a new Django app:
    ```bash
    python manage.py startapp todo
    ```

6. Apply initial migrations and start the server:
    ```bash
    python manage.py migrate
    python manage.py runserver
    ```

### Register the App

7. Open `backend/settings.py` and modify it:

    - Add `'todo'`, `'corsheaders'`, and `'rest_framework'` to `INSTALLED_APPS`:
        ```python
        INSTALLED_APPS = [
            ...,
            'corsheaders',
            'rest_framework',
            'todo',
        ]
        ```

    - Add `'corsheaders.middleware.CorsMiddleware'` to `MIDDLEWARE`:
        ```python
        MIDDLEWARE = [
            'corsheaders.middleware.CorsMiddleware',
            ...,
        ]
        ```

    - Add `CORS_ORIGIN_WHITELIST` at the bottom:
        ```python
        CORS_ORIGIN_WHITELIST = [
            'http://localhost:3000',
        ]
        ```

### Define the Model

8. Create a model in `todo/models.py`:
    ```python
    from django.db import models

    class Todo(models.Model):
        title = models.CharField(max_length=120)
        description = models.TextField()
        completed = models.BooleanField(default=False)

        def __str__(self):
            return self.title
    ```

9. Apply migrations for the new model:
    ```bash
    python manage.py makemigrations todo
    python manage.py migrate todo
    ```

### Admin Interface

10. Register the model in `todo/admin.py`:
    ```python
    from django.contrib import admin
    from .models import Todo

    class TodoAdmin(admin.ModelAdmin):
        list_display = ('title', 'description', 'completed')

    admin.site.register(Todo, TodoAdmin)
    ```

11. Create a superuser to access the admin interface:
    ```bash
    python manage.py createsuperuser
    ```

### Set Up APIs

12. Install Django Rest Framework and CORS headers:
    ```bash
    pipenv install djangorestframework django-cors-headers
    ```

13. Create a serializer in `todo/serializers.py`:
    ```python
    from rest_framework import serializers
    from .models import Todo

    class TodoSerializer(serializers.ModelSerializer):
        class Meta:
            model = Todo
            fields = ('id', 'title', 'description', 'completed')
    ```

14. Create a view in `todo/views.py`:
    ```python
    from rest_framework import viewsets
    from .serializers import TodoSerializer
    from .models import Todo

    class TodoView(viewsets.ModelViewSet):
        serializer_class = TodoSerializer
        queryset = Todo.objects.all()
    ```

15. Update `backend/urls.py` to include the new view:
    ```python
    from django.contrib import admin
    from django.urls import path, include
    from rest_framework import routers
    from todo import views

    router = routers.DefaultRouter()
    router.register(r'todo', views.TodoView, 'todo')

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('api/', include(router.urls)),
    ]
    ```

16. Run the Django server:
    ```bash
    python manage.py runserver
    ```

## Frontend Setup (React)

### Create React App

17. In the project root, create a React app:
    ```bash
    npx create-react-app frontend
    cd frontend
    ```

18. Start the React development server:
    ```bash
    npm start
    ```

19. Install necessary packages:
    ```bash
    npm install bootstrap@4.6.0 reactstrap@8.9.0 --legacy-peer-deps
    ```

### Update React App

20. Modify `frontend/src/index.js` to include Bootstrap:
    ```javascript
    import React from 'react';
    import ReactDOM from 'react-dom';
    import 'bootstrap/dist/css/bootstrap.css';
    import './index.css';
    import App from './App';
    import reportWebVitals from './reportWebVitals';

    ReactDOM.render(
      <React.StrictMode>
        <App />
      </React.StrictMode>,
      document.getElementById('root')
    );

    reportWebVitals();
    ```

21. Update `frontend/src/App.js` to handle API requests and state management:
    ```javascript
    import React, { Component } from "react";
    import Modal from "./components/Modal";
    import axios from "axios";

    class App extends Component {
      constructor(props) {
        super(props);
        this.state = {
          viewCompleted: false,
          todoList: [],
          modal: false,
          activeItem: {
            title: "",
            description: "",
            completed: false,
          },
        };
      }

      componentDidMount() {
        this.refreshList();
      }

      refreshList = () => {
        axios
          .get("/api/todo/")
          .then((res) => this.setState({ todoList: res.data }))
          .catch((err) => console.log(err));
      };

      toggle = () => {
        this.setState({ modal: !this.state.modal });
      };

      handleSubmit = (item) => {
        this.toggle();

        if (item.id) {
          axios
            .put(`/api/todo/${item.id}/`, item)
            .then((res) => this.refreshList());
          return;
        }
        axios
          .post("/api/todo/", item)
          .then((res) => this.refreshList());
      };

      handleDelete = (item) => {
        axios
          .delete(`/api/todo/${item.id}/`)
          .then((res) => this.refreshList());
      };

      createItem = () => {
        const item = { title: "", description: "", completed: false };

        this.setState({ activeItem: item, modal: !this.state.modal });
      };

      editItem = (item) => {
        this.setState({ activeItem: item, modal: !this.state.modal });
      };

      displayCompleted = (status) => {
        if (status) {
          return this.setState({ viewCompleted: true });
        }

        return this.setState({ viewCompleted: false });
      };

      renderTabList = () => {
        return (
          <div className="nav nav-tabs">
            <span
              onClick={() => this.displayCompleted(true)}
              className={this.state.viewCompleted ? "nav-link active" : "nav-link"}
            >
              Complete
            </span>
            <span
              onClick={() => this.displayCompleted(false)}
              className={this.state.viewCompleted ? "nav-link" : "nav-link active"}
            >
              Incomplete
            </span>
          </div>
        );
      };

      renderItems = () => {
        const { viewCompleted } = this.state;
        const newItems = this.state.todoList.filter(
          (item) => item.completed === viewCompleted
        );

        return newItems.map((item) => (
          <li
            key={item.id}
            className="list-group-item d-flex justify-content-between align-items-center"
          >
            <span
              className={`todo-title mr-2 ${
                this.state.viewCompleted ? "completed-todo" : ""
              }`}
              title={item.description}
            >
              {item.title}
            </span>
            <span>
              <button
                className="btn btn-secondary mr-2"
                onClick={() => this.editItem(item)}
              >
                Edit
              </button>
              <button
                className="btn btn-danger"
                onClick={() => this.handleDelete(item)}
              >
                Delete
              </button>
            </span>
          </li>
        ));
      };

      render() {
        return (
          <main className="container">
            <h1 className="text-white text-uppercase text-center my-4">Todo app</h1>
            <div className="row">
              <div className="col-md-6 col-sm-10 mx-auto p-0">
                <div className="card p-3">
                  <div className="mb-4">
                    <button
                      className="btn btn-primary"
                      onClick={this.createItem}
                    >
                      Add task
                    </button>
                  </div>
                  {this.renderTabList()}
                  <ul className="list-group list-group-flush border-top-0">
                    {this.renderItems()}
                  </ul>
                </div>
              </div>
            </div>
            {this.state.modal ? (
              <Modal
                activeItem={this.state.activeItem}
                toggle={this.toggle}
                onSave={this.handleSubmit}
              />
            ) : null}
          </main>
        );
      }
    }

    export default App;
    ```

22. Create a `components` directory and a

 `Modal.js` file within it:
    ```bash
    mkdir src/components
    touch src/components/Modal.js
    ```

23. Implement the modal component in `frontend/src/components/Modal.js`:
    ```javascript
    import React, { Component } from "react";
    import {
      Button,
      Modal,
      ModalHeader,
      ModalBody,
      ModalFooter,
      Form,
      FormGroup,
      Input,
      Label,
    } from "reactstrap";

    export default class CustomModal extends Component {
      constructor(props) {
        super(props);
        this.state = {
          activeItem: this.props.activeItem,
        };
      }

      handleChange = (e) => {
        let { name, value } = e.target;

        if (e.target.type === "checkbox") {
          value = e.target.checked;
        }

        const activeItem = { ...this.state.activeItem, [name]: value };

        this.setState({ activeItem });
      };

      render() {
        const { toggle, onSave } = this.props;

        return (
          <Modal isOpen={true} toggle={toggle}>
            <ModalHeader toggle={toggle}>Todo Item</ModalHeader>
            <ModalBody>
              <Form>
                <FormGroup>
                  <Label for="todo-title">Title</Label>
                  <Input
                    type="text"
                    id="todo-title"
                    name="title"
                    value={this.state.activeItem.title}
                    onChange={this.handleChange}
                    placeholder="Enter Todo Title"
                  />
                </FormGroup>
                <FormGroup>
                  <Label for="todo-description">Description</Label>
                  <Input
                    type="text"
                    id="todo-description"
                    name="description"
                    value={this.state.activeItem.description}
                    onChange={this.handleChange}
                    placeholder="Enter Todo description"
                  />
                </FormGroup>
                <FormGroup check>
                  <Label check>
                    <Input
                      type="checkbox"
                      name="completed"
                      checked={this.state.activeItem.completed}
                      onChange={this.handleChange}
                    />
                    Completed
                  </Label>
                </FormGroup>
              </Form>
            </ModalBody>
            <ModalFooter>
              <Button
                color="success"
                onClick={() => onSave(this.state.activeItem)}
              >
                Save
              </Button>
            </ModalFooter>
          </Modal>
        );
      }
    }
    ```

### Backend Integration

24. Install `axios` for making HTTP requests:
    ```bash
    npm install axios@0.21.1
    ```

25. Update `frontend/package.json` to add a proxy:
    ```json
    {
      "name": "frontend",
      "version": "0.1.0",
      "private": true,
      "proxy": "http://localhost:8000",
      "dependencies": {
        "axios": "^0.21.1",
        "bootstrap": "^4.6.0",
        "react": "^16.5.2",
        "react-dom": "^16.5.2",
        "react-scripts": "2.0.5",
        "reactstrap": "^8.9.0"
      }
    }
    ```

## Running the Project

### Start the Backend Server

26. From the `backend` directory, start the Django server:
    ```bash
    python manage.py runserver
    ```

### Start the Frontend Server

27. From the `frontend` directory, start the React development server:
    ```bash
    npm start
    ```

## Testing the Application

- Visit `http://localhost:3000` to interact with the React frontend.
- Visit `http://localhost:8000/admin` to manage the Todo items through Django admin.
- The React frontend will communicate with the Django backend using the proxy configuration. You can create, read, update, and delete Todo items through the frontend interface.

With this setup, you now have a fully functional notes application using Django for the backend and React for the frontend.
