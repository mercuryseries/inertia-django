![image](https://user-images.githubusercontent.com/6599653/114456558-032e2200-9bab-11eb-88bc-a19897f417ba.png)


# Inertia.js Django Adapter

## Installation

### Backend

Install the following python package via pip
```bash
pip install inertia-django
```

Add the Inertia app to your `INSTALLED_APPS` in `settings.py`
```python
INSTALLED_APPS = [
  # django apps,
  'inertia',
  # your project's apps,
]
```

Add the Inertia middleware to your `MIDDLEWARE` in `settings.py`
```python
MIDDLEWARE = [
  # django middleware,
  'inertia.middleware.InertiaMiddleware',
  # your project's middleware,
]
```

Finally, create a layout which exposes `{% block content %}{% endblock %}` in the body and set the path to this layout as `INERTIA_LAYOUT` in your `settings.py` file.

Now you're all set!

### Frontend

Django specific frontend docs coming soon. For now, we recommend installing [django_vite](https://github.com/MrBin99/django-vite) 
and following the commits on the Django Vite [example repo](https://github.com/MrBin99/django-vite-example). Once Vite is setup with
your frontend of choice, just replace the contents of `entry.js` with [this file (example in react)](https://github.com/BrandonShar/inertia-rails-template/blob/main/app/frontend/entrypoints/application.jsx)


You can also check out the official Inertia docs at https://inertiajs.com/. 

## Usage

### Responses

Render Inertia responses is simple, you can either use the provided inertia render function or, for the most common use case, the inertia decorator. The render function accepts four arguments, the first is your request object. The second is the name of the component you want to render from within your pages directory (without extension). The third argument is a dict of `props` that should be provided to your components. The final argument is `view_data`, for any variables you want to provide to your template, but this is much less common.

```python
from inertia import render
from .models import Event

def index(request):
  return render(request, 'Event/Index', props={
    'events': Event.objects.all()
  })
```

Or use the simpler decorator for the most common use cases

```python
from inertia import inertia
from .models import Event

@inertia('Event/Index')
def index(request):
  return {
    'events': Event.objects.all(),
  }
```

### Shared Data

If you have data that you want to be provided as a prop to every component (a common use-case is information about the authenticated user) you can use the `share` method. A common place to put this would be in some custom middleware.

```python
from inertia import share
from django.conf import settings
from .models import User

def inertia_share(get_response):
  def middleware(request):
    share(request, 
      app_name=settings.APP_NAME,
      user_count=lambda: User.objects.count(), # evaluated lazily at render time
      user=lambda: request.user, # evaluated lazily at render time
    )

    return get_response(request)
  return middleware
```

### Lazy Props
On the front end, Inertia supports the concept of "partial reloads" where only the props requested
are returned by the server. Sometimes, you may want to use this flow to avoid processing a particularly slow prop on the intial load. In this case, you can use `Lazy props`. Lazy props aren't evaluated unless they're specifically requested by name in a partial reload.

```python
from inertia import lazy, inertia

@inertia('ExampleComponent')
def example(request):
  return {
    'name': lambda: 'Brandon', # this will be rendered on the first load as usual
    'data': lazy(lambda: some_long_calculation()), # this will only be run when specifically requested by partial props and WILL NOT be included on the initial load
  }
```

### Json Encoding

Inertia Django ships with a custom JsonEncoder at `inertia.utils.InertiaJsonEncoder` that extends Django's 
`DjangoJSONEncoder` with additional logic to handle encoding models and Querysets. If you have other json 
encoding logic you'd prefer, you can set a new JsonEncoder via the settings.

### SSR 

#### Backend
Enable SSR via the `INERTIA_SSR_URL` and `INERTIA_SSR_ENABLED` settings

#### Frontend
Coming Soon!

## Settings

Inertia Rails has a few different settings options that can be set from within your project's `settings.py` file. Some of them have defaults.

The default config is shown below

```python
INERTIA_VERSION = '1.0' # defaults to '1.0'
INERTIA_LAYOUT = 'layout.html' # required and has no default
INERTIA_JSON_ENCODER = CustomJsonEncoder # defaults to inertia.utils.InertiaJsonEncoder
INERTIA_SSR_URL = 'http://localhost:13714' # defaults to http://localhost:13714
INERTIA_SSR_ENABLED = False # defaults to False
```

## Testing

Inertia Django ships with a custom TestCase to give you some nice helper methods and assertions.
To use it, just make sure your TestCase inherits from `InertiaTestCase`. `InertiaTestCase` inherits from Django's `django.test.TestCase` so it includes transaction support and a client.

```python
from inertia.test import InertiaTestCase

class ExampleTestCase(InertiaTestCase):
  def test_show_assertions(self):
    self.client.get('/events/')

    # check the component
    self.assertComponentUsed('Event/Index')
    
    # access the component name
    self.assertEqual(self.component(), 'Event/Index')
    
    # props (including shared props)
    self.assertHasExactProps({name: 'Brandon', sport: 'hockey'})
    self.assertIncludesProps({sport: 'hockey'})
    
    # access props
    self.assertEquals(self.props()['name'], 'Brandon')
    
    # view data
    self.assertHasExactViewData({name: 'Brian', sport: 'basketball'})
    self.assertIncludesViewData({sport: 'basketball'})
    
    # access view data 
    self.assertEquals(self.view_data()['name'], 'Brian')
```

The inertia test helper also includes a special `inertia` client that pre-sets the inertia headers
for you to simulate an inertia response. You can access and use it just like the normal client with commands like `self.inertia.get('/events/')`. When using the inertia client, inertia custom assertions **are not** enabled though, so only use it if you want to directly assert against the json response.

## Thank you

A huge thank you to the community members who have worked on InertiaJS for Django before us. Parts of this repo were particularly inspired by [Andres Vargas](https://github.com/zodman) and [Samuel Girardin](https://github.com/girardinsamuel). Additional thanks to Andres for the Pypi project.

*Maintained and sponsored by the team at [bellaWatt](https://bellawatt.com/)*

[![bellaWatt Logo](https://user-images.githubusercontent.com/6599653/114456832-5607d980-9bab-11eb-99c8-ab39867c384e.png)](https://bellawatt.com/)
