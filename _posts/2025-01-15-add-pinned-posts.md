---
layout: post
title: "TIL: How to Add Pinned Posts to a Jekyll Blog"
date: "2025-01-15 20:00:00 +0000"
categories: example
tags: [jekyll, github pages, html, css]
excerpt: "A bit of HTML and CSS to add pinned posts to the top of the blog, simple."
---

To add a pinned post to a `Jekyll` website, you only have to modify one file (`index.html`), or two files (including `main.css`) if you want to highlight the pinned post in some way.

Before adding the pinned post, my `index.html` was very straightforward (heavily commented for pedagogical reasons):

```html
---
layout: default
---

<div class="home">

   # create an unordered list with the CSS class "posts" (styled in the sites main.css)
  <ul class="posts">

    # Jekyll Liquid syntax for looping through site.post
    {% raw %} {% for post in site.posts %}

    <li>

      # format and display the post-date
      <span class="post-date">{{ post.date | date: "%b %-d, %Y" }}</span>

      # generate a clickable link (<a>) to the post.url, which is prepended with the site.baseurl, add that clickable link to the post.title, taken from the post frontmatter
      <a class="post-link" href="{{ post.url | prepend: site.baseurl }}"
        >{{ post.title }}</a
      >
      <br />

      # this Jekyll Liquid logic checks if there is a post.custom_excerpt in the posts frontmatter, and if so, displays it. If not, it displays the first paragraph of text from the post
      {% if post.custom_excerpt %}{{ post.custom_excerpt}} {% else %} {{
      post.excerpt }} {% endif %}
    </li>
    {% endfor %} {% endraw %}
  </ul>
</div>
```

This `index.html` treats all posts the same, but I needed a way for the site to treat pinned posts differently. There are many ways to do this, but the simplest I could find was to create the pinned post as a normal post in `_posts`, with the frontmatter tag `pinned`, and then scan through the posts to find the pinned posts, generate them first with custom css, and then go through again and generate the rest:

```html
---
layout: default
---

<div class="home">
  {% raw %}

  <!-- Pinned Posts Section -->
  <!-- scan through posts to find posts tagged pinned, so they can be generated first, and with a custom css -->
  <ul class="posts">
    {% for post in site.posts %} {% if post.tags contains 'pinned' %}
    <li class="pinned">
      <span class="post-date">Pinned Post</span>
      <a class="post-link" href="{{ post.url | prepend: site.baseurl }}"
        >{{ post.title }}</a
      >
      <br />
      {% if post.custom_excerpt %}
      <p>{{ post.custom_excerpt }}</p>
      {% else %}
      <p>{{ post.excerpt }}</p>
      {% endif %}
    </li>
    {% endif %} {% endfor %}
  </ul>

  <!-- Normal Posts Section -->
  <!-- scan through posts again to find posts not tagged pinned, so they can be generated in order, with their normal css -->
  <ul class="posts">
    {% for post in site.posts %} {% unless post.tags contains 'pinned' %}
    <li>
      <span class="post-date">{{ post.date | date: "%b %-d, %Y" }}</span>
      <a class="post-link" href="{{ post.url | prepend: site.baseurl }}"
        >{{ post.title }}</a
      >
      <br />
      {% if post.custom_excerpt %} {{ post.custom_excerpt}} {% else %} {{
      post.excerpt }} {% endif %}
    </li>
    {% endunless %} {% endfor %}
  </ul>
  {% endraw %}
</div>
```

The new class css `pinned` has to be defined. This can be done by adding something like the following to the `main.css` file:

```css
.posts li.pinned {
  border: 1px solid #d3d3d3;
  background-color: #f9f9f9;
  padding: 5px;
  margin-bottom: 29px;
  border-radius: 3px;
}
```

You can pick any colours, padding, margin etc. you like, but I'm happy enough with this:

<div style="text-align: center;">
  <img src="{{ site.baseurl }}/assets/posts/til-blog-pinned-post.png" style="width: 100%; border: 1px solid #d3d3d3;;">
</div>

## A Note on Jekyll's Liquid Syntax

`Jekyll` uses the Liquid template language (originally developed by Shopify, based on `Ruby`) to generate static `HTML` websites from dynamic data. The main Liquid features required for specifying pinned posts are as follows:

<u>Variables</u>: Denoted by ``{% raw %}{{ ... }}{% endraw %}`, these represent dynamic content pulled from Jekyll's data structures or post frontmatter.

```liquid
{% raw %}
{{ post.title }}
{% endraw %}
```

<u>Tags</u>: Denoted `{% raw %}{% ... %}{% endraw %}`, these control logic and flow, such as loops and conditionals.

```liquid
{% raw %}
{% for post in site.posts %}
{% endraw %}
```

<u>Filters</u>: Filters use a pipe \| to process the input on the left using the specified function or transformation on the right. In the following example the ouput of `post.date` is filterd using the format on the right.

```liquid
{% raw %}
{{ post.date | date: "%b %-d, %Y" }}
{% endraw %}
```

<u>Conditionals</u>: `{% raw %}{% if ... %}`{% endraw %}, `{% raw %}{% elsif ... %}{% endraw %}`, and `{% raw %}{% else %}{% endraw %}` tags enable conditional logic.

```liquid
{% raw %}
{% if post.tags contains 'pinned' %}
{% endraw %}
```
