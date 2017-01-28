---
# Specify the title here so it keeps the correct capitalization
title: BlackHat USA 2017 Training
layout: news_item
author: cbisnett
categories: [training]
venue: BlackHat USA 2017
registration_url: https://www.blackhat.com/us-17/training/fuzzing-for-vulnerabilities.html
---

It seems like only a few months ago that [Kyle](https://twitter.com/kylehanslovan) and I were in Las Vegas teaching at Blackhat but it's already time to start registration for this years courses. We're pleased to announce that we'll be back for the **7th** time teaching at a Blackhat event!

Each time we've taught [Fuzzing For Vulnerabilities](ffv/), we've incorporated student feedback to improve our teaching and the material and this offering is no exception. We've added several exercises and will be re-working the smart fuzzing exercise to make it easier for students to wrap their heads around. Block-based fuzzing is a hard concept to demonstrate to students, especially discussing how an input is decomposed into smaller chunks. The original exercise used a custom fuzzer that I wrote and required the students to define each of the building blocks that made up the complex structure of an MPEG4 file. Ultimately this left more people scratching their head than we would like.

Rather than redesign the fuzzer, we're going to switch to using [BooFuzz](https://github.com/jtpereyda/boofuzz) (an updated and re-factored version of Sulley). This allows us to focus more on the topic of how the input is decomposed from a high level and we'll pair that with several examples to show how to define the building blocks that make the approach so powerful. Stay tuned for additional details.

If you're interested in registering for the course, you can find the registration information on [the Blackhat site](https://www.blackhat.com/us-17/training/fuzzing-for-vulnerabilities.html). If you would like us to come to your facility and give a private training please contact me and we can set something up.