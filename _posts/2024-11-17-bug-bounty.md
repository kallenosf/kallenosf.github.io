---
layout: post
title: Race Condition Bug in the Most Popular Greek E-Commerce App
date: 2024-11-17 10:14:00-0400
description: I found a race-condition bug in a popular greek e-commerce app, that allowed me to redeem the <u>same gift-card twice</u>
tags: bug-bounty race-conditions
categories: web
giscus_comments: false
related_posts: true
thumbnail: assets/img/bug-bounty/race-conditions.png
toc:
  sidebar: left
---

**TL;DR :** I found a race-condition bug in a popular greek e-commerce app, that allowed me to redeem the <u>same gift-card twice</u>.
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/bug-bounty/race-conditions.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>


## Race conditions
- Back in October 2023, I was hunting bugs in a popular greek e-commerce app. Around that time, PortSwigger's Web Security Academy published new [content](https://portswigger.net/web-security/race-conditions?utm_source=chatgpt.com) on race condition vulnerabilities. This prompted me to explore the app for such issues.
- <u>Race conditions</u> usually occurr when websites handle requests **in parallel without adequate safeguards**, leading to "collisions" that result in unintended behavior.
- This type of vulnerability is similar to **business logic** flaws.
- In a race condition attack, an attacker sends **carefully timed** requests to **intentionally create collisions** and exploit the resulting unintended behavior for malicious purposes.

### Limit overrun race conditions
- The most common type of race condition allows an attacker to **bypass some kind of limit** imposed by the application's business logic.
- For instance, consider an e-shop where a customer wants to use a **one-time** discount code. The application might handle the process as follows:
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/bug-bounty/race-conditions1.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

- Now consider what would happen if a user who has never applied this discount code before tried to apply it **twice at almost exactly the same time**:
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/bug-bounty/race-conditions2.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>
<div class="caption">
    Images from Portswigger's web-security Academy
</div>

- The application enters a **temporary sub-state** during its operation.
- This sub-state **starts** when the server <u>begins</u> processing the <u>initial request</u> and **ends** once the <u>database is updated</u> to reflect that the discount code has been used.
- During this **brief window, a race condition occurs**, allowing users to repeatedly claim the discount multiple times.
- There are many variations of this kind of attack, including:
  - Redeeming a gift card multiple times (😉)
  - Rating a product multiple times
  - Withdrawing or transferring cash in excess of your account balance
  - Reusing a single CAPTCHA solution
  - Bypassing an anti-brute-force rate limit

## Testing for race condition vulnerabilities

### Understanding the gift card functionality
- The app allows users to **purchase gift cards**, so I bought one to test for any potential limit overruns.
- I noticed that the gift card was **not linked to any specific account**, likely to allow it to be gifted to anyone.
- I created **three different accounts** and confirmed that all of them could apply the gift card code during checkout, though not simultaneously.

### Discovering race conditions
- During checkout, when a user added a coupon, a `POST` request whas being sent to the `/coupons/claim.json` endpoint, containing the code in a JSON, such as `{"code":"XXXXXXX"}`.
- I sent this request to **Burp Repeater**, for each session from all three accounts (I used three different browsers to create **separate sessions**, one for each account - although this could also be done in the same browser using a browser extension).
- Burp Repeater allows you to create a **tab group** and add **multiple requests** together.
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/bug-bounty/race-conditions3.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

- Once I added all three requests to the group tab, I sent them in **parallel**.
- Sending requests in parallel requires <u>Burp Suite 2023.9 or later!</u>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/bug-bounty/race-conditions4.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/bug-bounty/race-conditions5.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

- By observing the responses, I discovered that the coupon was applied to **more than one** account!
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/bug-bounty/race-conditions6.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/bug-bounty/race-conditions7.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

- While I couldn’t get the coupon to apply to all three accounts simultaneously, 90% of the time, it was **successfully** applied to two of the three.
- I did not attempt to complete a purchase, but I verified that you can proceed through the entire checkout process up to the payment step. Therefore, **this vulnerability is exploitable**.
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/bug-bounty/race-conditions8.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

- The maximum amount for a gift card that can be purchased is **€150**, so an attacker could pay €150 and use it to buy **€300** worth of items—repeating this process an unlimited number of times.

### Reporting
- This app has a bug bounty program in [BugCrowd](https://www.bugcrowd.com/), so I reported the issue there.
- At first, they couldn’t reproduce the issue and requested some PoC videos, which I provided.
- The bug was classified as a `P3` and I was awarded `$650`.
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/bug-bounty/race-conditions9.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

- This was my second finding in the same app, as I had previously discovered a **business logic vulnerability** that was classified as `P4`.
