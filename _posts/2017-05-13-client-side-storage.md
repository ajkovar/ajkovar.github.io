---
layout: post
title:  "Redux With Client Storage"
date:   2017-05-13 16:37:55 -0500
categories: javascript
---

Right now, it seems like redux-storage is the best storage option available when working with redux.  It has a flexible arcitecture that allows swiping out different engines.  You can use, for example, indexeddb.

While it is only supported in modern browsers, I think this combination really cannot be beat.  Using indexeddb, you will get nonblocking writes to the database so you will not have to worry about creating stutter in the UI or general slowdown in your application.  You will also get a legitimate database that is capable of saving object structures with native support for the different data types.  This makes it really easy to examine exactly what you have stored.  And with redux-storage, you will get the entire state of the application persisted between sessions without even having to worry about a backend.  This can be great when prototyping something.  It's also moves your web application a step closer to being offline capable.

If you were to try to do this with a localstorage solution, you would get a blocking operation that would store it's data as a string serialized object.  This is obviously also not very performant for a large quantity of data, as it would have to reserialize the entire data structure any time it needed to perform a save.

Additionally, you can use redux-storage with [react native async storage](https://github.com/react-stack/redux-storage-engine-reactNativeAsyncStorage), giving you a uniform way to persist state across platforms.  And even better, there are plugins available that allow you to do things such as perform automated migrations for databases.


{% highlight javascript %}
import _ from 'lodash';
let result = _.isArray(a);
{% endhighlight %}

