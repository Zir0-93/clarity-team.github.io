---
title:  "API Documentation Using Tabular Expressions"
date:   2017-07-20 15:04:23
categories: [api, REST, tabular, documentation]
tags: [api, REST, tabular, documentation]
excerpt_separator: <!--more-->
---
It is wellknown that APIs need developer-friendly docs in order
to gain widespread adoption. However, little if any improvement in the way REST APIs are presented to developers have been made over the 
past few years. Nowadays, most API docs and developer portals adopt a [Swagger UI](http://petstore.swagger.io/) type of format where all the endpoints in the specification
are simply listed, one after another. This format does a good job of communicating the low level details of each endpoint; However, 
as the complexity of APIs grows, developers need a way to get a high level understanding of the API **before** diving down into those
low level details.
<!--more--> 

![swagger example](/images/petstorev2.png)

The image above depicts Swagger UI documentation for a sample pet resource. As an API specification grows in complexity,
I find it increasingly difficult to answer the following questions using the aformentioned format.

1. **What are all the resources that this API exposes?**

This one is pretty explanatory, even with tags, an API specification with many endpoints will take a long time to navigate.

2. ** What operations are available on key resources? **

This questions exists for the same reason as the first question. API specifications with increasing complexity make
it difficult force developers to sift through longer than necessary lists of endpoints to figure out what operations are
avaialable for a given resource.

3. **What relationships exist in the ways these resources are consumed?**

Developer apps seldom depend on a single endpoint or API call from a third party API. Instead, such apps are built
by stringing together numerous API calls on the fly based on certain user scenarios. By helping developers understand
the requirements of and relations between the consumption of resources, you are helping them  
documentation:

### API Tables

![tabexpr](/images/tabexprv3.svg)

#### Scalability
Many APIs are not as simple as the one presented in the sample above. If you have worked with the APIs provided by
GitHub, Twitter or Facebook to name a few, you'll know that





Solution




Why Its Better




Conclusion