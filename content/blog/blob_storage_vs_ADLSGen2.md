+++
title = 'Blob Storage vs Azure Data Lake Storage Gen2'
date = 2026-07-20T07:03:06+01:00
draft = false
+++

Recently, while deploying a [data engineering project](https://github.com/gleiria/data-engineering-pipeline) to Azure, I found myself asking a question: **when should I use Blob Storage and when should I use Azure Data Lake Storage Gen2 (ADLS Gen2)?** Here I share how things clicked for me.

## Blob Storage

Blob stands for **Binary Large Object**. A blob is simply a sequence of bytes. From the perspective of Azure Blob Storage, a JPEG image, a PDF, an MP4 video, a Parquet dataset, a CSV, or JSON file are all treated exactly the same: they are just blobs. Blob Storage is therefore Microsoft's object storage service. It provides a highly scalable and inexpensive way of storing files in the cloud without caring about their internal structure.

## ADLS Gen2

One of the biggest misconceptions I had was thinking that Azure Data Lake Storage Gen2 was a completely different storage service. It is not. ADLS Gen2 is **Blob Storage with the Hierarchical Namespace feature enabled**. The underlying storage engine is exactly the same. What changes is how Azure represents and manages the stored data.

## Flat namespace vs hierarchical namespace

This distinction finally made everything click for me. Imagine a blob with the following name:

```text
2026/sales/june/sports.parquet
```

In standard Blob Storage this entire path is simply a string. The slashes (`/`) are only part of the blob's name and folders displayed in the Azure Portal are essentially an illusion created to make human navigation easier. This has important implications.

Imagine you decide to rename the directory `2026` to `26`. In Blob Storage there isn't actually a directory called `2026`. So Azure has to copy every blob, create new blob names, and delete the old blobs. As you can imagine, for a storage account containing thousands or millions of files, this quickly becomes a very expensive operation. With ADLS Gen2, however, directories are real filesystem objects. Renaming a directory becomes a metadata operation rather than copying every file. The operation completes almost instantly.

## Why have both Blob Storage and ADLS Gen2?

If ADLS Gen2 is more capable, why would anyone still use Blob Storage? The answer is that not every workload needs a filesystem. Imagine a streaming platform storing millions of video files:

```text
movie1.mp4
movie2.mp4
movie3.mp4
```

The application simply uploads and retrieves objects. It is not constantly reorganising directories or moving files around. Blob Storage works perfectly for this type of workload.

Now contrast that with a modern data lake:

```text
bronze/
silver/
gold/

customers/
sales/
logs/
```

Every day new partitions are created, folders are reorganised and ETL pipelines manipulate directory structures. This is filesystem work, and ADLS Gen2 is specifically designed for these scenarios.

## Typical workloads

In general, standard Blob Storage is best suited for general application workloads such as storing web assets, images, videos, PDFs, backups, and static files. Basically where objects are simply uploaded and retrieved. ADLS Gen2, on the other hand, is built specifically for enterprise data lakes and analytical processing frameworks like Apache Spark, Azure Databricks, Azure Synapse, and Hadoop. These big data engines and large-scale ETL pipelines require a true file system and benefit a lot from the performance advantages of the hierarchical namespace.

## Choosing between Blob Storage and ADLS Gen2

### 1. How will the data be accessed?

Blob Storage exposes a REST API, while ADLS Gen2 supports both the Blob REST API and filesystem APIs. If your application simply stores and retrieves objects, Blob Storage is often sufficient. If analytical engines such as Spark or Databricks need efficient filesystem operations, ADLS Gen2 is the better choice.

### 2. What operations will be performed?

Azure Storage pricing isn't only based on how much data you store. It also depends on the operations you perform and the amount of data transferred. When evaluating a storage solution it is therefore useful to think not only about storage capacity, but also about how frequently files will be created, listed, renamed, read and deleted.

### 3. What security model do you need?

Both Blob Storage and ADLS Gen2 integrate with Azure Role-Based Access Control (RBAC). ADLS Gen2 goes one step further by supporting POSIX-style Access Control Lists (ACLs), similar to those found on Linux systems. This allows different folders within the same storage account to have different permissions, making it much easier to organise enterprise datasets across departments for example Finance, Marketing, Engineering.

## To Wrap Up

Blob Storage is essentially a cloud object store. You upload objects and retrieve objects. ADLS Gen2 uses the same storage engine but adds filesystem semantics through the Hierarchical Namespace feature. Same storage under the wood, different capabilities on top.
