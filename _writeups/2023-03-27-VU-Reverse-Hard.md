---
layout: post
title: "Task: Reverse Me (Hard)"
description: from VU Cyberthon 2023
# img: assets/img/12.jpg
date: 2023-03-27 10:14:00-0000
category: reverse
related_posts: false
---

This challenge was very similar with the [Reverse Me (Easy) task](/writeups/VU-Reverse-Easy/) and can be solved in the same way, even though the easy one was awarded with 50pts while the hard one with 250pts (🤔). Perhaps the biggest difference was that this [binary](https://github.com/kallenosf/CTF_Writeups/blob/main/VU_Cyberthon_2023/binaries/Task-ReverseME-hard) (Task-ReverseME-hard) was statically compiled. We can see that using the `file` command:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/TaskReverseME-hard1.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

The code from the shared libraries is included within the binary, and that's why it's over 60 times larger than the [Reverse Me (Easy) task](/writeups/VU-Reverse-Easy/) binary:

<div class="row mt-3">
    <div class="col-8 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/TaskReverseME-hard2.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

It's an **ELF executable** (Executable and Linkable Format) which means it can be run in Unix-like operatng systems. Another important information is that it's **not stripped**, so we could search for interesting symbols.

When we run it, it asks for login and password information. Our goal is to find them since the flag is in the form of *VU{user,password}*.

After disassembling the binary using `objdump` we can identify the functions 
- `succes` at address `0x401ad5` and 
- `fail` at address `0x401aef` 

which are being called after the check of the credentials, depending their validation result.

<div class="row mt-3">
    <div class="col-8 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/TaskReverseME-hard3.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/TaskReverseME-hard4.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

Let's have a more dynamic approach than what we did in [Reverse Me (Easy) task](https://github.com/kallenosf/CTF_Writeups/blob/main/VU_Cyberthon_2023/Reverse%20Me%20(Easy).md). We are going to run the binary using a debugger, `gdb`:

<div class="row mt-3">
    <div class="col-6 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/TaskReverseME-hard5.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

We set a breakpoint at the main function, we run the program and then we disassemble:
<div class="row mt-3">
    <div class="col-8 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/TaskReverseME-hard6.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>
<div class="row mt-3">
    <div class="col-8 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/TaskReverseME-hard7.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

We then set two additional breakpoints at each `cmp` (compare) instruction. We continue execution and we are prompted for login and password input. We give two random numbers, `1234` and `4321`:

<div class="row mt-3">
    <div class="col-8 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/TaskReverseME-hard8.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

We continue execution and we reach the breakpoint at the first `cmp` instruction which compares registers `$eax` and `$edx`. The `$eax` register holds our input, while the `$edx` register is the expected value. We can inspect both by typing `info registers eax edx`. We see that the expected login value is `2018`.

<div class="row mt-3">
    <div class="col-7 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/TaskReverseME-hard9.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

If we contnue the execution the program will output "Sorry, try again" and exit, because the comparison is false. We can change the value of `$eax` to the expected one (2018) by typing `set $eax=2018`. Or we can just run from the start and give the correct login value.

<div class="row mt-3">
    <div class="col-7 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/TaskReverseME-hard10.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

Next, we can similarly find the password value by inspecting the values of `$eax` and `$edx` at the **second** comparison:

<div class="row mt-3">
    <div class="col-7 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/writeups/vu/TaskReverseME-hard11.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
</div>

According to the task description the answer format was like *VU{user,password}*, so the flag is:
**`VU{2018,3158}`**