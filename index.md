---
layout: default
title: Marco Sabatini, Software Crafter
---

## Overview
Hi folks.
My name is Marco Sabatini I'm a Software craftsman, clean code disciple, tdd lover, xp-er, quantified self geek, more than 10 years on the field and still love it.
This is my chance to share something with you!

## Blog

<div>
    <ul>
        {% for post in site.posts %}
        <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}" title="{{ post.title }}" target="_blank">{{
            post.title }}</a></li>
        {% endfor %}
    </ul>
</div>
