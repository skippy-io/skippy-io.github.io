---
layout: post
title:  "Skippifying Spring Boot"
date:   2024-01-08 00:00:00 -0600
author: Florian McKee
categories: news
permalink: /posts/skippifying-spring-boot
---

Check out the tutorial [**Skippifying Spring Boot**](../tutorials/skippifying-spring-boot) to learn how to skippify
[Spring Boot](https://github.com/spring-projects/spring-boot).
The scale and complexity of Spring Boot’s build make it a great choice to put Skippy to the test.

### TL;DR

- Spring Boot can be skippified using 5 lines of configuration changes
- Skippy's Test Impact Analysis analyzes over 4,000 tests and more than 2,500 classes in a timeframe comparable to that 
  of `./gradlew test`
- Skippy’s Predictive Test Selection
  - Catches all regressions that are caught when running all tests
  - Reduces the number of tests that need to be executed by 90% to 100%
  - Cuts down the time required for test execution by 84% to 98%


{% include_relative comments.markdown %}