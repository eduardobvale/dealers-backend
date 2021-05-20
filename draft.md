## Context

Given the domain presented on the exercise it seems that we are modeling a system with one entity in mind: Company.
Our main concern remains in data synchronization with our main data source: fakeapi REST JSON endpoint.

# Design V1.0
Assumption: Dealers order of magnitude would be near 10^2 or 10^3 and would not grow too fast. 

We could start with a simple solution where we have a resource on our endpoint that is responsible for fetching the data source and returning a response containing all dealers. As the exercise requests, we need to store data locally and be careful with deleted items, since we need no specific feature of a NoSQL at this point we could start with a relation database and implement an upsert for fetched records. 

I would create the application based on Sinatra (since we dont need to deal with views, models, authentication, mailing) and use PostgreSQL for our RDB.

![image](https://user-images.githubusercontent.com/400858/118904221-cf53c400-b8ef-11eb-8ad1-b9baac8e5b80.png)

This would be our minimal architechture, in which I included a CDN (Cloudfront, Cloudflare) in front of our API and also our Asset storage (S3,GCS)

I've discarded Paas alternatives (eg Heroku) for the sake of exercising more Infrastructure concepts on this exercise.

V1.0 Is dead simple, but this simplicity increases our **Latency** time since we have to always check our source in each request, and we would also overload our database since every request would rewrite our database.

# Design V2.0

Assumption: New dealer onboarding should be a time consuming process, I think it is safe to assume that dealers are updated on a daily basis. But in the wrost case we could say that they would but updated hourly.

To solve our latency problem we'll introduce **CDN server side caching**, we're going to use a datetime expiration invalidation strategy, for example 1h expiry date for our dealers request. This is a very simple case for caching since we are always returning all dealers on our endpoint, caching complexity would increase when this schema and queries evolves.

This wouldn't change our architechture overall but would save us in **bandwidth** costs and reduce load on our backend.
![image](https://user-images.githubusercontent.com/400858/118905906-0bd4ef00-b8f3-11eb-8912-4eacc17d3886.png)

# Design V3.0

Now we have a fast and consistent endpoint, but we're not reliable nor scalable. First we would have to solve the most obvious **Singe point of Failure** that is having only one app instance (I'm aware we have others SPOFs but this touchpoint is more to failures). If we introduce a load balancer and a vm scaling group (minimal of 2) we could configure them to deal with instances failures and this would also enable us to scale up if we need more computing power.

![image](https://user-images.githubusercontent.com/400858/118907945-d16d5100-b8f6-11eb-8fb7-cb66af11a5c0.png)

There's another SPOF that is our Database single instance, we could introduce a cluster with reading replicas and failover&failback configurations, or even use a managed relational database service for easier configuration and maintenance (at a higher cost). But I think it's overkill for our scenario (max of 10^3 records) since we'd have a large cache window to replace this instance and manually fill cache.

# Disclaimers

Taking in consideration the requirementes of this exercise going for a NoSQL database seems arbitrary, but if we could assume that we will introduce a search input in the frontend app or dealer order of magnitude grows too much and demands a to fetch only visible dealers in the visible area of the map for example, then I would consider NoSQL alternatives (Discussed in the Alternatives section)

# Conclusion

For the proposed challenge (taking assumptions in consideration), a simple cache strategy seems to be best to architechture to start, avoiding over-engineering and keeping a low cost for resources and bandwidth.

# Alternatives

If we had more complex data sync scenarios I would suggest having a backgroun job queue (Sidekiq, Shoryuken) in a separate instance to handle synchronization (retriving records and upserting) but we would be consuming more computing services (EC2) for the workers instances and for the backend (Redis,SQS). In this situation I would also use some in memory cache between application and database to avoid load on database and reduce latency. For the presented scenario it seems overkill. 

For the database we didn't have any specific necessity like high-write throuput, geo spatial queries, schema flexibilty that's why I haven't explored any **NoSQL** options. But if we try to assume that at some point geo-spatial queries would be needed or the dealers schema could evolve rapidly. We could think of a **mongodb** (document based) wich has geo-spatial queries or even **ElasticSesrch** that also bring a strong search engine on top of the geo-spatial query feature.

## Frontend
[wip]
For the frontend, we would need a framework to render our API response in visual components and interact with a map. For that we'll use React and MateriaUI as our design system and for our map we could use Leaflet (a widely known js interactive map library).

