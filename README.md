# flask-example

This is from the tutorial found here: https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world

## General notes

- Use `python-dotenv` package and create a `.flaskenv` file to avoid having to `export FLASK_APP=` every time
- Consider storing configs in a `config.py` file in the top level directory, instead of in an `app.config` file

### Forms

`forms.py`:

```
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import DataRequired

class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    remember_me = BooleanField('Remember Me')
    submit = SubmitField('Sign In')
```

`login.html`:

```
{% extends "base.html" %}

{% block content %}
    <h1>Sign In</h1>
    <form action="" method="post" novalidate>
        {{ form.hidden_tag() }}
        <p>
            {{ form.username.label }}<br>
            {{ form.username(size=32) }}
        </p>
        <p>
            {{ form.password.label }}<br>
            {{ form.password(size=32) }}
        </p>
        <p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
        <p>{{ form.submit() }}</p>
    </form>
{% endblock %}
```

>This template expects a form object instantiated from the LoginForm class to be given as an argument, which you can see referenced as form. This argument will be sent by the login view function.
>
>The HTML &lt;form&gt; element is used as a container for the web form. The action attribute of the form is used to tell the browser the URL that should be used when submitting the information the user entered in the form. When the action is set to an empty string the form is submitted to the URL that is currently in the address bar, which is the URL that rendered the form on the page. The method attribute specifies the HTTP request method that should be used when submitting the form to the server. **The default is to send it with a GET request, but in almost all cases, using a POST request makes for a better user experience because requests of this type can submit the form data in the body of the request, while GET requests add the form fields to the URL, cluttering the browser address bar.** The novalidate attribute is used to tell the web browser to not apply validation to the fields in this form, which effectively leaves this task to the Flask application running in the server. Using novalidate is entirely optional, but for this first form it is important that you set it because this will allow you to test server-side validation later in this chapter.

#### Security of forms

>The form.hidden_tag() template argument generates a hidden field that includes a token that is used to protect the form against CSRF attacks. All you need to do to have the form protected is include this hidden field and have the SECRET_KEY variable defined in the Flask configuration. If you take care of these two things, Flask-WTF does the rest for you.

### Databases

>Flask does not support databases natively. This is one of the many areas in which Flask is intentionally not opinionated, which is great, because you have the freedom to choose the database that best fits your application instead of being forced to adapt to one.

>While there are great database products in both groups, my opinion is that relational databases [like SQL] are a better match for applications that have structured data such as lists of users, blog posts, etc., while NoSQL databases tend to be better for data that has a less defined structure.

We are using `flask-sqalchemy`:

>...which is an Object Relational Mapper or ORM. ORMs allow applications to manage a database using high-level entities such as classes, objects and methods instead of tables and SQL. The job of the ORM is to translate the high-level operations into database commands.

>SQLAlchemy supports a long list of database engines, including the popular MySQL, PostgreSQL and SQLite. This is extremely powerful, because you can do your development using a simple SQLite database that does not require a server, and then when the time comes to deploy the application on a production server you can choose a more robust MySQL or PostgreSQL server, without having to change your application.

#### Database migrations

>...relational databases are centered around structured data, so when the structure changes the data that is already in the database needs to be migrated to the modified structure.

We are using `flask-migrate` with `SQLite`::

>...a Flask wrapper for Alembic, a database migration framework for SQLAlchemy.

>Working with database migrations adds a bit of work to get a database started, but that is a small price to pay for a robust way to make changes to your database in the future.

>SQLite databases are the most convenient choice for developing small applications, sometimes even not so small ones, as each database is stored in a single file on disk and there is no need to run a database server like MySQL and PostgreSQL.

`SQLALCHEMY_DATABASE_URI` is found in `config.py`:

>The Flask-SQLAlchemy extension takes the location of the application's database from the SQLALCHEMY_DATABASE_URI configuration variable.

>...it is in general a good practice to set configuration from environment variables, and provide a fallback value when the environment does not define the variable. In this case I'm taking the database URL from the DATABASE_URL environment variable, and if that isn't defined, I'm configuring a database named app.db located in the main directory of the application, which is stored in the basedir variable.

- SQLite will automatically check for an existing database and create one for you, if there isn't one present. **This is not the case for MySQL or PostgreSQL**
- `flask-sqlalchemy` uses "snake case" naming for DB tables by default

#### Database models

>The data that will be stored in the database will be represented by a collection of classes, usually called database models. The ORM layer within SQLAlchemy will do the translations required to map objects created from these classes into rows in the proper database tables.

`id` fields:

>The id field is usually in all models, and is used as the primary key. Each user in the database will be assigned a unique id value, stored in this field. Primary keys are, in most cases, automatically assigned by the database, so I just need to provide the id field marked as a primary key.

### Extensions

Most extensions are initialized like this (in the `__init__.py`):

```
db = SQLAlchemy(app)
migrate = Migrate(app, db)
```

### Logins

Password hashing and verification:

```
from werkzeug.security import generate_password_hash, check_password_hash

# ...

class User(db.Model):
    # ...

    def set_password(self, password):
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        return check_password_hash(self.password_hash, password)
```

`flask-login`:

- Required items
  - `is_authenticated`: a property that is True if the user has valid credentials or False otherwise.
  - `is_active`: a property that is True if the user's account is active or False otherwise.
  - `is_anonymous`: a property that is False for regular users, and True for a special, anonymous user.
  - `get_id()`: a method that returns a unique identifier for the user as a string (unicode, if using Python 2).

>Flask-Login keeps track of the logged in user by storing its unique identifier in Flask's user session
