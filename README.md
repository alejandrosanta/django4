## Forms & Class Views
### 1. Forms
> app_name/forms.py
```python
from django import forms

class ReviewForm(forms.Form):
    user_name = forms.CharField(label="Your name", max_length=100, error_messages={
        "required": "Your name must not be empty",
        "max_lenght": "Please enter a shorter name"
    })
    review_text = forms.CharField(label="Your feedback", widget=forms.Textarea, max_length=200)
    rating = forms.IntegerField(label="Your rating", min_value=1, max_value=5)
```

> app_name/views.py
```python
from django.shortcuts import render
from django.http import HttpResponseRedirect
from .forms import ReviewForm

def review(request):
    if request.method == 'POST':
        form = ReviewForm(request.POST)
        if form.is_valid():
            print(form.cleaned_data)
            return HttpResponseRedirect("/thank-you")
    else:
        form = ReviewForm()
    
    form = ReviewForm()
    return render(request, "reviews/review.html", {
        "form":form
    })

def thank_you(request):
    return render(request, "reviews/thank_you.html")
```

> app_name/templates/app_name/review.html
```html
{% load static %}

<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Your Review</title>
        <link rel="stylesheet" href="{% static "reviews/styles.css" %}">
    </head>
    <body>
        <form action="/" method="POST">
            {% csrf_token %}
            {% for field in form %}
                <div class="form-control {% if field.errors %}errors{% endif %}">
                    {{ field.label_tag }}
                    {{ field }}
                    {{ field.errors }}
                </div>
            {% endfor %}
            <button type="submit">Send</button>
        </form>
    </body>
</html>
```

> app_name/templates/app_name/thank_you.html
```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Thank you</title>
    </head>
    <body>
        <h1>Thank you!</h1>
    </body>
</html>
```
> project_name/urls.py

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include("reviews.urls"))
]
```

> project_name/settings.py

```python
INSTALLED_APPS = [
    'reviews',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

### 2. Database
> app_name/models.py
```python
from django.db import models

class Reviwe(models.Model):
    user_name = models.CharField(max_length=100)
    review_text = models.TextField()
    rating = models.IntegerField()
```
> app_name/views.py
```python
from django.shortcuts import render
from django.http import HttpResponseRedirect

from .forms import ReviewForm
from .models import Review

def review(request):
    if request.method == 'POST':
        form = ReviewForm(request.POST)

        if form.is_valid():
            review = Review(
                user_name=form.cleaned_data['user_name'], 
                review_text=form.cleaned_data['review_text'], 
                rating=form.cleaned_data['rating'])
            review.save()
            return HttpResponseRedirect("/thank-you")
        
    else:
        form = ReviewForm()
        
    
    form = ReviewForm()
    return render(request, "reviews/review.html", {
        "form":form
    })

def thank_you(request):
    return render(request, "reviews/thank_you.html")
```

### 3. Modelforms

> app_name/forms.py
```python
from django import forms
from .models import Review

class ReviewForm(forms.ModelForm):
    class Meta:
        model = Review
        fields = '__all__'
        labels = {
            "user_name":"Your Name",
            "review_text":"Your Feedback",
            "rating":"Your Rating"
        }
        error_messages = {
            "user_name": {
                "required": "Your name must not be empty",
                "max_lenght": "Please enter a shorter name"
            }
        }
```

> app_name/views.py
```python
from django.shortcuts import render
from django.http import HttpResponseRedirect

from .forms import ReviewForm

def review(request):
    if request.method == 'POST':
        form = ReviewForm(request.POST)

        if form.is_valid():
            form.save()
            return HttpResponseRedirect("/thank-you")
        
    else:
        form = ReviewForm()
        
    form = ReviewForm()
    return render(request, "reviews/review.html", {
        "form":form
    })

def thank_you(request):
    return render(request, "reviews/thank_you.html")
```

### 4. Class Based Views

> app_name/urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path("",views.ReviewView.as_view()),
    path("thank-you",views.thank_you)
]
```
> app_name/views.py
```python

from django.shortcuts import render
from django.http import HttpResponseRedirect
from django.views import View

from .forms import ReviewForm

class ReviewView(View):
    def get(self, request):
        form = ReviewForm()
        return render(request, "reviews/review.html", {
            "form":form
        })
    def post(self, request):
        form = ReviewForm(request.POST)

        if form.is_valid():
            form.save()
            return HttpResponseRedirect("/thank-you")
        
        return render(request, "reviews/review.html", {
            "form":form
        })

def thank_you(request):
    return render(request, "reviews/thank_you.html")
```

### 5. Templates

> app_name/templates/app_name/base.html
```html
{% load static %}
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>
            {% block title %}
            {% endblock %}
        </title>
        <link rel="stylesheet" href="{% static "reviews/styles.css" %}">
    </head>
    <body>
        {% block content %}
        {% endblock %}    
    </body>
</html>
```

> app_name/templates/app_name/review.html
```html
{% extends "reviews/base.html" %}

{% block title %}
    Your Review
{% endblock %}
{% block content %}      
    <form action="/" method="POST">
        {% csrf_token %}
        {% for field in form %}
            <div class="form-control {% if field.errors %}errors{% endif %}">
                {{ field.label_tag }}
                {{ field }}
                {{ field.errors }}
            </div>
        {% endfor %}
        <button type="submit">Send</button>
    </form>
{% endblock %}
```

> app_name/templates/app_name/thank_you.html
```html
{% extends "reviews/base.html" %}

{% block title %}
    Thank You
{% endblock %}
{% block content %}
    <h1>Thank you!</h1>
{% endblock %}
```

### 6. TemplateView

> app_name/urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path("",views.ReviewView.as_view()),
    path("thank-you",views.ThankYouView.as_view())
]
```

> app_name/views.py
```python
from django.shortcuts import render
from django.http import HttpResponseRedirect
from django.views import View
from django.views.generic.base import TemplateView

from .forms import ReviewForm

class ReviewView(View):
    def get(self, request):
        form = ReviewForm()
        return render(request, "reviews/review.html", {
            "form":form
        })
    def post(self, request):
        form = ReviewForm(request.POST)

        if form.is_valid():
            form.save()
            return HttpResponseRedirect("/thank-you")
        
        return render(request, "reviews/review.html", {
            "form":form
        })

class ThankYouView(TemplateView):
    template_name = "reviews/thank_you.html"

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["message"] = "This works!!"
        return context
```

> app_name/templates/app_name/thank_you.html
```html
{% extends "reviews/base.html" %}

{% block title %}
    Thank You
{% endblock %}
{% block content %}
    <h1>Thank you!</h1>
    <p>{{ message }}</p>
{% endblock %}
```

### 7. List

> app_name/templates/app_name/review_list.html
```html
{% extends "reviews/base.html" %}

{% block title %} All Reviews {% endblock %}

{% block content %}
<ul>
    {% for review in reviews %}
        <li>
            {{ review.user_name }} (Rating: {{ review.rating }})
        </li>
    {% endfor %}
</ul>
{% endblock %}
```

> app_name/views.py
```python
from .models import Review

class ReviewListView(TemplateView):
    template_name = "reviews/review_list.html"

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        reviews = Review.objects.all()
        context["reviews"] = reviews
        return context
```

> app_name/urls.py
```python
    path("reviews",views.ReviewListView.as_view())

```
### 8. ListView

> app_name/views.py
```python
from django.views.generic import ListView

class ReviewListView(ListView):
    template_name = "reviews/review_list.html"
    model = Review
    context_object_name = "reviews"
```

### 9. Detail

> app_name/urls.py
```python
    path("review/<int:id>",views.SingleReviewView.as_view()),   
```

> app_name/views.py
```python
class SingleReviewView(TemplateView):
    template_name = "reviews/single_review.html"

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        review_id = kwargs["id"]
        selected_review = Review.objects.get(pk=review_id)
        context["review"] = selected_review
        return context
```

> app_name/templates/app_name/single_review.html
```html
    {% extends "reviews/base.html" %}

    {% block title %} Review Details {% endblock %}

    {% block content %}
        <h1>{{ review.user_name }}</h1>
        <p>Rating:{{ review.rating }}</p>
        <p>{{ review.review_text }}</p>
    {% endblock %}
```

### 10. DetailView

> app_name/urls.py
```python
    path("review/<int:pk>",views.SingleReviewView.as_view()),   
```
> app_name/views.py
```python
    from django.views.generic import DetailView

    class SingleReviewView(DetailView):
        template_name = "reviews/single_review.html"
        model = Review
```

### 11. CreateView

> app_name/views.py

```python
    from django.views.generic.edit import CreateView

    class ReviewView(CreateView):
        model = Review
        form_class = ReviewForm
        template_name = "reviews/review.html"
        success_url = "/thank-you"
```

## File Uploads
### 1. First Version

> app_name/views.py

```python
    from django.shortcuts import render
    from django.views import View
    from django.http import HttpResponseRedirect

    def store_file(file):
        with open("temp/image.jpg", "wb+") as dest:
            for chunk in file.chunks():
                dest.write(chunk)

    class CreateProfileView(View):
        def get(self, request):
            return render(request, "profiles/create_profile.html")

        def post(self, request):
            store_file(request.FILES["image"])
            return HttpResponseRedirect("/profiles")
```

> app_name/templates/app_name/create_profile.html

```html
    {% load static %}

    <!DOCTYPE html>
    <html lang="en">
    <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Create a Profile</title>
    <link rel="stylesheet" href="{% static "profiles/styles.css" %}">
    </head>
    <body>
    <form action="/profiles/" method="POST" enctype="multipart/form-data">
        {% csrf_token %}
        <input type="file" name="image" />
        <button>Upload!</button>
    </form>
    </body>
    </html>
```

### 2. Adding a Form with a Filefield

> app_name/forms.py

```python
    from django import forms

    class ProfileForm(forms.Form):
        user_image = forms.FileField()
```

> app_name/templates/app_name/create_profile.html

```html
  <form action="/profiles/" method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form }}
    <button>Upload!</button>
  </form>
```

> app_name/views.py

```python
    from .forms import ProfileForm

    def store_file(file):
        with open("temp/image.jpg", "wb+") as dest:
            for chunk in file.chunks():
                dest.write(chunk)

    class CreateProfileView(View):
        def get(self, request):
            form = ProfileForm()
            return render(request, "profiles/create_profile.html", {
                "form":form
            })

        def post(self, request):
            submitted_form = ProfileForm(request.POST, request.FILES)
            
            if submitted_form.is_valid():
                store_file(request.FILES["image"])
                return HttpResponseRedirect("/profiles")
            
            return render(request, "profiles/create_profile.html", {
                "form":submitted_form
            })

```

### 3. Using Models for File Storage

> project_name/settings.py

```python
    MEDIA_ROOT = BASE_DIR / "uploads"
```

> app_name/models.py

```python
from django.db import models

class UserProfile(models.Model):
    image = models.FileField(upload_to="images")
```

> app_name/views.py

```python
from .models import UserProfile

if submitted_form.is_valid():
    profile = UserProfile(image=request.FILES["user_image"])
    profile.save()
    return HttpResponseRedirect("/profiles")
```

### 4. Using an Imagefield
> $python3 -m pip install Pillow

> app_name/models.py
```python
    image = models.ImageField(upload_to="images")
```

> app_name/forms.py

```python
    user_image = forms.ImageField()
```