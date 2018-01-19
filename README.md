# Greetings, mate.

Not much to see here - just a repo with commits that document the steps I'm taking with my first foray in [Django](https://www.djangoproject.com/). This is an effort to familiarize myself with standard Django lingo and with the Django paradigm.

Read my commit history and the rest of this README to get a condensed understanding of the material that's covered in each part of [this Django tutorial](https://docs.djangoproject.com/en/2.0/intro/tutorial01/). My primary motivation for this repo is the desire to write about my learning process so that Future Me has a handy journal to re-read someday.

I *could* use my personal blog for this purpose, but I enjoy the challenge of keeping things focused, clear, and pithy - i.e., in the restrictive format of a commit message or a well-outlined README doc. Because learning is an iterative process, it makes sense to track it with Git. Per [these guidelines](https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project), I stick with the imperative present tense in the commit message header. As for the body of each commit, I'm still experimenting with writing quality messages that explain more *why* than *how*. For this reason, the messages themselves are less contained, more freewheeling and varied in tone and tense.

### Running tests & interacting with the Django test client
Run tests for the Polls app with `python manage.py test polls`.

Run `python manage.py shell` to invoke the Python shell and interact with Django's test [Client](https://docs.djangoproject.com/en/2.0/topics/testing/tools/#django.test.Client). (`manage.py` sets the `DJANGO_SETTINGS_MODULE` environment variable, which gives Django the Python import path to `mysite/settings.py`.)

Interacting with `django.test.Client` allows you to simulate user interactions with code at the view level. It can be used in `tests.py` as well as in the shell. However, our `tests.py` file already uses the `django.test.TestCase` class, which comes with its own client. (Therefore, there's no need to import the test client class there.)

In the shell, you'll need to set up the test environment. `setup_test_environment()` installs a template renderer that allows us to examine attributes on responses that otherwise wouldn't be available (e.g., `response.context`). Note that `setup_test_environment` **does not** set up a test database.

    >>> from django.test.utils import setup_test_environment
    >>> setup_test_environment

After setting up the test environment, import the test client class, create an instance of the client, and get to testing!

    >>> from django.test import Client
    >>> client = Client()

    # Get a response from '/'
    >>> response = client.get('/')
    Not Found: /

    >>> response.status_code
    404

    >>> from django.urls import reverse
    >>> response = client.get(reverse('polls:index'))
    >>> response.status_code
    200

    >>> response.content
    b'\n    <ul>\n    \n        <li><a href="/polls/5/">Are you pretty parched?</a></li>\n    \n        <li><a href="/polls/4/">Do you happen to have a bucket or a hose, bro?</a></li>\n    \n        <li><a href="/polls/3/">Do you have any plankton?</a></li>\n    \n        <li><a href="/polls/2/">Want a chip, bro?</a></li>\n    \n        <li><a href="/polls/1/">WHAT&#39;S UP, MATE?</a></li>\n    \n    </ul>\n\n'

    >>> response.context['latest_question_list']
    <QuerySet [<Question: Are you pretty parched?>, <Question: Do you happen to have a bucket or a hose, bro?>, <Question: Do you have any plankton?>, <Question: Want a chip, bro?>, <Question: WHAT'S UP, MATE?>]>

### The Django ORM
You can also interact with the object-relational mapper within the shell. (Sample questions and choices are inspired by [this](http://www.postkiwi.com/2008/beached-whale-in-new-zealand/).)

    # Import and make use of the Question and Choice models:
    >>> from polls.models import Question, Choice
    
    # Import other Django modules:
    >>> from django.utils import timezone
    
    # Create a new Question:
    >>> q = Question(question_text="WHAT'S UP, MATE?",
    pub_date=timezone.now())
    
    # Save the object to the database:
    >>> q.save()
    
    # Access the ID for an object:
    >>> q.id

    # Access model field values via Python attributes:
    >>> q.question_text
    >>> q.pub_date

    # Change values by resetting the attributes:
    # (Make sure to call save() explicitly!)
    >>> q.question_text = "WHAT'S NEW, MATE?"
    >>> q.save()

    # Return all class members:
    >>> Question.objects.all()
    >>> Choice.objects.all()
    
    # Take advantage of Django's database lookup API:
    >>> Question.objects.filter(id=1)
    >>> Question.objects.filter(question_text__startswith='WHAT')

    # Get the question that was published this year:
    >>> current_year = timezone.now().year
    >>> Question.objects.get(pub_date__year=current_year)

    # (`get()` expects to find and return only one object, whereas
    # `filter()` serves to return a QuerySet, i.e., a collection)

    # Get the question that matches the given ID:
    >>> Question.objects.get(id=1)

    # Get the [same] question by primary-key lookup:
    >>> q = Question.objects.get(pk=1)

    # Access custom methods:
    >>> q.was_published_recently()

    # Return a question's choice_set:
    >>> q.choice_set.all()
    >>> q.choice_set.count()

    # Call `create` on a question's choice_set:
    >>> q.choice_set.create(choice_text="I'M BEACHED AS!", votes=0)
    >>> q.choice_set.create(choice_text="I can't chew!", votes=0)
    >>> q.choice_set.create(choice_text="I need to get wet, ASAP!", votes=0)

    # (`create` constructs a new Choice object, does the INSERT statement,
    # adds the choice to the set of available choices, and returns the new
    # Choice object)

    # Access a Choice object's related Question object:
    # c = Choice.objects.get(pk=3)
    # c.question

    # Find all Choices for any question whose pub_date is in this year:
    # Choice.objects.filter(question__pub_date__year=current_year)

    # Delete a choice:
    # c.delete()

### The Django admin interface
* Create an admin user with the following command:
`python manage.py createsuperuser`

* Follow the prompts to create login credentials for that user.

* Start the development server with:
`python manage.py runserver`

* Go to `/admin/` in your browser - e.g., [http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/)

To make the app modifiable in the admin interface, update its admin file to include an import and registration statement for each model:

    # polls/admin.py
    from django.contrib import admin
    from .models import Question, Choice
   
    admin.site.register(Question)
    admin.site.register(Choice)
