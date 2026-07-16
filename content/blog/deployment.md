+++
title = 'A Simple Azure Deployment'
date = 2026-07-16T14:30:53+01:00
draft = false
+++

As part of a take-home assignment, a hypothetical team of data scientists asked me, as part of the data engineering team, to build Dockerised data pipeline in Python to collect daily meteorological data from the publicly available OpenWeather API (https://openweathermap.org/). The GitHub repository for the project can be be found [here](https://github.com/gleiria/data-engineering-pipeline). There you can find a detailed README describing the project's architecture, design decisions, implementation, and instructions for running it locally. This post focuses exclusively on the deployment to Microsoft Azure.

As part of my learning journey into cloud computing, and after completing the Azure AZ-900 and DP-900 certifications, I decided to get my hands dirty and deploy the pipeline to Microsoft Azure. In this post, I share the main steps I took to make that happen.

Since the application and all of its dependencies were already Dockerised, it was portable and ready to run on another machine. The objective was therefore to make it run on Microsoft's infrastructure instead of my local computer.

The first step, after creating an Azure account, was to set up an Azure Container Registry (ACR). A container registry is simply a centralised repository used to store and distribute container images. In practice, ACR plays the same role as Docker Hub but is Microsoft's private container registry, tightly integrated with the Azure ecosystem. It allows you to upload Docker images, control access to them, and make them available for deployment across Azure services.

With the Docker image stored in the cloud, the next question became: who actually runs it?

For that I used Azure Container Instances (ACI). This was actually a source of confusion for me because the term container instance can have more than one meaning. On one hand, it can simply refer to a running container. On the other hand, Azure Container Instances is an Azure service that allows you to execute containerised applications on demand without provisioning or managing virtual machines. During deployment you configure resources such as CPU, memory, restart policies, environment variables and storage mounts before Azure executes the container. I'll leave some of those details for another post.

At this point I had a service responsible for storing Docker images and another responsible for executing them. The remaining question was where to store the output produced by the pipeline.

Azure offers several storage services, and for this project I decided to use Azure Blob Storage. Blob Storage is Microsoft's object storage service and provides a simple, inexpensive and highly scalable way of storing unstructured data such as images, videos, logs and files. In my case, the pipeline stores the generated Parquet dataset directly in Blob Storage, making the data available outside of the container once execution has finished.

Another important aspect of the deployment was allowing the different services to communicate securely. The container needs access to the OpenWeather API to retrieve weather data and also needs permission to upload the resulting Parquet file to Azure Blob Storage. Rather than embedding secrets inside the code, API keys and Azure connection strings are injected into the container as environment variables at runtime. This keeps sensitive information separate from the application code while allowing the container to authenticate with the required services.

Finally, I wanted the pipeline to run automatically every day at 05:00 without any manual intervention. To achieve this, I used Azure Data Factory as the orchestration layer. Rather than processing data itself, Azure Data Factory is responsible for scheduling the execution of the Azure Container Instance on a daily basis.

To complete the deployment workflow, I also extended the project with a simple Continuous Deployment (CD) pipeline using GitHub Actions. Continuous Integration (CI) was already in place, automatically running the project's unit tests on every push. The new deployment stage builds a fresh Docker image whenever changes are merged into the main branch and pushes it to Azure Container Registry. As a result, the next scheduled execution of the pipeline automatically uses the latest version of the application without requiring any manual deployment steps.

{{< figure src="/images/cloud_diagram.png" title="The pipeline is deployed on Microsoft Azure using Docker containers. GitHub Actions automatically runs tests on every push and builds/pushes the Docker image to Azure Container Registry when changes are merged into the main branch. Azure Data Factory triggers the Azure Container Instance daily at 05:00, which executes the pipeline and stores the resulting Parquet dataset in Azure Blob Storage.">}}
