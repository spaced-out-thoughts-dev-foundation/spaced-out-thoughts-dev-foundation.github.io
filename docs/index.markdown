---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Playwrite+IT+Moderna:wght@100..400&display=swap" rel="stylesheet">


<center><img src="../images/DevelopmentFoundation_logo.png" alt="drawing" width="200"/></center>

<br>

<h2 style=" font-family: 'Playwrite IT Moderna', cursive"> We are a small Ruby shop determined to make programming smart contracts as easy as possible. </h2>

<br>

***

<h2>Portfolio:</h2>
* [Digicus](digicus): a block-based programming language supporting visual construction, manipulation, and comprehension of smart contracts

***

<ul>
  {% for post in posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>