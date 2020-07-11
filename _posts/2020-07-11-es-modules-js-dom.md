---
layout: post
title:  "Using JSDOM with ES modules"
date:   2020-07-11 13:05:55 -0500
categories: javascript
---
I spent some time trying to upgrade an older library that was using JSDOM to mock out DOM operations while testing.  After converting it more or less verbatim to comply with API changes in JSDOM, I ended up with something more or less like this:

    var exposedProperties = ['window', 'navigator', 'document'];
    
    const jsdom = new JSDOM('', , {
      url: 'https://localhost',
    });
    global.window = jsdom.window;
    global.document = global.window.document;
    Object.keys(global.window).forEach((property) => {
      if (typeof global[property] === 'undefined') {
        exposedProperties.push(property);
        global[property] = global.window[property];
      }
    });

What it is essentially doing here, is creating a new DOM representation and copying out all methods that would normally be globally available inside the browser into the global namespace of node.  In this way it is more or less simulating a similar environment in node that you would get in the browser.

You might wonder why the `url: 'https://localhost'` was needed.  That was because without it, JSDOM was throwing a security exception of `localStorage is not available for opaque origins`.  After some research, I discovered that [adding a url property to the config object was a possible solution to this](https://github.com/jsdom/jsdom/issues/2304).  

Adding the URL fixed that problem.  However, if you read carefully through the previously linked github issue, you might have noticed that it also stated that this technique of copying JSDOM properties to the global object is [*explicitly unsupported*](https://github.com/jsdom/jsdom/wiki/Don't-stuff-jsdom-globals-onto-the-Node-global) by JSDOM itself. However, while unsupported, this technique seems to be extremely popular.  Even the commenters in the said github issue seem to suggest using it as a possible solution.  

Not being happy with settling for a 'hack', I put some effort into doing the officially sanctioned way by JSDOM itself.  What they suggest is something like this (pulled from the wiki page):

    const { JSDOM } = require("jsdom");
    const myLibrary = fs.readFileSync("../../whatever.js", { encoding: "utf-8" });
    
    let window;
    beforeEach(() => {
      window = (new JSDOM(``, { runScripts: "dangerously" })).window;
    
      // Execute my library by inserting a <script> tag containing it.
      const scriptEl = window.document.createElement("script");
      scriptEl.textContent = myLibrary;
      window.document.body.appendChild(scriptEl);
    });
    
    it("should do the right thing", () => {
      assert.equal(window.myLibrary.doThing("foo"), "bar");
    });

This is all fine an good, but in my particular case, the file being tested was an ES module.  At the time of writing this post, [JS modules seem to be be unsupported by JSDOM](https://github.com/jsdom/jsdom/issues/2475).  As far as I can tell, the only way around this is to first run the scripts through babel to convert them to [Universal Module Definition (UMD)](https://github.com/umdjs/umd) and then import each one individually in your test files.  In other words, for each import in the file in your testing, you need to add something like this to your test script:

     const importedLib = fs.readFileSync("../../imported.js", { encoding: "utf-8" });
     const scriptEl = window.document.createElement("script");
     scriptEl.textContent = importedLib;
     window.document.body.appendChild(scriptEl);

That is in addition to doing that for the file you are testing itself.  This is an aweful lot of overhead as well as not very [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) at all as you are having to repeat the imports in your test files as well.  It might be possible to simplify this a little bit by bundling your scripts together using [browserify](http://browserify.org/) or something of the like but that would require one more step of setup and transcompilation prior to even starting to run the tests.

There might be a better solution out there but most examples out there that I could find seemed to be using the 'bad' solution of copying over things to the node global namespace.  I think for now, that might be the best approach if you need to use JSDOM for testing a library that requires a transpilation step.  Maybe once JSDOM get support for [ES modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules), that will change.


