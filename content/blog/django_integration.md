+++
title = 'Django_integration'
date = 2026-07-22T14:39:50+01:00
draft = true
+++

---------------------------



## Goal of the project

In my previous article I described how I built and deployed a Dockerised data pipeline to Microsoft Azure. Every day, Azure Data Factory orchestrates the execution of a container that collects weather data from the OpenWeather API and stores it as a Parquet file in Azure Blob Storage.

The next step was to build a lightweight Django application that reads the latest dataset directly from Blob Storage and presents it through an interactive Plotly dashboard.

This article is not intended to be a deployment tutorial, but rather a summary of the concepts I had to understand to deploy a Django web application.

## Development vs Production


I think the very first thing I did was to add Azure Blob Storage credentials to a .env file so that Django settings.py could access them.

Created a .env file and added:

```
AZURE_CONTAINER=weather-data
AZURE_ACCOUNT_NAME=blobforweatherapp
AZURE_ACCOUNT_KEY=myAPIkey
```
Important to mention that these are not supposed to be sent to GitHub, particularly the API access key, so this .env file should be added to .gitignore.

I think then settings.py accesses these (only in development, not production). The way I did this was to install django-environ. This is a package that allows you to use the Twelve-factor methodology to configure your Django application with environmental variables (don't really get it).


## The four deployment concerns

Apart from a couple of other bits and bobs, a Django deployment has four main concerns:

Static files
Database
WSGI server
Web server

I think I will skip database here as my project does not really have one.

## Static files

If I understood correctly, we do not want Django to deal with static files. The general idea is that we want the web server to deal with them.

How does Django do that?

It puts all static files in one directory (STATIC_ROOT in settings.py), which makes it easy for the web server to find the static files it needs to serve.

I only have one static file I believe! The architecture diagram (this is probably wrong. What about the Bootstrap files for example?).

Then we use the static tag in the template to tell Django to pass the request to the web server.

Django web applications produce two types of responses: dynamic content and static content.

Dynamic content normally comes from database data that are dynamically served to templates. In my case I do not have a database but I have a Parquet file that changes every day and feeds my dashboard (Plotly graph), so this is dynamic too.

Static content relates to things that never change (images, bootstrap.css, logo.png, favicon.ico, styles.css, etc.).

There is absolutely no reason for Django to run Python code to serve static content. That would be wasteful. Instead:

```
Browser
      │
      ▼
Nginx
      │
      ├── image?
      │      return it immediately
      │
      └── Django page?
             pass to Gunicorn
```

STATIC_ROOT

In a toy development scenario we may have:

```
your_app/
    static/
        images/
            some_image_you_want_to_render.png
```

This works perfectly fine. However, in a real world project we may have:

```
app1/static/

app2/static/

django admin/static/

third-party package/static/

etc...
```

The web server cannot search through 50 folders so Django collects everything into one directory. We use this with ```manage.py collectstatic```. All this does is to collect all static files in the project and place them in the STATIC_ROOT directory. This must be done whenever we change or add static files

Once that is done, Nginx only serves STATIC_ROOT. That is it. When we are developing locally Django handles this because manage.py runserver is the only server available. Once we deploy, the production webserver needs to handle this.



## Database

In this project I simply have a data producer that sends data to Azure Blob Storage and a consumer, Django, that consumes the data. Still, I still leave this very simple high level paragraph for my own notes.

The main idea is that in development environment Django provides a dev database for us: SQLIte. Now, in production, if you use a cloud provider (which you will most likely do) you will be remotelly connecting to a database that is managed for you. 


## WSGI server

this is the thing that actually run your app in deployment. It makes the bridge between Django and the web. Django alone cannot handle production traffic efficiently so the WSGI server receives HTTP requests from the web via the web server, talks to Django using python via the WSGI standard (wsgi.py?) and returns HTTP responses to the web server. 

All I did, I think, what to pip install gunicorn and add this to my Dockerfile:

```
gunicorn web_app.wsgi:application --bind 0.0.0.0:8000
```

Need to check but I think this runs Gunicorn to serve your Django app (from web_app/wsgi.py) on port 8000, accepting connections from any network.


## Webserver

The web server (like Nginx or Apache) is the front door that receives HTTP requests from browsers. It serves static files (CSS, JS, images) directly and forwards dynamic requests to Gunicorn (your WSGI server), which runs Django.

The web server has nothing to do with Django. Its job is to handle HTTPS, compression, caching, serving images, forwarding requests. Apache and Nginx are the two main options. 

But!!!!

Do you ever need to configure a web server yourself? Well, it depends on your deployment choice. I would say most of us, most of the time these days, will deploy to the cloud. 

To discuss the differences between IaaS vs PaaS is behond the scope of this post so I will stick to this simplication: PaaS abstracts a lot away from you and off course costs a bit more $$$ but if you do not want to become a part-time system administrator go for PaaS. 

-----

## Docker

no notes yet


## Azure APP Service

Also numenclature is very confusing in Azure has we have the App Service, the webapp, and our own Django web application.. It basically goes like this:

Datacenter
│
├── Physical servers
│
├── App Service Plan
│
├── Web App
│
└── Your Django application

The App Service Plan is basically rented compute. The machine(s) your applications are allowed to use. We must define CPU, RAM, princing tier, region, reduncancy for resilience (correct term?)

**Web App**
This is what got me a bit confused because is the hosting environment for our application and not our application itself. The Web App knows things like Python runtime, environmental variables, deployment source, custom domain, logs, etc...

**Your Django application**
This is our code and business logic and Azure copies it into the Web App

This might be obvious but it is worth to mention that an App Service Plan can run multiple applications given that there are enough resources. An internal business app, a company website, and an internal dashboard could all share the same computation.

----
Remember this is a PaaS!
Similarly to Azure Container Instances, when using APP Services we do not manage Ubunto, linux updates, firewals, Nginx installation, virtual machine, etc...
The focus is to work on business logic and push it to production.

Other options I considered were Heroku, Python Anywhere and Fly.io but since the pipeline was already build in Azure I wanted to stay in the same ecosystem for this project. 

----

**Princing Tier**

I decided to go for B1 tier which gives me access to the following linux machine in the cloud:

```
CPU
1 virtual core

Memory
1.75 GB RAM

Disk
Persistent storage

Network
Public internet
```
This machine is on 24/7. This is not a critical system so I did not pick anmy redundancy resilience option. Inside the machine Azure starts my Docker container.


**What does 1 CPU and 1.75 GB RAM machine actually buy me?**

The CPU is used whenever someone visits the website. When you visit gleiria-pipeline.com Django has to
receive the request, authenticate with Blob Storage, download the Parquet, load it into pandas, generate Plotly HTML, return HTML, all of that costs CPU. The point is that the web app will have very from very little to almost no traffic and I expect the CPU to be doing nothing 99.9% of the time


Memory

Memory stores the running application. The container itself probably uses something like:

```
Python
~100 MB

Django
~100 MB

Gunicorn
~100 MB

Libraries
~100 MB
```

I would say 400–600 MB total. Even if the Parquet file becomes 50MB it is still ok so I will still have capacity to run other projects. I think CPU is really the limiting factor here but again, the app will have almost no traficc as it is a demo. In the next article I will explore the scenario where sudendly a 1000 users join your app at the same time.

## Deployment

No notes yet


## Lessons learned
no notes yet

Things I think were important for the deployment and I did not mention:

* Configure ALLOWED_HOSTS
* Debug mode 
* Add Azure Blob Storage environment variables in the Web App configuration.
* Current flow and the next step of updating CD (the manual process of build image, tag, push to registry everytime something changes)
* what is a load balancer
