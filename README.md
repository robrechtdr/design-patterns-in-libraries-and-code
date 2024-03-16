# Design patterns in common libraries
Collection of design patterns in how they occur in common libraries and more based upon patterns in the GoF book and sources like refactoring.guru.

The point of this repo is to leverage pre-existing experience with third party repo's  for people with some exposure to common third party libraries. It's much faster to learn from code you've already seen with real context vs code from scratch that in it's simple example implementation can be hard to see the benefit/sense from or to remember.

## Factory (Creational)

The factory pattern is leveraged when you use a method that creates a new instance each time you call it. 

Vs e.g. Singleton, where if you call this method multiple times, it 
will point to the same instance.


### Python Example - Django: objects.filter() returning new QuerySet instance

The `MyMode.objects.filter` method returns a new instance of a QuerySet each time you call it. 

```python
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.CharField(max_length=100)
    publish_year = models.IntegerField()

    def __str__(self):
        return self.title

# Somewhere in your view or another function
# Retuns a QuerySet instance
recent_books = Book.objects.filter(publish_year__gt=2010)
for book in recent_books:
    print(book.title, book.author)

```

### Python Example - Requests: requests.get() returning new Response instance

```python
import requests

response = requests.get('https://api.example.com/data')
if response.status_code == 200:
    data = response.json()
    for item in data:
        print(item)
else:
    print("Failed to retrieve data")

```

Every time we call `requests.get('https://myurl.com')` we get a new Response instance.

## Builder (Creational)

### Python Example - Matplotlib: building Axes for plot

```python
import matplotlib.pyplot as plt

# Creating a figure
fig, ax = plt.subplots()

# Incrementally building the plot
ax.plot([1, 2, 3, 4], [1, 4, 2, 3], label='Line 1') # Adding a line plot
ax.scatter([1, 2, 3, 4], [1, 4, 2, 3], color='red', label='Scatter Points') # Adding scatter plot
ax.set_xlabel('X Axis')  # Setting X-axis label
ax.set_ylabel('Y Axis')  # Setting Y-axis label
ax.set_title('Simple Plot')  # Setting title of the plot
ax.legend()  # Adding a legend

# Display the plot
plt.show()
```

Here we incrementally add to (or 'build') the `ax` instance (`Axes` object) which is bound to the `plt` instance, preparing it for calling the `plt.show()` method.


### Python Example - Pandas: method chaining as kind of builder

```python
import pandas as pd

# Sample data
data = {'A': [1, 2, 3, 4],
        'B': [5, 6, 7, 8],
        'C': [9, 10, 11, 12]}

df = pd.DataFrame(data)

# Using method chaining to build up a series of transformations
result = (df.assign(D=lambda x: x['A'] + x['B'])
           .query('D > 5')
           .drop(columns=['C'])
           .rename(columns={'D': 'SumAB'})
           .reset_index(drop=True))

print(result)
```

Pandas allows you to incrementally build results but it does not use the traditional building pattern in the sense that calling a method doesn't change the state of the object, it simply returns the result as a new object. 

## Singleton (Creational)

### Python Example - Python: same logger instance accross modules

```python
# module1.py
import logging

logger = logging.getLogger('myapp_logger')

def function1():
    logger.info('Message from function1 in module1')
```

```python
# module2.py
import logging

logger = logging.getLogger('myapp_logger')

def function1():
    logger.info('Message from function2 in module2')
```

You can retrieve the same logger instance accross modules. However, the logger class doesn't enforce the use of a singleton pattern as it gives you a different instance if you call `getLogger` with a different name. So it's not a singleton in the strict sense.


## Adapter (Structural)

The Adapter pattern is when we leverage a class (usually) that provides a 
uniform interface to access different implementations/interfaces with similar functionality.

Often used in:
* Uniform interface for db access bc many different db products (e.g. Postgresql, sqlite, ...) that have different interfaces and functionality. 
* Uniform interface to read files from different file formats (e.g. csv, json,...)
* Uniform interface to access different web services (e.g. REST, SOAP,...)
* Uniform interface to access different APIs (e.g. Google, Facebook,...)
* Uniform interface (e.g. [splinter lib](https://splinter.readthedocs.io/en/stable/)) in browser automation to access different browsers (e.g. Chrome, Firefox,...)

### Python Example - Django: Under the hood of django.db.backends.postgresql

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'db_name',                      
        #...
    }
}
```

Under the hood django will use the [`django.db.backends.postgresql.base.DatabaseWrapper`](https://github.com/django/django/blob/main/django/db/backends/postgresql/base.py#L107) which is an adapter class to provide an interface 
when accessing postgres that is uniform accross different relational database products.

## Composite (Structural)

### Python Example - BeautifulSoup: Tag instances

```python
from bs4 import BeautifulSoup

html_doc = "<div><p>Paragraph in a div.</p><p>Another paragraph in a div.</p></div>"
soup = BeautifulSoup(html_doc, 'html.parser')

div_tag = soup.div

# To get first p tag 
p_tag = div_tag.children[0]
```

The implementation is such that you can treat each Tag object uniformly, whether it's a single element or a composite of several elements.

## Decorator (Structural)

The decorator pattern allows you to add functionality to an existing function or class without modifying it.

### Python Example - Pytest: pytest.mark.skipif decorator

```python
import pytest
import sys

@pytest.mark.skipif(sys.version_info < (3, 7), reason="requires python3.7 or higher")
class TestClass:
    def test_method_1(self):
        # Test code here...
        assert True

    def test_method_2(self):
        # Test code here...
        assert True

```

Applied to a class.

### Python Example - Python Retry: retry decorator

```python
@retry(
retry_on=(ZeroDivisionError,),
    max_retries=2,
    backoff_factor=1,
    supress_exception=True,
    retry_logger=LOGGER
)
def my_division_handler(num: int, den: int):
    return num / den

```

Applied to a function.

## Flyweight (Structural)

### Python Example - Python: quoted strings

```python
one = 'hi'
two = 'hi'
three = "hi"

>>> id(one) == id(two)
True
>>> id(two) == id(three)
True

```

Python internally doesn't create a new instance when you use the same quoted string in order to save memory.

Traditionally flyweight refers to a pattern leveraging an object instance with a certain state shared accross other instances to save memory.
In the above it's not enforced to use it as a part. But e.g. `li = ['hi', 'hello']; li2 = ['hoho', 'hi']` uses `'hi'` as a sub part. Here also the flyweight only carries a single attribute whereas the traditional example you would carry multiple attributes on the flyweight.


## Proxy (Structural)

The Proxy pattern is a structural design pattern that provides a surrogate or placeholder for another object to control access to it. 

### Python Example - Django: ForeignKey proxy instance

```python
class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)

# step1: When accessing 'author', the related Author object is fetched lazily.
book = Book.objects.get(id=1)
# Step 2: The related Author object is loaded here
author = book.author  

```

Just after step 1, the ForeignKey field is represented by an instance of ForwardManyToOneDescriptor, a proxy. As soon as you actually access the attribute in step 2, then the Author instance is loaded.

## Command (Behavioural)

### Python Example - Celery: tasks

```python
from celery import Celery

app = Celery('my_app', broker='pyamqp://guest@localhost//')

@app.task
def long_running_task():
    # Long running task implementation
    pass

# Queue the task
task = long_running_task.apply_async()

# Later, if needed, cancel (revoke) the task
app.control.revoke(task.id)

```

A task is an object that represents a request for an operation, a 'task to be executed'. `apply_async()` is the main command method for this 
operation.


## Observer (Behavioural)

The Observer pattern allows tracking state changes in an object ('subject') from an 'observer object'.

### Python Example - Django: receiver in django signals

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from myapp.models import MyModel

@receiver(post_save, sender=MyModel)
def my_handler(sender, instance, created, **kwargs):
    # Code to run after saving an instance of MyModel
```

The receiver (using decorator pattern here) allows you to mark a function as an observer that tracks changes in the MyModel 'subject'. E.g. you can access a MyModel instance
via the `instance` parameter as soon as a MyModel instance
is saved.

## Strategy (Behavioural)

The strategy pattern is about encapsulating or abstracting different strategies of execution and utilization of  attributes; each which may result in different outputs.

These strategy abstractions are then to be uniformly 'plugged in' or selected from an interface as alternatives.


### Example Python - logger.addHandler()

```python
import logging
logger = logging.getLogger('my_logger')
logger.addHandler(logging.StreamHandler())
logger.addHandler(logging.FileHandler('logfile.log'))
```

Different handlers (like StreamHandler, FileHandler, SocketHandler) represent different strategies for where and how log messages should be output.

* `StreamHandler`
    * Where: StreamHandler writes logging messages to a stream, which could be any object with a write() method. By default, it uses sys.stderr.
    * How: It sends the log messages to the console (standard error or standard output, depending on configuration).
* `FileHandler`
    * Where: FileHandler writes logging messages to a file on the disk.
    * How: It sends the log messages to a specified file. This is useful for persistent logging, where logs are stored for later analysis.

### Example Python - Pandas: df.sort_values(kind=...)

```python
import pandas as pd
df = pd.DataFrame({'data': [3, 1, 4, 1]})
df.sort_values(by='data', kind='mergesort')
```

The kind parameter lets you choose a different strategy for sorting (like quicksort, heapsort, mergesort). 

## Template method (Behavioural)

The template method pattern leverages a class with a 'template method'
which does some kind of setup process using other of its methods which are 
meant to be overwritten for customization.

So simply inheriting from a class with some methods is not sufficient to talk about the Template method pattern, it's only when we rely on the 
core setup mechanism from that parent class that we talk about utilizing this pattern.


## Python Example - Django: CBV 

```python
from django.views.generic import TemplateView

class MyView(TemplateView):
    template_name = "my_template.html"

    def get_context_data(self, **kwargs):
        # Call the base implementation first to get a context
        context = super().get_context_data(**kwargs)
        # Add in custom context data to pass on to HTML template.
        context['page_title'] = 'My sugar addiction'
        return context

```

Inheriting MyView will also inherit the instantiation code of TemplateView (here the 'template method') which calls the `get_context_data`, here overwritten 
for customization (to pass on an additional variable to a Django served html template).

## Python Example - Pandas: Custom function with df.apply?

```python
import pandas as pd

df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})

# Define a custom function
def my_custom_function(x):
    return x * x

# Use apply to use the custom function across each column
result = df.apply(my_custom_function)

```

This is not a strict example of the Template method as we aren't subclassing
from a parent class or overwriting any of it's methods. But the broader similarity is that we are leveraging a class with method which expects
a function for customizing this process.


