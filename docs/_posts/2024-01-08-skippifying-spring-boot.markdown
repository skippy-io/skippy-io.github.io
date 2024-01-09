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

### Highlights

- 5 lines of configuration changes is all it takes.
- Skippy's Test Impact Analysis is shown to be highly efficient for large projects. It analyzes over 4000 tests and more
  than 2500 classes with minimal overhead, adding only 23 seconds (or 13%) to the total time compared to `./gradlew check`.
- Skippy’s Predictive Test Selection demonstrates its effectiveness in identifying all regressions that are caught when
  running all tests. This feature reduces the number of tests that need to be executed by 90% to 100% and
  cuts down the time required for test execution by 84% to 98%.