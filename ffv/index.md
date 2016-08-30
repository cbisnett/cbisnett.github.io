---
layout: news
title: Fuzzing For Vulnerabilities
permalink: /ffv
author: all
---

# Fuzzing For Vulnerabilities

Even in 2016, it is still possible to find zero-day vulnerabilities in production software using simple fuzzers. The last couple of years have seen numerous companies launch bug bounty programs in an attempt to crowd-source a solution to this problem. Whether your a member of a development team looking to fuzz your software before release or a researcher looking to find vulnerabilities to score some bug bounty prizes, Fuzzing For Vulnerabilities will get you started developing fuzzers and running them against target software.

Fuzzing For Vulnerabilities continues to be updated based on previous student feedback and incorporates new material and labs.

### Topics

- **Fuzzing Overview** – An introduction to the fundamental techniques of fuzzing including mutation-based and generative-based fuzzers, and covers the basics of target instrumentation.

- **Dumb Fuzzing** – An overview of the benefits and drawbacks of generic fuzzers which have little to no insight into the format of the data being fuzzed.

- **Smart Fuzzing** – An in-depth discussion of specialized mutation-based and generative-based fuzzers, choosing fuzzed values to increase the likelihood of a crash, and using protocol specifications as a guide to develop a fuzzer.

- **Advanced Techniques** – Covers advanced techniques to increase fuzzer efficiency and effectiveness. Topics include: using AddressSanitizer to enhance vulnerability detection, collecting code coverage statistics, corpus distillation, in-memory fuzzing, differential fuzzing, and introduces whitebox fuzzing (input generation).

- **Crash Analysis** – Discussion of tools and methods that aid in analyzing large numbers of crashes to determine uniqueness and give a hint at the severity.

### Who Should Take This Course?

Ideally anyone who is looking to learn more about fuzzing. This course starts with beginner level concepts to get everyone up to speed and then delves into how to write custom fuzzers and more advanced topics.

### Student Requirements

- Development experience in Python. All course exercises incorporate custom Python scripts that students will modify. All students will be supplied with solutions, so students with no or limited Python experience can still learn from the course.

- Basic knowledge of C/C++ (high level) data-types

- Basic understanding of memory corruption vulnerabilities (stack buffer overflow, heap buffer overflow, etc.)

### What Students Should Bring?

**Training courses at conferences (Blackhat, etc.):**

Students should bring a laptop with a 64-bit processor and operating system and at least 4Gb of RAM. The provided virtual machine is 64-bit Ubuntu 16.04 and will not run on 32-bit host machines.

**Training courses at other venues (training centers):**

A computer will be provided for you at the training center and will come already setup with the virtual machine image installed. You will also be provided with the virtual machine image to take home with you. It is up to you if you decide to use the provided machine or your own. If using your own laptop, see the above requirements.

### What Students Will Be Provided With?

In addition to the training manual and exercise booklet, students will receive an Ubuntu 16.04 virtual machine loaded with all the course exercise material including solutions to all of the exercises.

### Trainers

[Chris Bisnett](https://twitter.com/chrisbisnett) is co-founder and product manager at Huntress Labs, a startup focused on automated breach detection through hunting persistence mechanisms. In the past Chris has worked as a defense contractor for the U.S. government and as a vulnerability analyst at the NSA RedTeam. He has extensive experience reverse engineering proprietary protocols and developing fuzzers. When not working, Chris enjoys participating in hacker capture-the-flag events.

[Kyle Hanslovan](https://twitter.com/kylehanslovan)
