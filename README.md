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
