---
layout: post
title: Creating a URL Shortener Part 0 - Introduction
---

I've wanted to implement a full backend system for a while now, with full
control over the end-to-end architecture on a greenfield project. 
I haven't had a relatively simple idea to work with until I stumbled upon 
[John Crickett's URL Shortener Challenge](https://codingchallenges.substack.com/p/coding-challenge-12-build-your-own).

I think this is a great idea to implement because it encompasses:

- A small and clear API.
- Scalability concerns, especially globally, and associated costs.
- Performance and availability (users expect a URL shortener to be fast and always available).
- Authentication and authorisation (users own their own shortened URLs).

I'll largely cover the architectural- and product-level decisions rather than the actual code, although
that will be open-sourced. Where possible, I'll explain the trade-offs with each decision. My hope is that
this won't just be a series about how to design this specific thing, but a general example of how you could
design more complex things.
