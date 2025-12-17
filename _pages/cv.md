---
layout: archive
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}

Education
======
* Ph.D in Astronomy, Nanjing University, 2028 (expected)
* B.S. in Astronomy, Nanjing University, 2021

Work experience
======
* Summer 2020: Internship
  * Shanghai Academy of Spaceflight Technology
  
Skills
======
* Programming languages include:
  * C++
  * Python

Publications
======
  <ul>{% for post in site.publications reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>
  
Talks
======
  <ul>{% for post in site.talks reversed %}
    {% include archive-single-talk-cv.html  %}
  {% endfor %}</ul>
  
Teaching
======
  <ul>{% for post in site.teaching reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>
