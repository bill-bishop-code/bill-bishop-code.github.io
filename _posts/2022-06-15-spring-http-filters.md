---
title: "HTTP Request Interceptors (Spring Boot Filters)"
date: 2022-06-15
categories:
  - Blog
tags:
  - Spring Filters
  - Interceptors
---
How many filters are "too many"?

Spring Boot Filters (HTTP Request Interceptors) seem like a good idea, and they can be.  It's a good place to perform
token authentication/authorization for example.  However, you should use them judiciously.  It can get out of hand
quickly.  When I saw a couple dozen interceptors in a recent project I had that reaction.  What's worse is that some
of the filters **depended** on other filters and therefore, specific order had to be maintained.

In my opinion you've gone too far if you see a situation like this.  Use them for auditing, logging, and other non-business
logic.  When you start seeing business logic show up filters, it's time to rethink.
