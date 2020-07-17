---
layout: post
title:  "useRef - more than just for elements"
date:   2020-07-15 13:05:55 -0500
categories: javascript
---
If you have played around with the relatively recently introduced react hooks at all, you might have wondered if it was possible to create a variable in one useEffect and reuse it in another and what the best way to do that would be.  The answer to this might not be so obvious but it turns out that [React](https://reactjs.org/) provides a way.

Let's give an example.  Let's say you want to instantiate something on mount and then reuse it later once a specific prop changes:

    function({ someProp }) => {
      let something;
      // this one will run only once after mount (notice the [])
      useEffect(() => {
        something = new Something();
      }, []);
    
      // this one will run every time someProp changes
      useEffect(() => {
        // WARNING: This won't work!
        something.doSomething(someProp);
      }, [someProp]);
    }

The above (broken) code demonstrates this more or less.  However, if you are expecting that `something` will hold it's value between renders, you are wrong.  You might expect this due to the fact that it is closing over the variable, but what it is closing over here is `undefined` during the execution of `render`.  The object you actually want to hold a reference to doesn't get created until after render has finished executing.  Additonally, each time render runs, it creates a new version of that callback.  So the next time `someProp` changes, the `something` in the second callback will still hold a reference to `undefined`, which obviously is going to cause some problems in the above code.

The answer is [`useRef`](https://reactjs.org/docs/hooks-reference.html#useref).  Let's rewrite that previous example: 

    function({ someProp }) => {
      let somethingRef = useRef();
      useEffect(() => {
        somethingRef.current = new Something();
      }, []);
    
      useEffect(() => {
        somethingRef.current.doSomething(someProp);
      }, [someProp]);
    }

This will work.  Now what's being closed over is a refence variable.  And React will hold onto the value inside that variable for you and pass it back to you the next time another render cycle comes along.  This is similar to how instance fields worked when using class based components.  The good thing about using this method is that setting `somethingRef.current` to a new value will not trigger another render so the above code won't cause un unnessesary double render (as opposed to using `setState` to hold onto a value).  And `useRef`, obviously, is usable in other React hooks that accept callbacks so it can be used for more than just `useEffect`.

And that's about it.  This is a relatively simple concept but it may not be obvious if you are new to hooks (nor is it covered in [React's own write up about `useEffect`](https://reactjs.org/docs/hooks-effect.html)) and there seems to be some [misinformation out there](https://stackoverflow.com/questions/58140009/how-to-use-variable-declared-in-useeffect-in-another-function) so I thought it merited posting about.