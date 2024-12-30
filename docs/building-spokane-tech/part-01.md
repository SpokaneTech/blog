# **Building Spokane Tech: Part 1**

Welcome to the first part of the "Building Spokane Tech" series! In this article, we explore the tech stack, design decisions, and how to run the site locally on your system.

## **Requirements**
For the first phase of our project we want to identify all the tech related community groups in the Spokane area, gather data about them and ingest and present events they host in one location. To make this happen we'll need a couple things. 
- web interface for displaying groups and events
- a database to store the groups, event, and associated information
- code that can gather data from applicable event sites
- a means to execute that code on a regular cadence 


## **Tech Stack**
Our tech stack will be comprised of the follow technologies (accompanied with a brief description of each):

### **Python**
**Role:** *Primary programming language*

**Responsibility:** *Powers the application backend, providing a robust, readable, and flexible foundation for building web functionality and handling logic.*

### **Django**
**Role:** *Web framework*

**Responsibility:** *Facilitates rapid development of secure and maintainable websites, handling URL routing, views, models, forms, and authentication. It integrates well with databases and supports REST API development.*

### **Gunicorn**

**Role:** *WSGI HTTP server*

**Responsibility:** *Serves as the bridge between your Django application and the web server (e.g., Nginx). It efficiently handles multiple requests concurrently and scales well for production.*

### **Redis**
**Role:** *In-memory data store*

**Responsibility:** *Used as a message broker for Celery tasks, caching, and real-time features like notifications or session management.*

### **Postgres**
**Role:** *Database*

**Responsibility:** *Provides a reliable, scalable, and feature-rich relational database for storing application data, such as user information, product records, and transaction logs.*

### **Celery**
**Role:** *Task queue*

**Responsibility:** *Manages asynchronous tasks (e.g., sending emails, processing files) by offloading time-consuming operations to background workers, improving responsiveness.*

### **Celery Beat**
**Role:** *Scheduler for Celery tasks*
Responsibility: *Executes periodic tasks by scheduling them at specific intervals (e.g., daily reports or regular database cleanup).*

### **HTMX**
**Role:** *Frontend interaction library*

**Responsibility:** *Enhances user experience by enabling server-side rendered dynamic content updates without full page reloads. Simplifies AJAX requests, WebSockets, and DOM updates.*

### **Bootstrap 5**
**Role:** *CSS framework*

**Responsibility:** *Simplifies frontend design with a responsive, mobile-first grid system and pre-designed components such as buttons, modals, and navigation bars. Speeds up development and ensures a consistent, modern UI.*
