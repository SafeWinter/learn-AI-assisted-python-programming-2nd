# 《Redis 4.x Cookbook》Learning Notes



## 1. Profiles

![Redis 4.x Cookbook](assets/cover.png)

|    **Title**    | **Redis 4.x Cookbook** [ISBN: 9781783988167] |
| :-------------: | :----------------------------------------------------------: |
|   **Author**    |               **Pengcheng Huang, Zuofei Wang**               |
| **Publication** |                      **Packt, 2018.2**                       |
|    **Pages**    |                           **374**                            |

> **Introduction**
>
> Redis is considered the world's most popular key-value store database. Its versatility and the wide variety of use cases it enables have made it a popular choice of database for many enterprises. Based on the latest version of Redis, this book provides both step-by-step recipes and relevant the background information required to utilize its features to the fullest. It covers everything from a basic understanding of Redis data types to advanced aspects of Redis high availability, clustering, administration, and troubleshooting. This book will be your great companion to master all aspects of Redis.
>
> The book starts off by installing and configuring Redis for you to get started with ease. Moving on, all the data types and features of Redis are introduced in detail. Next, you will learn how to develop applications with Redis in Java, Python, and the Spring Boot web framework. You will also learn replication tasks, which will help you to troubleshoot replication issues. Furthermore, you will learn the steps that need to be undertaken to ensure high availability on your cluster and during production deployment. Toward the end of the book, you will learn the topmost tasks that will help you to troubleshoot your ecosystem efficiently, along with extending Redis by using different modules.



## 2. Outlines

Status available：:heavy_check_mark: (Completed) | :hourglass_flowing_sand: (Working) | :no_entry: (Not Started) | :orange_book: (Finish reading)

| No.  |        Chapter Title        |          Status          |
| :--: | :-------------------------: | :----------------------: |
| Ch01 | [Chapter Title1](./Ch01.md) |    :heavy_check_mark:    |
| Ch02 | [Chapter Title2](./Ch02.md) | :hourglass_flowing_sand: |
| Ch03 | [Chapter Title3](./Ch03.md) |        :no_entry:        |



Powershell script for generating markdown files in batch:

```powershell
# Create 13 empty markdown files named Ch##.md:
for($i=1; $i -le 13; $i=$i+1){ New-Item -Name "Ch$('{0:d2}' -f $i).md"; }
```

 
