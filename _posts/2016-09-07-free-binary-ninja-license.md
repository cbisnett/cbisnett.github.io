---
layout: news_item
author: cbisnett
categories: [training]
---

When I updated my [Fuzzing For Vulnerabilities](/ffv) course for Blackhat USA 2016, I switched the exercise virtual machine from Windows 7 x86 to Ubuntu 16.04 x86_64. I did this for several reasons including licensing issues, performance increases, and a general lack of fuzzing tools and OS support. This meant that I could now provide exercises and hands on experience for things like [AFL](http://lcamtuf.coredump.cx/afl/) and [Radamsa](https://github.com/aoh/radamsa) but one major downside was that the free demo version of [IDA Pro](https://www.hex-rays.com/products/ida/index.shtml) does not support 64-bit executables. The course doesn't heavily rely on reverse engineering but it is helpful for some of the exercises and really fundamental to visualizing code coverage results. Until today I didn't have a good answer to this problem.

After talking with the guys over at [Vector 35](http://vector35.com/), I've decided that not only am I going to integrate [Binary Ninja](https://binary.ninja/) into the course, I'm going to give every student a complementary personal license! This way every student will be able to disassemble and inspect all of the exercise binaries and learn a little about writing scripts to visualize the results of their fuzzing. I'm really happy to see how far this tool has come and support some great friends and colleagues.

If you're interested in [registering](http://theelectronjungle.com/2016/08/29/fuzzing-for-vulnerabilities-october-2016/) for the course and claiming your free Binary Ninja license, please email me at [cbisnett@gmail.com](mailto://cbisnett@gmail.com).
