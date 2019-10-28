# flask-example

This is from the tutorial found here: https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world

## Notes

- Use `python-dotenv` package and create a `.flaskenv` file to avoid having to `export FLASK_APP=` every time
- Flask is extremely flexible in use
- Consider storing configs in a `config.py` file in the top level directory, instead of in an `app.config` file
-

### On forms

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

>This template expects a form object instantiated from the LoginForm class to be given as an argument, which you can see referenced as form. This argument will be sent by the login view function, which I still haven't written.
>
>The HTML <form> element is used as a container for the web form. The action attribute of the form is used to tell the browser the URL that should be used when submitting the information the user entered in the form. When the action is set to an empty string the form is submitted to the URL that is currently in the address bar, which is the URL that rendered the form on the page. The method attribute specifies the HTTP request method that should be used when submitting the form to the server. **The default is to send it with a GET request, but in almost all cases, using a POST request makes for a better user experience because requests of this type can submit the form data in the body of the request, while GET requests add the form fields to the URL, cluttering the browser address bar.** The novalidate attribute is used to tell the web browser to not apply validation to the fields in this form, which effectively leaves this task to the Flask application running in the server. Using novalidate is entirely optional, but for this first form it is important that you set it because this will allow you to test server-side validation later in this chapter.

#### On security of forms

>The form.hidden_tag() template argument generates a hidden field that includes a token that is used to protect the form against CSRF attacks. All you need to do to have the form protected is include this hidden field and have the SECRET_KEY variable defined in the Flask configuration. If you take care of these two things, Flask-WTF does the rest for you.
