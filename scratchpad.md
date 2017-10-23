# Open terminal

Click Start | Run | Enter "cmd"   # Windows

Open terminal.app  # macOS

Open terminal with ctrl + alt + t # Linux


# navigate to folder you want, for example:

cd dev   #  if folder "dev" is what you want


# Cloning(Downloading) example project
```
git clone https://github.com/SeiryuZ/my_expenses.git
```

# The web app
We are going to create our own personal expenses calculator web application. The idea is for user to able to access the web app to enter their expenses, and get the list of expenses back and get a report where their money went.

The project that you have cloned are just a skeleton project, we will build the web app together.

# Validate the skeleton web app is working
To validate the web app is working, run the following command
```python
python manage.py runserver 0:8000
```

and access `localhost:8000` in your browser. It should show some basic Django Warning. 

**Tip:** Leave this open in a separate tab


# The models
We are going to create models for our web app. The basic idea of the model is to think on what kinds of information we want to store and retrieve.

For our web app, let's make `Expense` our model. Navigate to `my_expenses/apps/expenses/models.py` and create our first Model:

```python
# Every models has to subclass from django's model`
from django.db import models


class Expense(models.Model):
    description = models.CharField(max_length=255)
    amount = models.IntegerField()

    TYPE = (
        (1, 'Food'),
        (2, 'Entertainment'),
        (3, 'Rent'),
        (4, 'Other'),
    )
    type = models.SmallIntegerField(choices=TYPE, default=4)
    
    # automatically use current time at insertion as value
    created = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        # Method to print object representation
        return f"{self.amount} on {self.created}"

```


After we create our models, we need to make sure that the database know that we have new model. Run these commands

```bash
python manage.py makemigrations expenses
python manage.py migrate
```

`makemigrations` command will create a "Migration file" that contains Django operation that represent an SQL statements to create appropriate tables for our model, everytime you change the model you need to run `makemigrations`. `migrate` command executes those instructions to the database.

Now we have our database ready for accepting or retrieving expenses information. To test that the model persists information to the database correctly we can run python with our project's Django environment with the following command:

```bash
python manage.py shell
```

Once inside the shell
```python
# Import the model first
from my_expenses.apps.expenses.models import Expense

# Create some expenses by hand
Expense.objects.create(description="KFC", amount=35000, type=1)
Expense.objects.create(description="Hokben", amount=25000, type=1)
Expense.objects.create(description="Bayar kosan", amount=2000000, type=3)
Expense.objects.create(description="Bayar hutang", amount=30000, type=4)

# Try to retrieve all expenses
Expense.objects.all()

# Get the first expenses
Expense.objects.all().order_by('created').first()

# last
Expense.objects.all().order_by('created').last()

# Get only "food" expenses
Expense.objects.filter(type=1).all()

# Get all expenses except "food"
Expense.objects.exclude(type=1).all()
```


# urls.py, views.py, and templates

Now we need to add a view function to list all expenses. To do that we are going to add some function to accept requests and render the correct response. Navigate to `my_expenses/views.py` and add the function below:

```python
from django.db.models import Sum
from django.shortcuts import render
from my_expenses.apps.expenses.models import Expense


def expense_list(request):
    # We retrieve all of our expenses in the database, we order them based on created time newest to oldest
    expenses = Expense.objects.all().order_by('-created')
    
    # We aggregate every expenses and sum them
    total = expenses.aggregate(Sum('amount'))['amount__sum']
    
    # We pass these variable to the templates
    context = {
        'expenses': expenses,
        'total': total,
        'extra': "Ignored",   # Any extraneous variable that is not used by the template will be ignored
    }
    return render(request, 'index.html', context)
```


Now,we need to prepare the template file which is referenced by the view functions above, navigate to the folder `templates` and create a new file called `index.html`

```html+django
{% extends 'base.html' %}

{% block content %}

    Your expenses:
    <table>
      {% for expense in expenses  %}
      <tr>
        <td>{{ expense.id }}</td>
        <td>{{ expense.description}}</td>
        <td><strong>{{ expense.amount}}</strong></td>
        <td>{{ expense.created }}</td>
      </tr>
      {% endfor %}
    </table>

    Your total expenses: {{ total }}

    <br /><br />
    <a href="#">Add expenses</a>

{% endblock %}
```

This templates expects a variable called `expenses` and `total` to be passed by the views. It iterates through the `expenses` variable and print its attributes in a table.

Now that we are done with the templates, we should hook the `urls.py` so requests are directed to the views we just created. We are going to catch an empty path or `root` or `/` as our path for the `expense_list` function. Open up `my_expenses/urls.py` and add the following

```python
from django.conf.urls import url
from django.contrib import admin

from .views import hello_world, expense_list, add_expense

urlpatterns = [
    url(r'^$', expense_list, name='index'),  # if client does not specify which page it want, assume it want to get expense_list
    url(r'^hello-world/$', hello_world),  
    url(r'^admin/', admin.site.urls),
]

```

Now validate that our views are correct by running the server once again

```bash
python manage.py runserver 0:8000
```

and access `localhost:8000` in your browser, it should show the expense list page correctly


