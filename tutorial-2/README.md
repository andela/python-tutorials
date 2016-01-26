## Django Class-Based Views

**Expected Completion Time:** 60 mins

### Outline

- [What are Class-Based Views?](https://github.com/andela/python-tutorials/tree/master/tutorial-2#what-are-class-based-views)
- Use Case: ListView
- Use Case: TemplateView
- Use Case: FormView
- Use Case: DetailView
- Comparing CBVs and FBVs
- More material
- References

#### What are Class-Based Views?

Class-Based views are Django's attempt at making creation of views a whole lot more intuitive and less repetitive by leveraging on OOP.

A lot of times in Django or any other programming language the flow is usually one of creating a route and then creating a controller function which is called whenever that route is hit. This is true for most MVC frameworks and also true in Django. In the previous tutorials, we reference view functions that were defined in the views.py module and mapped the different routes to their respective view functions. What we find is that the process becomes very repetitive. Submitting form data, viewing a list of items from a query, viewing a single item, rendering a page etc. and we see that there is a lot of boiler plate code that we're writing to achieve these very repetitive task. OOP is well equipped to help with this and class-based views leverage on this to solve this problem.

> The first thing to note is that class-based views are constructed using classes. The advantage of this is that using the inheritance structure we are able to package all the reusable pieces into superclasses making it possible for us to write less code to achieve the same functionality.

#### Use Case: `ListView`

A typical example of when using class-based views could be advantageous is when we need to show a list of items on a certain page. A class based view exists to help us with this which is the `ListView` class contained in the `django.views.generic` module.

We would write the logic in our `views.py` like this

```python
from django.views.generic import ListView
from .models import SomeModel

class SomeViewClass(Listview):
  model = SomeModel
```

In our `urls.py` module we would write this line of code:

```python
from django.conf.urls import url
from .views import SomeViewClass

urlpatterns = [
  url(r'^some-route/$', SomeViewClass.as_view())
]
```

In this example, navigating to a route `/app/some-route/` will match against the route above. Although the view logic looks very thin, a lot of things happen here. Seeing as the model is already specified within the class, it goes ahead to run `SomeModel.objects.all()` for us. The query set is then stored within the context as `object_list`. What this means is that with this few lines of code we have queried our database, stored the result in a context variable and passed it to the template.

By default, this code looks for a template named `./templates/app/somemodel_list.html`. This behaviour can be overriden by specifying the `template_name` variable within the class like this:

```python
from django.views.generic import ListView
from .models import SomeModel

class SomeViewClass(Listview):
  model = SomeModel
  template_name = 'app/some_template.html'
```

In our template, we can easily write this:

```html
<div class="object-list">
  {% for object in object_list %}
  <p>{{ object.name }}</p>
  {% endfor %}
</div>
```

This can easily be applied to anything involving generating a long list of objects in a database.

Another very obvious advantage of using CBVs, as they are referred, is that, again, because of the inheritance structure, we can easily override any of the superclass method to specify our custom behaviour. An example would be if we wanted this to be used to list only the first five items in the database, we would modify `SomeViewClass` like this:

```python
class SomeViewClass(ListView):
  model = SomeModel
  template_name = 'app/some_template.html'

  def get_queryset(self, **kwargs):
    return SomeModel.objects.all()[:5]
```

This way, we have cut down the number of items that will display in our list.

We could also do this in an even simpler way.

```python
class SomeViewClass(ListView):
  model = SomeModel
  template_name = 'app/some_template.html'
  queryset = SomeModel.objects.all()[:5]
```

We can not only override functions, we can override variable and the `queryset` variable stores the queryset before passing it to the template.

If you don't like the variable `object_list` to store your queryset, you can change it using another class variable:

```python
class SomeViewClass(ListView):
  model = SomeModel
  template_name = 'app/some_template.html'
  queryset = SomeModel.objects.all()[:5]
  context_object_name = 'somemodel_objects'
```

Doing this, we have been able to change the name of the context variable containing our queryset to `somemodel_objects`.

As you can see from this example, we can do a number of things using the CBVs that can be advantageous when compared with the FBVs.

#### Use Case: `TemplateView`

Another typical example of where CBVs are used is when we want to display a single template. The `TemplateView` class is found in `django.views.generic` module as well. We would create the class this way:

```python
class SomeViewClass(TemplateView):
  template_name = 'app/some_template.html'
```

In the urls.py we would rewrite this route as:

```python
from django.conf.urls import url
from .views import SomeViewClass

urlpatterns = [
  url(r'^some-route/$', SomeViewClass.as_view())
]
```

This will load the specified template when we try to access the route `/app/some-route/`.

If we wanted to pass a variable to this template, we can do this by overriding the `get_context_data` function. See the example below:

```python
class SomeViewClass(TemplateView):
  template_name = 'app/some_template.html'

  def get_context_data(self, **kwargs):
    context = super(SomeViewClass, self).get_context_data(**kwargs)
    context.update({'some_variable': 'some_value'})
    return context
```

Doing this, we load the already existing context variable by calling the superclass `get_context_data` method. This returns to us the current state of the context. In the next line, we add in a new variable called `some_variable` and assign it a value `some_value`. We return the modified context as this method demands that we return some context data. The same process occurs as before i.e the specified template is loaded, but this time around we will be able to access the new context variable in our template.

Again, due to the inheritance structure of the CBVs, this function is also available in the `ListView` class that we discussed earlier.

#### Use Case: `FormView`
