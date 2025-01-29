# **Building Spokane Tech: Part 7**

Welcome to part 7 of the "Building Spokane Tech" series! In this article, we'll explore integrating Celery for scheduling tasks, executing work asynchronously, and monitoring task statuses.


## **Django Tasks and Celery**

In a Django project, tasks.py is typically used to define background tasks that are executed asynchronously. This is common in applications where long-running or scheduled operations (such as sending emails, processing files, or performing API calls) should not block the main request-response cycle.

Celery is a distributed task queue that integrates well with Django to handle background jobs asynchronously. It allows you to schedule and execute tasks efficiently.


## **Creating Tasks**
In our service we'll use tasks defined in the tasks.py file to perform data ingestion from eventbrite and meetup. We have a few different tasks defined, namely a 'parent' launcher task, and a separate 'child' task for performing the bulk of the work. 

***Parent Tasks***

The parent task approach allows us to enable some basic control over what work gets queued for execution. For example, to run event ingestion for eventbrite groups we can first get a list of groups from our database, then launch individual jobs for each. Each job can succeed or fail independently of other jobs, and we'll have separate outputs and results. 

Here's an example of a parent task:
``` python
@shared_task(time_limit=900, max_retries=0, name="web.launch_eventbrite_event_ingestion")
def launch_eventbrite_event_ingestion():
    """parent task for ingesting future events for tech groups on Eventbrite"""
    tech_group_list = TechGroup.objects.filter(enabled=True, platform__name="Eventbrite")
    for group in tech_group_list:
        job = ingest_future_eventbrite_events.s(group.pk)
        job.apply_async()
```
In the code above, the ```tech_group_list``` is a queryset of groups that are enabled and on the eventbrite platform. The ```ingest_future_eventbrite_events``` is the task to be executed, and ```job.apply_async()``` is the code to put the job on the queue for a background worker to consume.


## **Scheduling Tasks**

***Django Celery Beat***

Django Celery Beat is an extension for Celery that enables scheduling periodic 
tasks in a cron-like manner using the Django Admin panel. It allows you to manage scheduled tasks dynamically without modifying code.

In our service we use this to automatically run the parent tasks at regularly scheduled intervals, for example, every eight hours.


## **Monitoring Tasks**

***Celery Flower***

Celery Flower is a real-time web-based monitoring tool for Celery. It provides insights into task execution, workers, queues, and performance. Key features include:

Dashboard Overview
- Displays task statistics (Succeeded, Failed, Pending, etc.)
- Shows real-time execution of Celery tasks

Workers Monitoring
- Lists all active workers
- Shows worker status, uptime, and queue load

Task Monitoring
- Provides details of each task (Arguments, Status, Runtime, etc.)
- Allows you to retry failed tasks

Queues & Broker Inspection
- Displays Celery queues and their task load
- Shows messages in Redis or RabbitMQ queues
