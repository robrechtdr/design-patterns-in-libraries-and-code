# Python - Java design patterns

Potential alternate name: 

Design patterns in third party libraries


General distinction:
Building using a pattern - In the creation of the 
Leveraging existing code using pattern / leveraging a pattern - 


## Approach

This  


## Factory (Creational)

The factory pattern 



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


