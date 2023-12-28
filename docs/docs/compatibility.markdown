---
layout: page
title: Compatibility
permalink: /docs/compatibility
---

Compatibility matrix for Skippys’s integration with Java, JUnit and build tools. Versions not mentioned here might
work, but there is no guarantee.

# Java

| Skippy       | Java         |
|--------------|--------------|
| all versions | 17 or later  |

# JUnit

| Skippy  | JUnit 4      | JUnit 5      |
|---------|--------------|--------------|
| ≥ 0.0.8 | 4.9 or later | 5.9 or later |
| ≤ 0.0.7 | ❌           | 5.9 or later |

# Build Tool

| Skippy  | Gradle                         | Maven        |
|---------|--------------------------------|--------------|
| ≥ 0.0.9 | 7.3 or later<br/> 8.0 or later | 3.9 or later |
| ≤ 0.0.8 | 7.3 or later<br/> 8.0 or later | ❌           |

# JaCoCo

| Skippy   | JaCoCo           |
|----------|------------------|
| ≥ 0.0.9  | 0.8.7 or greater |
| ≤ 0.0.8  | 0.8.9            |