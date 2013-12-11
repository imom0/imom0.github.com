=====================================
Dark Cornors of Django's Auto_now_add
=====================================

*created at: 2013-08-25 03:41*


There is a model declaration in one of our existing projects::

    class Event(models.Model):
        name = models.CharField(max_length=20)
        created_at = models.DateTimeField(auto_now_add=True)

We use *auto_now_add* for specifying the created time automatically.When the Event model is used more and more, sometimes we have scenarios that *created_at* is not equal to the created time of model instance::

    >>> event = Event(name='new event')
    # today is 2013-08-24
    >>> event.created_at = datetime.datetime(2013, 8, 10)
    >>> event.save()
    >>> event.created_at
    datetime.datetime(2013, 8, 24, 14, 10, 11, 181193)

It's not expected, but the django docs explain the `rule <https://docs.djangoproject.com/en/1.5/ref/models/fields/#django.db.models.DateField.auto_now_add>`_
:

> Note that the current date is always used; it’s not just a default value that you can override.

When you want set the datetime to what you want, just save again::

    >>> event.created_at = datetime.datetime(2013, 8, 10)
    >>> event.save()

Then I add a field default value to replace *auto_now_add*::

    import datetime

    class Event(models.Model):
        name = models.CharField(max_length=20)
        created_at = models.DateTimeField(default=datetime.datetime.now)

But this is not completely same effect as *auto_now_add*, a class based CreateView of Event gets involved into a form validation error *created_at is needed*.To solve this problem, add *blank=True* to *created_at* field is working but not good enough.The best way is to create a ModelForm with only fields related::

    from django import forms

    class EventForm(forms.ModelForm):
        class Meta:
            model = Event
            fields = ('name',)

    class EventCreateView(CreateView):
        model_form = EventForm

The conclusion I made from this is that using Form to handle user's posted form data is always a good practice even the Form looks like having done nothing.
