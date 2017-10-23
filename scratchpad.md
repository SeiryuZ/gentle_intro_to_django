# Open terminal

Click Start | Run | Enter "cmd"   # Windows

Open terminal.app  # macOS

Open terminal with ctrl + alt + t # Linux


# navigate to folder you want, for example:

cd dev   # folder "dev" is what you want


# Cloning(Downloading) example project
git clone https://github.com/SeiryuZ/my_expenses.git


# The web app
We are going to create our own personal expenses calculator web application. The idea is for user to able to access the web app to enter their expenses, and get the list of expenses back and get a report where their money went.

The project that you have cloned are just a skeleton project, we will build the web app together.

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

Now we have our database ready for accepting or retrieving expenses information.


# urls.py, views.py, and templates

Now we need to add a view function to list all expenses. To do that we are going to add some function to accept requests and render the correct response. Navigate to `my_expenses/views.py` and add the function below:

```python
from django.shortcuts import render
from my_expenses.apps.expenses.models import Expense


def expense_list(request):
    expenses = Expense.objects.all().order_by('-created')
    total = expenses.aggregate(Sum('amount'))['amount__sum']
    context = {
        'expenses': expenses,
        'total': total
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
