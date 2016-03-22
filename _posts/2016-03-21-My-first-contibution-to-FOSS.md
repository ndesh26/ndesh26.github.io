---
layout:     post
title:      My first contribution to FOSS
date:       2016-03-21 15:31:19
excerpt_separator: <!--more-->
categories: javascript 
comments:   true

---
I made my first contribution to FOSS to [gnome-maps](https://wiki.gnome.org/Apps/Maps/). The feeling that the code you wrote reaches hundreds of thousands of people is great. It makes me feel proud even when I write about it.

<!--more-->

![gnome-maps]({{ site.baseurl }}/assets/images/maps-highlight-route.png)

I am little late for this post as I made my first contribution on 1 March anyways gnome-maps allows you to search for a route from source to destination and you can choose the points via which you want to go. Before my contribution if you added a via point to a existing route and if no route exists then an infinte spinner is displayed as shown in figure. 

![maps-before]({{site.baseurl}}/assets/images/maps-before.png)

After my patch if you add a new via point and it results in no route then the warning is shown and it reverts back to the previous route and TextBox of the via point is emptied as in the figure 

![maps-after]({{ site.baseurl }}/assets/images/maps-after.png)

This is the [link](https://git.gnome.org/browse/gnome-maps/commit/?id=20a0ee8373216ece0e0b077f8f4ac17926817e97) of my commit. This contribution would not have been possible without the patience of Jonas Danielsson, the maintainer of gnome-maps. He helped me a lot with the patch and with submission of the patch. I would also like to thank Hashem Nasarat who helped me setup jhbuild. Thanks Jonas and Hashem. 

You can also contribute to gnome-maps we have lots of newcomer bugs. If you are interested we would be happy to get you up to speed and coding along with us. 
