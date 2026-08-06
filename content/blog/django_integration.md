+++
title = 'Django + Data Pipeline Deployment'
date = 2026-07-22T14:39:50+01:00
draft = false
+++


## Goal of the project

In my previous article I described how I built and deployed a Dockerised data pipeline to Microsoft Azure. Every day, Azure Data Factory orchestrates the execution of a container that collects weather data from the OpenWeather API and stores it as a Parquet file in Azure Blob Storage.

The next step was to build a lightweight Django application that reads the latest dataset directly from Blob Storage and presents it through an interactive Plotly dashboard.

This article is not intended to be a deployment tutorial, but rather a summary of the concepts I had to understand to deploy a Django web application.



## The four deployment concerns

Apart from other bits and bobs, I think a Django deployment has four main concerns:

1) Static files
2) Database
3) WSGI server
4) Web server


## 1. Static files

We do not want Django to serve static files. Instead, the web server should handle them directly. Django facilitates this by collecting all static files into a single directory (STATIC_ROOT in settings.py), making it easy for the web server to locate and serve them. In templates, the {% static %} tag generates the correct URLs for these files.
Django web applications produce two types of content: dynamic and static. Dynamic content comes from a data source (in my case, a Parquet file that updates daily and feeds my Plotly dashboard). Static content includes files that never change, such as images, CSS (bootstrap.css, styles.css), or icons (logo.png, favicon.icon). Thus, there is absolutely no reason for Django to run Python code to serve static content. That would be wasteful. Instead, this is what happens:



## 2. Database

In this project I do not use a database. Instead, a data producer collects meteriological data from OpenWeatherAPI, sends data to Azure Blob Storage, and Django consumes it from there. Still, I leave this very simple high level paragraph for my own notes. The main idea is that in development environment Django provides a development database for us: SQLIte. Now, in production, if you use a cloud provider (which you will most likely do) you will be remotelly connecting to a database that is managed for you. 



## WSGI server

This is what actually runs your Django app in production. It acts as a bridge between Django and the web server. Django alone cannot handle production traffic efficiently, so the WSGI server (like Gunicorn) receives HTTP requests from the web server, communicates with Django using Python via the WSGI standard (through wsgi.py), and returns HTTP responses back to the web server.

All I did was:

pip install gunicorn
Add this to my Dockerfile:

```
gunicorn web_app.wsgi:application --bind 0.0.0.0:8000
```

This runs Gunicorn to serve your Django app (from web_app/wsgi.py) on port 8000, accepting connections from any network.


## Webserver

The web server is the front door that receives HTTP requests from the browser. It serves static files (CSS, JS, images) directly and forwards dynamic requests to Gunicorn (WSGI server), which runs Django. The web server has nothing to do with Django. Its job is to handle HTTPS, compression, caching, serving images, and forwarding requests. Apache and Nginx are the two main options. Now, the point is, do we ever need to configure a web server ourselves? Well, it depends on the deployment choice. Most of us, most of the time these days, will deploy to the cloud. Discussing IaaS vs PaaS is beyond the scope of this post, so I will keep it simple: PaaS abstracts a lot away from you and of course costs a bit more $$$, but if you don't want to become a part-time system administrator, go for PaaS.


-----


## Azure APP Service

Nomenclature in Azure can be confusing. The hierarchy is:

```
Datacenter
│
├── Physical servers
│
├── App Service Plan
│
├── Web App
│
└── Actual Django application
```

The **App Service Plan** is the rented compute: CPU, RAM, pricing tier, region, and redundancy options for resilience. The **Web App** is the hosting environment for your application, not the application itself. It manages the Python runtime, environment variables, deployment source, custom domain, logs, etc. The actual **Django application** is your code and business logic, which Azure copies into the **Web App**. An **App Service Plan** can run multiple applications as long as resources allow. An internal business app, company website, and internal dashboard could all share the same compute.
Also, **App Service** is PaaS. Similar to Azure Container Instances, you don't manage Ubuntu, Linux updates, firewalls, Nginx installation, or virtual machines. The focus is on business logic and pushing to production.

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
This machine runs 24/7. Since this is not a critical system, I didn't select any redundancy/resilience options. Inside the machine, Azure starts my Docker container.

## What does 1 CPU and 1.75 GB RAM actually buy me?

The CPU is used whenever someone visits the website. When you visit gleiria-pipeline.com, Django must:

* Receive the request
* Authenticate with Blob Storage
* Download the Parquet file
* Load it into pandas
* Generate Plotly HTML
* Return HTML

All of this costs CPU. However, the app will have very little to almost no traffic, so I expect the CPU to be idle 99% of the time.


## Memory

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

**Total:** ~400–600 MB. Even if the Parquet file grows to 50 MB, there's still capacity to run other projects. I think CPU is the real limiting factor here, but again, the app will have almost no traffic as it's a demo. In the next article, I will explore what happens if suddenly 1000 users join the app at the same time.

---

## To Wrap up

Looking back, I probably spent far more time understanding all the deployment process than actually writing the code to make that happen. But that was exactly the point of this project. I wanted to move beyond running applications locally and understand what actually happens when software is deployed to the cloud. There is still plenty to learn and improve, but I now have a much better mental model of how all the pieces fit together. As always, I will keep documenting what I learn as I build my next project. 

Thanks for reading :rocket:.



