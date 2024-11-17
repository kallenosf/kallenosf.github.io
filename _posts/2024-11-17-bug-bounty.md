---
layout: post
title: Race condition bug in the most popular Greek e-commerce app
date: 2024-11-17 10:14:00-0400
description: an example of a blog post with table of contents on a sidebar
tags: bug-bounty race-condition
categories: web
giscus_comments: false
related_posts: false
toc:
  sidebar: left
---

**TL;DR :** I found a race-condition bug in a popular greek e-commerce app, that allowed me to redeem the <u>same gift-card twice</u>.

## Race conditions

- Back in September 2023, Portswigger published [content](https://portswigger.net/web-security/race-conditions) in their web-security academy about race condition vulnerabilities.
- <u>Race conditions</u> usually occurr when websites handle requests **in parallel without adequate safeguards**, leading to "collisions" that result in unintended behavior.
- This type of vulnerability is similar to **business logic** flaws.
- In a race condition attack, an attacker sends **carefully timed** requests to **intentionally create collisions** and exploit the resulting unintended behavior for malicious purposes.

### Limit overrun race conditions

- The most common type of race condition allows an attacker to **bypass some kind of limit** imposed by the application's business logic.
- For instance, consider an e-shop where a customer wants to use a **one-time** discount code. The application might handle the process as follows:
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/race-conditions1.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

- Now consider what would happen if a user who has never applied this discount code before tried to apply it **twice at almost exactly the same time**:
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/race-conditions2.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Images from Portswigger web-security Academy
</div>

- The application transitions through a temporary sub-state.
- This sub-state begins when the server starts processing the first request, and ends when it updates the database to indicate that a discount code is used. 
- This introduces a small race window during which you can repeatedly claim the discount as many times as you like.
- There are many variations of this kind of attack, including:
  - Redeeming a gift card multiple times (😉)
  - Rating a product multiple times
  - Withdrawing or transferring cash in excess of your account balance
  - Reusing a single CAPTCHA solution
  - Bypassing an anti-brute-force rate limit

## Testing for race condition vulnerabilities

