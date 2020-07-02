---
layout: post
title: Partial Functions vs Partially Applied Functions
---

So lately I've been doing a lot of playing around with scala.  Quite a fun and interesting language.  Pretty soon I plan on posting about some my initial thoughts about the language so far.

Anyway, a key concept in scala is that of partial functions.  Until recently, I had this concept mixed up with that of partially applied functions, which while sounds very similar is actually not related at all.  The worst part is, appearantly *a lot* of people are confused about this and there is *a lot* of misinformation out there.  Because of this, I thought it might be a good idea to do a quick post on it and hopefully clear some of the confusion.

## Partial Functions

Basically, partial functions like many cs concepts (in scala in particular) are very mathematical in nature.  You can think of it as a function that is only valid over a particular range.  A classic example might be:

<pre>
Y=1/X
</pre>

For almost all values, this function can be plotted on a graph except at 0. It is undefined at that value.

In scala, you could express this as so:

<pre class="brush:java">
object F extends PartialFunction[Double, Double] {
      def isDefinedAt(x: Double): Boolean = x!=0
      def apply(x: Double): Double = 1/x
}
</pre>

A partial function in scala is basically a subclass of a regular function that defines a contract that it promises to adhere to.  The parametrized types [Double, Double] refer to the type of the argument and the return value. On its own, this isn't anything new or exciting, but the neat thing about scala is that it has support for this construct built into the language.  Another way of writing this would be:

<pre class="brush:java">
val f:PartialFunction[Double, Double] = { case x if x!=0 => 1/x }
</pre>

According to section 8.5 of the [scala language specification](http://www.scala-lang.org/docu/files/ScalaReference.pdf) This is called a "pattern matching anonymous function" and can be expanded out into either a function or a partial function, so the type of p cannot be inferred and has to be written explicitly.

## Partially Applied Functions

Partially applied functions, on the other hand refers to the concept of taking a function of arity x and creating a new function of arity < x.  In other words, automatically supplying some of the arguments in advance, creating a new function that accepts fewer arguments.

<pre class="brush:java">
val f = (x:String, y:String) => println(x + " " + y)
val f2 = f("hello ", _:String)
f("hello", "there") // outputs "hello there"
f2("there") // also outputs "hello there"
</pre>

I won't go into detail about why this is useful, there's plenty of material out there that describes that.  Daniel Spiewak actually wrote a [great writeup](http://www.codecommit.com/blog/scala/function-currying-in-scala) on partially applied functions and currying in scala.

To make things just a bit more confusing (as Daniel touches on in his article), partially applied functions in scala are slightly different than the strict definition above.  They refer to the concept of supplying *any number* of arguments at a later time.  Possibly even none.  What is the point of partially applying a function to which you supply no arguments, you ask?  Well in scala, you can partially apply a function, supplying no arguments to tell the compiler that you want to treat the function as a value, instead of trying to apply it.  For instance:

<pre class="brush:java">
def f(s:String) = println(s)
val f2 = f _ // shorthand for f(_:String)
// essentially the same thing:
f("asdf")
f2("asdf")
</pre>

So the arity doesn't change, but using this, you can pass a function around as a value.  This, I suppose, is necessary because scala allows functions to be called without parenthesis, so it needs some mechanism for indicating that you don't want to invoke it.

This gets done automatically for you in certain situations:

<pre class="brush:java">
List.range(1,10).foreach(println)
// same as:
List.range(1,10).foreach(println _)
</pre>

Anyway, hopefully this helps clear up some of the confusion on these otherwise pretty simple concepts.
