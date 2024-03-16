# Python - Java design patterns

Potential alternate name: 

Design patterns in third party libraries


General distinction:
Building using a pattern - In the creation of the 
Leveraging existing code using pattern / leveraging a pattern - 


## Approach

This  


What are some ways to group design patterns? (other than creational, behav..)
By type of problem they solve? 
Eg. caching and flyweight both for better performance (speed optim or memory opt)

Could create graph diagram (eg via mermaid).

## Factory (Creational)

The factory pattern is leveraged when you use a method that creates a new 
instance each time you call it. 

Vs e.g. Singleton, where if you call this method multiple times, it 
will point to the same instance.


### Python Example - Django: objects.filter() returning new QuerySet instance

The `MyMode.objects.filter` method returns a new instance of a QuerySet each time you call it. 

```
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

```
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

```
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


## Flyweight (Structural)

### Python Example - Python: quoted strings

```
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

## Strategy (Behavioural)

The strategy pattern is about encapsulating or abstracting different strategies of execution and utilization of  attributes; each which may result in different outputs.

These strategy abstractions are then to be uniformly 'plugged in' or selected from an interface as alternatives.


### Example Python - logger.addHandler()

```
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

```
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

```
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

```
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


