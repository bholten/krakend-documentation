---
aliases:
- /overview/introduction/
lastmod: 2018-10-20
date: 2016-10-25
linktitle: Introduction
menu:
  documentation:
    parent: getting started
next: /overview/quickstart
title: Introduction to KrakenD
weight: -10000
---

# What is KrakenD?
KrakenD is a high-performance open source API Gateway.

Its core functionality is to create an API that acts as an aggregator of many microservices into single endpoints, doing the heavy-lifting automatically for you: aggregate, transform, filter, decode, throttle, auth and more.

KrakenD needs **no programming** as it offers a declarative way to create the endpoints. It is well structured and layered and open to extending its functionality using plug-and-play middleware developed by the community or in-house.

KrakenD focuses on being a pure API gateway,  not coupled to the HTTP transport layer and it has been in production in large Internet businesses in Europe since early 2017.

# Why an API Gateway?

When consumers of API content (especially in microservices) query backend services, the implementations suffer a lot of complexity and burden with the sizes and complexity of their microservices responses.

KrakenD is an **API Gateway** that sits between the client and all the source servers, adding a new layer that removes all the complexity to the clients, providing them only the information that the UI needs.

KrakenD acts as an **aggregator** of many sources into single endpoints and allows you to group, wrap, transform and shrink responses. Additionally, it supports a myriad of middleware and plugins that allow you to extend the functionality, such as adding OAuth authorization or security layers (SSL, certificates, HTTP Strict Transport Security, Clickjacking protection, HTTP Public Key Pinning, MIME-sniffing prevention, XSS protection).

KrakenD is written in [Go](https://golang.org/) with support for multiple platforms and is based on the [KrakenD framework](https://github.com/devopsfaith/krakend).

## A practical example
A mobile developer needs to construct a single front page that requires data from several calls to their backend services, e.g:

    1) api.store.server/products
    2) api.store.server/marketing-promos
    3) api.users.server/users/{id_user}
    4) api.users.server/shopping-cart/{id_user}

The screen is straightforward and needs to retrieve data from only 4 different sources, wait for the round trip, and then pick only a few fields from the responses. Instead of doing these calls, the mobile client could call a single endpoint to KrakenD:

    1) krakend.server/frontpage/{id_user}

So this is how it would look like:

![Gateway](/images/documentation/krakend-gateway.png)

By choosing this implementation, the mobile client isolated itself from the backend implementation. Whenever the backends change their contract, the API contract for the mobile client remains the same and the gateway is updated via a simple change of configuration.

At the same time, there is a difference in size between the amount of data generated by the backends and what is finally traveling to the client.