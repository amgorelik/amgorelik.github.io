---
layout: post
title:  Code Writing Code
date:   2019-03-02
categories: programming
excerpt: "A long time ago, in the midst of spending a few happy years writing C++
code, I happened to work on a project in plain C. It felt awkward
after C++, but ended up being somewhat tolerable, until I needed a
couple of hash tables for objects of different types. In my mind, it
was a simple, innocent request, but this is where my foray back
to the C land hit a brick wall."
---

<!--![](https://upload.wikimedia.org/wikipedia/commons/2/29/Gunung_Rinjani_from_Gili_Air_1.jpg)-->

![](https://www.telegraph.co.uk/content/dam/Travel/2018/July/gili-GettyImages-873507622.jpg)

<!-- Photo credit: Getty -->

_Until recently, the first and last time I used Java was in the
year 2000. Back then, it was a relatively young language, simple and
inefficient, looking as some seriously bastardized C++ (regardless of
what its authors had to say on the subject). I shrugged and moved away
from Java._

_Fast forward to 2019: I am writing in Java again, and of course it
has a different feel to it now, neither simple nor slow
anymore. Somehow, I got especially fascinated by its metaprogramming
capabilities, or the lack thereof. Here is a story of what I found—but
first, some personal history._


## Doing It By Hand

A long time ago, in the midst of spending a few happy years writing C++
code, I happened to work on a project in plain C. It felt awkward
after C++, but ended up being somewhat tolerable, until I needed a
couple of hash tables for objects of different types. In my mind, it
was a simple, innocent request, but this is where my foray back
to the C land hit a brick wall.

Dismayed that there was no standard solution to this simple problem,
and having similar issues with code duplication elsewhere on the
project, I ended up implementing a simple templating system, and a
Perl script that would generate C files from the templates.

Originally I thought it was a rather heavy-handed, desperate approach,
but it worked remarkably well, and later it grew on me. All it
required was an extra line in the makefile; the intermediate C file
were readily available for inspection and debugging; and of course the
power of Perl was far greater than any other solution available.

Since then, I became a big proponent of code writing code.
Essentially, you construct a higher-level language for
yourself. A higher language always means greater expressive power and
programmer productivity.

Let's talk, for a second, about other options available to me back
then. Of course, C does have a built-in solution for code generation I
was “supposed” to use: the preprocessor. Alas, the C preprocessor is a
sad joke (beyond a few simple tasks like conditional compilation);
it’s not even Turing-complete. Which is kind of disappointing,
historically speaking, since there were industrial languages before C
with vastly superior preprocessing capabilities.

Take, for example,
[PL/I](http://web.eah-jena.de/~kleine/history/languages/IBM-PLI-F-LangRefMan.pdf),
developed by IBM in the 1960s. I remember very well learning how to
program it in school. It had a full-featured preprocessor with typed
variables, loops and subroutines, not to mention the syntax being a
subset of the PL/I language itself.

With the rise of minicomputers and PCs all that somehow got
forgotten. The history of computing made one of its many resets of
complex to simple. One can easily argue that even in 2019 the modern
mainstream languages still haven’t caught up with the 1960s state of
the art.

If I were writing in C++ instead of C, I of course would have easily
solved my hash table problem with templates. However, C++ templates
make a poor general solution for metaprogramming, even though they
_were_ shown to be Turing-complete. I still remember reading the famous
_[Modern C++ Design](http://erdani.com/index.php/books/modern-c-design/)_
by Andrei Alexandrescu, and briefly considering checking myself into a
mental asylum after finishing it. Of course, it was the same Alexandrescu who wrote a
multi-page
[paper](http://www.drdobbs.com/generic-min-and-max-redivivus/184403774)
exploring different implementation options for `max(a, b)` in
templated C++ and coming to the conclusion that no satisfactory
solution exists (or, at least, didn’t exist in 2001 vintage C++).

## All That Is Gold Doesn’t Glitter

After my happy C++ years, I happen to work for a company that was
doing enterprise software development in Lisp—an endangered species,
indeed!

My time there was an eye-opener. Lisp, the second oldest high-level
programming language (1958; FORTRAN was released in 1957), for decades
was ahead of much more recent languages, that only recently caught up
with it—mostly.

One of the areas where Lisp was far ahead of its time was its
unmatched preprocessing capability (not explicitly existing in
[the original version of Lisp](http://www-formal.stanford.edu/jmc/recursive.html),
but introduced soon thereafter, in the early 1960s). In Lisp’s macro
system, the preprocessor is not just a subset of the source language, as
in PL/I; it _is_ the source language, just applied to the source code
at compile time. Here is a very short example of a unit test framework
definition in Common Lisp (from _Practical Common Lisp_ by Peter Seibel):

```common_lisp
(defun report-result (result form)
    (format t "~:[FAIL~;pass~] ... ~a~%" result form))

(defmacro check (&body forms)
    `(progn
        ,@(loop for f in forms collect `(report-result ,f ',f))))
```

Given that, you can now start testing something, like arithmetic expressions:

    > (check
          (= 3 (+ 1 2))
          (= 5 (* 2 2)))

    pass ... (= 3 (+ 1 2))
    FAIL ... (= 5 (* 2 2))

Again, I don’t know any commonly used language that can accomplish the
same as elegantly and with such a pleasing end result. And it is
conceivable that no such language exits: the power of Lisp macros
comes from the ability to treat the program source as data, which in
turn comes from Lisp the language having essentially no syntax.

Very few programming languages have trivial syntax. Try to manipulate
C++ or Perl sources programmatically!

Another powerful application of Lisp macros is defining new
languages. For example, in CLSQL (a Common Lisp library providing
interface to SQL databases) you can write something like this:

```cl
(select [email] :from [users]
    :where [= ["lower(name)"] (string-downcase customer)])
```

—not only protecting yourself from SQL injection, but also getting
compile-time syntax checking for SQL!

But probably even more impressive use of macros is in Common Lisp
itself, to extend the basic underlying Lisp language constructs. You
might have noticed a _for-each_ loop in the code above: `(loop for f
in forms collect ...)`. When every other language on the planet wants
to have a for-each loop, they have to add it to the basic language
definition (thank you C++ and Java for finally doing that after so
many years of pain). In Common Lisp, this loop is just a part of the
standard library: yet another macro defined in terms of much more
basic language constructs.

The ease and the natural way of defining macros in Lisp have a very
interesting practical effect. Whereas writing macros in C is
(rightfully) frown upon, the Lisp programmer has no such
inhibitions. From what I could see, real-world Lisp programmers would
write macros literally every day; for every bit of functionality you
need to implement, you just make a decision whether it is better
executed at run time (then it’s a function) or at compile time (then
it’s a macro).

## Dynamic All The Way

Eventually, I had to keep up with the times, and move to a dynamic
language. It happened to be Python, because pragmatically it’s the
best choice these days, much better than my beloved Perl. I have no
idea why a language created in 1991 feels modern—maybe it is just a
function of me being old.

<!-- It actually turns out to be older than C++ (1993)! -->

<!-- It’s definitely younger than the languages we discussed so far,
though (Lisp: 1958, Pl/I: 1964, C: 1972, C++: 1985, Java: 1995) -->

Metaprogramming in Python differs from all the other languages we’ve
explored so far, because it happens at run time. And where else?  To be
sure, Python does have a compiler, which even produces artifacts on
hard disk (the infamous *.pyc files), but Python compilation is so
fast and boring that nobody really cares about it. Everything
interesting in the Python land happens at run time.

Let’s say we want to implement a facility that would add getter and
setter methods for class member variables (called in Python
_attributes_). This is not a common Python pattern, but if were
insisting on it, we would probably use Python decorator. Here is one
way to do it:

```python
def getter_setter(*args):
    def getter(member): return lambda self: getattr(self, member)
    def setter(member): return lambda self, val: setattr(self, member, val)
    def decorator_getter_setter(cls):
        for member in args:
            setattr (cls, 'get_' + member, getter(member))
            setattr (cls, 'set_' + member, setter(member))
        return cls
    return decorator_getter_setter

@getter_setter('x', 'y')
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
```


Now you can do:

```python
p = Point(1, 2)

p.set_x(11)
print(p.get_x())
```

By the way: only when I tried to implement it, I realized that unlike
in C++ and Java, the problem statement is ambiguous. How do you even
know what attributes your class has? They can be added and deleted
dynamically, at run time.

In fact, that’s the _only_ way they are created (in the case of our
`Point`, in the constructor); there are no capabilities in Python to
pre-define instance attributes. In this example, we solve this problem
by explicitly telling the decorator for which attributes to define
getters and setters; not an unreasonable choice, since it gives
full control to the programmer while still keeping the code short and
clean.

Note that even though the code mutation happens at run time, it is
only done once for the given program run (at _import time_). The
performance implications are thus almost certainly negligible.

Of course, decorators can be used in an even simpler fashion: to
define pre- and post-actions for functions, for example if we wanted
to time them:

```python
def timeit(func):
    def wrapped(*args, **kwargs):
        start = time.time()
        func(*args, **kwargs)
        print("%s took %f s." % (func.__name__, time.time() - start))
    return wrapped

@timeit
def foo(...):
    ...
```

This is quite similar to aspect-oriented programming in Java, only so
much simpler. There is no magic involved; after all, decorators don’t
give you any special power; they are just a syntactic sugar for
`func = decorator(func)`.

If they are just some syntactic sugar, why do we even need them? Here
is an important part: because they are concise, and make the
programmer’s intention clear. In essence, they make the code more
declarative and less procedural. They help us to define a higher level
language.

On the other hand, just willy-nilly reassigning functions on the fly
is the opposite: confusing, error-prone, hard to inspect and to reason
about. It feels similar to the (in)famous `#define TRUE FALSE` in C: a
good reason why the preprocessor usage is frowned upon in that
language.

(On yet another hand, Lisp macros are totally fine: their _definitions_
may be slightly harder to read, but their _usage_ is as readable as
regular functions.)


## Beautiful Hacky Island

And now we come to my latest foray into Java. Last time I used it was
a long, long time ago: in the year 2000. Back then it was a young and
simple language, essentially C++ stripped of its perceived
complexities, including its meager metaprogramming tools:
preprocessing and templates. That was yet historical reset from
complex to simple, not unlike Go in our days.

Sop, speaking about simple: data in Java data structures turned into
~~a pumpkin~~ `Object` references, and there was absolutely no way
around it, short of falling back to generating code with hand-written
Perl scripts.

I call this _ostrich typing:_ bury your head in the sand and pretend
that you have a strongly, statically-typed language.

Fast-forward 19 years: Java is no longer young not simple, and it got
templates all right (called “generics”).

Now, while re-learning Java by programming in it, I was quite
surprised to see code like this:

```java
@Getter
@Setter
class Point {
    int x;
    int y;
}
```

It looked almost identical to the Python code we’ve written above,
and achieved the same purpose: programmatically generate getter and setter
functions for all the class attributes. Clearly, Java has come a long
way since I touched it last time!

I was curious to learn what was that and how it worked. Here is what I
learned.

The @-prefixed modifiers—annotations—have been a part of Java for
[quite some time](http://www.javainthebox.net/laboratory/J2SE1.5/LangSpec/Metadata/materials/metadata-1_0-prd-spec/metadata-public-draft.html). (Curiously, even though my first reaction was,
“how cute, it looks almost like Python”, in reality it went the other
way around: it was the Java annotations syntax that
[inspired decorators in Python](https://www.python.org/dev/peps/pep-0318/#background).)

Annotations are widely used in modern Java—typically, for what their
name suggest: to annotate, or mark, certain code elements for the
run-time inspection via reflection. For example, the JUnit framework
iterates over test class methods at run time, finds the ones marked
with `@Test`, and executes them as test cases. This is fine, and
clearly useful, but not particularly impressive, and can hardly be
classified as code writing code. In more powerful languages, where
functions/methods are first-class object, one can set their attributes
directly, with no need for special syntax:

    def foo ():
        ...

    foo.test = True

Interestingly, one motivation for the unusual annotation syntax was
the need to maintain backwards compatibility—in particular, not to
introduce any new reserved keywords. (At this point, I can’t resist
the temptation to mention the good old PL/I again. Somehow it managed
to have no reserved keywords: you could name your variables “if” and
“do” to your heart’s content, and the compiler was competent enough to
tell apart statements from identifiers based on the grammar. How come
we can’t do it anymore fifty years later?)

In any case, the language designers’ plan seems to have worked well
enough. Different Java frameworks are using annotations to enrich the
language, essentially defining a higher language on top of Java. One
good example is the Spring framework, essentially
[built around annotations](https://zeroturnaround.com/rebellabs/spring-framework-annotations-cheat-sheet/).
They seem to be so pervasive that are naturally causing some
[thoughtful backlash](https://blog.softwaremill.com/the-case-against-annotations-4b2fb170ed67).
Anyhow, Spring annotations, like many others, seem to be based on the
same run-time reflection behavior we just discussed for JUnit.

There is, however, a way to handle annotations at compile time. You
can write your own annotation processor (a class implementing
[`javax.annotation.processing.Processor`](https://docs.oracle.com/javase/10/docs/api/javax/annotation/processing/Processor.html)—or,
usually, extending `AbstractProcessor`) and hook it up into a
compiler. You just place a jar file with
[a special processor description](http://hannesdorfmann.com/annotation-processing/annotationprocessing101)
in your build path, and the compiler automatically picks it up an
calls your annotation processor as needed.

This is where things get exciting. First of all, the Java compiler is
actually running a JVM and executing Java code during the compilation
process, and runs your code at compile time! Second, your annotation
processor has access to AST (abstract syntax tree, the parsed
representation of the source code being compiled) and can _generate_
more source code, which would also be compiled during the same
compilation process. This definitely introduces a completely new,
powerful capability into the language, unlike anything it had before.

Unfortunately, the access to AST within an annotation processor is
read-only. You can’t _change_ the code being compiled, just generate
more.

If an annotation processor cannot change the currently compiled class,
then how that `@Getter`/`@Setter` business can possibly work? You will
not find an answer in the Java language documentation, yet this is
exactly what _Project Lombok_ does.

Lombok is a beautiful Indonesian island not far from the island of
Java (see the title image). Project Lombok defines a bunch of
annotations that generate so-called boilerplate code: getters,
setters, constructors, `equals()` and `hashCode()`—this kind of
stuff. They do it by taking the next step in the annotation processor:
after getting access to the program’s AST, they use undocumented,
internal Sun compiler API to directly modify AST. Fortunately, most
everyone happens to use Sun’s (now Oracle’s) `javac`, and the entire
thing happens to work quite well in practice.

The Java community appears to be split on Project Lombok. Some
rightfully consider it to be one giant hack: a cardinal sin for the
language and the community obsessed with defining interfaces,
separating access, and hiding private methods. Others applaud it as a
great hack making source code shorter, cleaner, more declarative, more
maintainable: in essence, improving the source language.

Though this obviously is not the only application of code writing
code, to me it’s been a Holy Grail of all the techniques we reviewed
so far: extend the language to make it more powerful, easier to use,
more adopted to the problem domain.

And the fact is, Project Lombok manages to do just that. It is indeed
code writing code, moreover, Java writing Java: using the same
language for metaprogramming rightfully feels powerful. I am still
excited by it.

Unlike most other annotations uses, that seem to be within the context
of this or that framework, Lombok is not a framework. Its annotations
do not add new functionality to the annotated code; they just make it
more concise and maintainable.

For example, consider `@Data`, one of the most powerful annotations in
Project Lombok.

```java
@Data
class Point {
    int x;
    int y;
}
```

As its documentation
[states](https://projectlombok.org/features/Data), “`@Data` generates
all the boilerplate that is normally associated with simple POJOs
(Plain Old Java Objects) and beans: getters for all fields, setters
for all non-final fields, and appropriate `toString`, `equals` and
`hashCode` implementations that involve the fields of the class, and a
constructor that initializes all final fields”. This pretty mechanical
code that you are somehow _supposed_ to write yourself (if not for
Project Lombok), tens of lines of it even for a trivial class like
this, not only would completely obscureq the meaningful parts of
`Point` and make it next to impossible to read. It is also a few more
functions to test. And if you don’t test them, because they are boring
and trivial,—well, then don’t be surprised when you decide to make
your Point three-dimensional and add a `z` coordinate that you forget
to update `hashCode()` and get some weird behavior of your Points in a
HashMap.

Every time you have to write lines of code irrelevant to the task at
hand, it is a sign of a low-level language. By saving you from having
to write it, Project Lombok elevates Java, makes it higher-level and
better language.

However, there a huge “but”, of course,—even a few. Even though
Project Lombok is _implemented_ in Java, the actual language of
defining and specifying annotations is rather weak, with poor type
system no loops and conditionals, and no ability for different
annotations to inter-operate. 

Lombok’s annotation implementations live in a completely separate
project, very far from the source code being modified. And boy, aren’t
they complex!  Just take a quick look at this
[blog post](http://notatube.blogspot.com/2010/12/project-lombok-creating-custom.html)
describing how to add a `@HelloWorld` annotation to Lombok. Count how
many screens takes something that would only require a few lines in
Lisp or Python.

The complexity is partially driven by the hackish nature of the
project, but partially it is also the nature of the beast: compiler
plugins are hard, and Java syntax and its AST are not trivial,
either. Not to mention the need for Project Lombok to inter-operate
with IDE (somehow, Java programming is unthinkable without an IDE these days).

Either way, the takeaway is the same: even though Lombok’s
annotations are super-useful (I firmly belong to that camp), their
implementation is not for the faint of heart. Developers are unlikely
to casually add them on the day to day basis the way they do in Lisp,
Python—even in C preprocessor, for that matter. Lombok will remain its
own beautiful island, separate from the island of Java.

Interestingly enough, good ideas keep cross-pollinating
languages. Just as the Python decorator syntax was influenced by Java,
Lombok’s annotations seem to be influenced by Python decorators. And
now, Python 3.7 (the latest version of Python)
[introduced](https://www.python.org/dev/peps/pep-0557/) a standard
`@dataclass` decoration, very similar to Project Lombok’s `@Data`
annotation we have just discussed.

This proves once again the broad cross-language appeal of using code
writing code to get read of mundane, “boilerplate” code. However some
languages—and language cultures—clearly produce much more of that than
the others. The language that produced Project Lombok is probably one
of the worst offenders. Some of it is cultural, and some is caused by
the lack of language power. For example, coming back to the getter and
setter functions: I mentioned that they are not used in a properly
Pythonic code, and the reason is that the language i more powerful. It
has the concept of _properties_, which allow to define getters and
setters completely transparently, and only when they do something
nontrivial: that is, _when they are not boilerplate._

## * * *

And here, on this unsatisfactory note, I have to end my story. Yes,
Java has come a long way, but in terms of code writing code it not
just haven’t caught up with the Lisp of _fifty years ago:_ it is still
light years behind. For me, I’ll hold my enthusiasm for some younger
and bolder languages, like Rust, that appears to have a well-developed
[macro capability](https://doc.rust-lang.org/book/ch19-06-macros.html).

What are your favorite examples of metaprogramming? Let me know! And
as I said, my C++ and Lisp are somewhat rusty at this point, and I am
just starting to re-learn Java. If I got something about these
languages wrong, let me know too.
