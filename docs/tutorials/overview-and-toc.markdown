Documentation for Skippy version `{% include_relative version.markdown %}`.

## Overview


A quick tour how to use Skippy with {% if page.permalink contains "gradle" %}Gradle{% else %}Maven{% endif %} and JUnit {% if page.permalink contains "junit5" %}5{% else %}4{% endif %}.

__What You Need__
- About 15 minutes
- Your favorite text editor or IDE
- Java 17 or later
{% if page.permalink contains "gradle" %}- Gradle 7.3+{% else %}- Maven 3.2.3+{% endif %}

## Table Of Contents

- [Setting Up Your Environment](#setting-up-your-environment)
- [Exploring The Codebase](#exploring-the-codebase)
- [Run The Tests](#run-the-tests)
- [Re-Run The Tests](#re-run-the-tests)
- [Testing After Modifications](#testing-after-modifications)
