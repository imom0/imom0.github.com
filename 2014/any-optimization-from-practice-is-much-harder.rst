=============================================
Any Optimization from Practice Is Much Harder
=============================================

We adopted an orphan python web project this week, it was maintained by lots of developers once.The former maintainers haven't stopped the bad smell in the project.Now it has serious performance problems, opening a page will cost several seconds (usually 3~5s, how terrible), users complain about it, we have to face the problem and solve it.

My mentor noticed there were several *SELECT COUNT* s on the dashboard page.It was meaningless to make it such real-time::

    class Car(model):

        @classmethod
        def count(cls, category):
            pass  # select count from db


The optimization was just **caching the count result**::

    class Car(model):

        @classmethod
        @cache('car:{category}:count', expire=30)
        def count(cls, category):
            pass  # select count from db

I used this quick and not dirty way.Then users felt the speed improvement on the dashboard page.

But you entered every category list page, still slow.Mysql explain showed category query type is *all*, the worst possible type. The query contains two where conditions and an order. I tried **add index** to one of where condition, type became *range*.After I add indices to other fields, it became *ref*, which can be accepted.

At that time, average request cost was 1000ms.

Thanks to the app engine team, by adding *app_engine_profile=true* to every URL we can easily get the detail for profiling.I tried, *gzip* were called 60000 times and cost 400ms in production environment, but other projects did not show this abnormal gzip calls.With the hint of my menter, I looked into the application entry point.

When I totally commented the gzip middleware, request time declined from 120ms to 20ms in development environment.It was convinced that some hot spots related to gzip, then I noticed the application's return value::

    class Publisher(object):

        def __call__(self, env, start_response):
            body = 'string'
            return body # Iterable object.

Wow, every body knows it's an iterable object and **every article about WSGI will tell you not send the output one byte by one byte**, but the developers **still did this**.It caused gzip is applied to every single byte.Changing it to *return [body]* or *yield body* solved this problem.

This aftermoon, when I deployed the modified code to staging, the production environment raised a lot of exceptions caused by mako cache.I checked, the project saved mako cache to mfs and the two environments used the same mako cache.After solved this bug, the speed may be a little improvement, because mfs is a distributed network file system.

In the end, the project performance had a considerable improvement, but still a lot of work to do.
