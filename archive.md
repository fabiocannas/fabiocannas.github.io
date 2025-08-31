---
layout: archive
classes: wide
title: Articles
permalink: /articles/
author_profile: true
feature_row_1:
  - image_path: "/assets/images/2025-07-27-The_Azure_Developers_Superpower_azd_CLI/2025-07-27-The_Azure_Developers_Superpower_azd_CLI_intro_slide.jpg"
    alt: "2025-07-27-fabio-cannas-the-azure-developers-superpower-azd-cli"
    title: "The Azure Developer's Superpower: azd CLI"
    excerpt: "Following is the content of my session at the Azure Meetup Casteddu, held a few days ago at Sa Manifattura, Cagliari."
    url: "/2025/07/27/The_Azure_Developers_Superpower_azd_CLI.html/"
    btn_label: "Read more"
    btn_class: "btn--primary" 
---
{% include feature_row id="feature_row_1" type="left" %}

{% for post in site.posts limit: 5 %}
  {% include archive-single.html %}
{% endfor %}