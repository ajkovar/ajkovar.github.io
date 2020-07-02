---
layout: post
title:  "Something to Took Look Out For When Importing in Frontend Javascript"
date:   2017-03-25 16:37:55 -0500
categories: javascript
---
Be careful when importing functions from libraries while developing web applications. For example:

{% highlight javascript %}
import _ from 'lodash';
let result = _.isArray(a);
{% endhighlight %}

When you import like this, you will be importing the entire lodash library into your front end application.  Instead, prefer to load the specific utility that you are looking for.

{% highlight javascript %}
import isArray from 'lodash/isArray';
let result = isArray(a);
{% endhighlight %}

Doing so in this example can means the difference between 20k bytes of extra download.  Now you may be thinking that you can get away with this instead:

{% highlight javascript %}
import { isArray } from 'lodash;
let result = isArray(a);
{% endhighlight %}

And while this would certainly be more concise (especially if you are loading multiple functions) it also results in loading the entire library of lodash into your application.  Libraries like webkit are not able to do tree shaking using with this method.
