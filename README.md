### 1. Forms
> app_name/forms.py
```python
from django import forms

class ReviewForm(forms.Form):
    user_name = forms.CharField(label="Your name", max_length=100, error_messages={
        "required": "Your name must not be empty",
        "max_lenght": "Please enter a shorter name"
    })
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
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Your Review</title>
    </head>
    <body>
        <form action="/" method="POST">
            {% csrf_token %}
            <div class="form-control {% if form.user_name.errors %}errors{% endif %}">
                {{ form.user_name.label_tag }}
                {{ form.user_name }}
                {{ form.user_name.errors }}
            </div>
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