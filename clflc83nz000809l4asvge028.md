---
title: "How I built domain hunter using OpenAI on a weekend"
datePublished: Thu Mar 23 2023 16:38:24 GMT+0000 (Coordinated Universal Time)
cuid: clflc83nz000809l4asvge028
slug: how-i-built-domain-hunter-using-openai-on-a-weekend
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1679589469000/bb5f67cb-639d-474f-8339-f14342830b08.png
tags: python, django, openai, chatgpt

---

## Introduction

When ChatGPT was released, everyone was in awe, and a few people built production on top of it. I was also in the same hype and created a product. It is called [domain hunter](https://domainhunterr.com) — A tool that generates domains that are available for registration using OpenAI. You give terms in the search bar and it generates the domains.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679586354166/ac35cfa5-a896-4d9c-abcf-8dfd962b43f1.gif align="center")

## System Design

### Tech Stack

1. Python
    
2. Django
    
3. Celery
    
4. Redis
    
5. PostgreSQL
    
6. TailwindCSS
    
7. jQuery
    

![Domain Hunter System Design](https://cdn.hashnode.com/res/hashnode/image/upload/v1679587812902/619e020c-e171-489e-9b5a-897d394595dc.png align="center")

As explained in the above diagram, the user enters the search terms, they will be put in a rabbitmq queue. They will be picked up by celery workers and call OpenAI for domain suggestions. At this point, we don't know whether all the domains suggested by OpenAI are available or not. So, we need to call domain APIs to get the availability and price.

There are other sets of workers which consume messages from rabbitmq and call domain APIs. Then they put that data in redis and the database. So, the REST API controller will be polling redis for new data. Whenever it is found, it will return to the client.

## Database Design

![Low Level Design of Domain Hunter](https://cdn.hashnode.com/res/hashnode/image/upload/v1679588571985/cd55d046-712a-4e2e-b713-ade76f9ed446.png align="center")

1. `core_searchterm` is the table that stores the terms and `max_tokens` to be used by OpenAI.
    
2. `core_apiprovider` contains the configuration of the API provider. It contains the base URL, token, etc...
    
3. `core_apicallaudit` contains other requests and responses of every API call. It has a foreign key to the API provider and search term.
    
4. `core_domainsuggestion` contains a foreign key to the search term and its available domain details.
    
5. `core_visitor_search_terms` and `users_user_search_terms` maintains the mapping of user/visitor to search terms.
    

## End Notes

Currently, it has a free and paid plan. I released it 3 weeks ago. So far I did not see a single user sign up. People are visiting the website, searching, and disappearing. It seems I did not find the market fit yet. I am still trying, let's see.