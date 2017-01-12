---
layout: page
title: Archive
---

## Blog Posts

<!--{% for post in site.posts %}-->
   <!--{{ post.date | date: "%Y"  }} &raquo; [ {{ post.title  }}  ]({{ site.baseurl }}{{ post.url  }})-->
<!--{% endfor %}-->
{% for post in site.posts %}
{% capture this_year %}{{ post.date | date: "%Y" }}{% endcapture %}
{% unless year == this_year %}
{% assign year = this_year %}
### {{ year }}
{% endunless %}
{{ post.date | date: "%b %d"  }} &raquo; [ {{ post.title  }}  ]({{ site.baseurl }}{{ post.url  }})
{% endfor %}


