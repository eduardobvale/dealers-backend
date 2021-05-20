## Context

Given the domain presented on the exercise it seems that we are modeling a system with one entity in mind: Company.
Our main concern remains in data synchronization with our main data source: fakeapi REST JSON endpoint.

There are some integration patterns we could apply to achieve the desired behaviour, but I'll explore some scenarios so we can
decide which design to implement.

We could start with a simple solution where we have a resource on our endpoint that is responsible for fetching the data source and returning a response containing all dealers. As the exercise requests, we need to store data locally and be careful with deleted items, since we need no specific feature of a NoSQL at this point we could start with a relation database and implement an upsert for fetched records. 








For the frontend, we would need a framework to render our API response in visual components and interact with a map. For that we'll use React and MateriaUI as our design system and for our map we could use Leaflet (a widely known js interactive map library).
