---
layout: post
title: "Task: Reverse Me (Easy)"
description: from VU Cyberthon 2023
# img: assets/img/12.jpg
date: 2023-03-26 10:14:00-0000
category: reverse
related_posts: false
---

We have the [binary](https://github.com/kallenosf/CTF_Writeups/blob/main/VU_Cyberthon_2023/binaries/Task-ReverseME) named `Task-ReverseMe`

Firstly, let's try to get some details about the [binary](https://github.com/kallenosf/CTF_Writeups/blob/main/VU_Cyberthon_2023/binaries/Task-ReverseME) using the `file` command:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/Reverse_easy1.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

It's an **ELF executable** (Executable and Linkable Format) which means it can be run in Unix-like operatng systems. Another important information is that it's **not stripped**, so we could search for interesting symbols.

We can try run it:

<div class="row mt-3">
    <div class="col-6 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/Reverse_easy2.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

It seems that it asks for login information. We probably need to find a username and a password to get the flag.

Let's search for through its symbols, using the `readelf` command, since it's not stripped. We filter our results to **FUNC**tion symbols:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/Reverse_easy3.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>
The functions 
- `succes` at address `0x11a9` and 
- `fail` at address `0x11c3` 

are probably the functions that are being called after the check of the credentials.

Let's disassemble the binary using `objdump`. In the main function we observe the following:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/Reverse_easy4.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/Reverse_easy5.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

We can see two `cmp` (compare) instructions and immediately afterwards a `jne` (jump if not equal) for each one of them. If the two compared registers have not the same value, we jump to the `fail` function.
Those two comparisons are probably for username and password.

We can see this more clearly in **Ghidra**:

<div class="row mt-3">
    <div class="col-6 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/Reverse_easy7.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

- The user input for *login* is saved in *local_20*
- The user input for *password* is saved in *local_1c*

Those values are compared with two local integer variables that are initialized at lines 15 and 16. Therefore, if we input `0x3f1` for the login and `0x62b` for the password we will be able to get the flag:

<div class="row mt-3">
    <div class="col-6 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/Reverse_easy6.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

**Note: 3f1(hex) = 1009(dec), 62b(hex) = 1579(dec)**

According to the task description the answer format was like *VU{user,password}*, so the flag is:
**`VU{1009,1579}`**