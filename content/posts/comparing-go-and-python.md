---
title: "Comparing the Basics of Go and Python for New Programmers -- Part 1"
tags:
  - python
  - go
  - series
  - series-go-vs-python
publishdate: "2022-04-02"
---

I'm not trying to start a flame war, I swear.  So let's start off with this:
if you're already fluent in either Python or Go, this blog post isn't meant
to change your mind.  Instead, it aims to compare some fundamental operations
in each language and talk about how I feel about them.  This is only Part 1.
If it's well-received, I'll write more.  If it's not, I won't.


> The title of this post includes the words `for New Programmers`, but it does
> expect the reader to have at least a passing familiarity with programming --
> or at least be able to understand pseudo-code.

## Packaging and Distribution

This is possibly the most obvious comparison one could make, but it's here
because it's an important one.

In Python, there is the [Zen of Python][zen-of-python], an actual PEP (Python
Enhancement Protocol) that aims to succinctly guide Python developers on their
path.  One of the aphorisms is:

> There should be one-- and preferably only one --obvious way to do it.

Unfortunately, Python falls down repeatedly in the face of this guiding
principle.  One need look no further than managing dependencies and/or
building packages.  You can use `pip` to install your dependencies pretty
easily.  This seems fine on the surface.  But then you _very_ quickly lose
track of what dependencies your application has, making it challenging to
share with others.  Ok, there's a solution for that: just run
`pip freeze > requirements.txt`, and then others can install your dependencies
with `pip install -r requirements.txt`.  But oh no, you didn't use a dedicated
["virtual environment"][virtualenv], so now your list of dependencies includes
every Python package you've ever installed, even though more than half of them
may be unrelated to your current project!

Great, so we can solve that problem with virtual environments.  Now there are
more tools to layer on top, because managing virtual environments is its own
whole thing.  Do you use [Poetry][poetry] or [Pipenv][pipenv]?  Even once you
figure that out, you run into another snag -- if you want to publish your
package on PyPI, the default Python Package Index, you need to write a
[`setup.py`][setup-py], and _that_ has its _own_ way of listing dependencies!
Oh, did I mention that when you go the Poetry or Pipenv route, they have their
own way independent of `requirements.txt`?  You might even pass down some
dependencies to the end user that are non-obvious, such as needing to have the
OpenSSL development libraries installed in order to even install a Python
package.  Honestly, it's all confusing and exhausting.  There is not one
obvious way of packaging, especially when it comes to dependencies.

Go, on the other hand?  Your dependencies are straightforward: you `import`
a package by specifying its URL in your code.  For example:

```go
import (
	"fmt"
	"gopkg.in/yaml.v3"
)
```

In this tiny snippet, we import the built-in `fmt` package and a package from
an external source, `gopkg.in/yaml.v3`.  And if you're wondering, you can open
your browser to the exact package you specified as an import; just add
`https://` to the beginning (e.g., [`https://gopkg.in/yaml.v3`][yamlv3]).

And you "fetch" or _get_ the external module with the `go get` command:
`go get gopkg.in/yaml.v3`.  When you later build your Go project, it gets
distributed as a single binary.  This means that _even if_ there were multiple
ways to get your dependencies, you're not passing the burden down to your end
users.  Oh, and the end user doesn't even need Go installed to use your
application, unlike Python which requires the end user to have Python
installed...and deal with virtual environments and the management thereof.

So the Zen of Python tells us that there should be one obvious way, and then
Go does a better job of this (at least in the realm of package and dependency
management).  Testing, formatting, and even looping are other areas where Go
does this much, much better.

## Looping

Speaking of looping: Go's looping is more intuitive than Python's.  Python has
multiple loop types -- `for` and `while` -- while Go has only `for`.  It's
deeper than this, though.  Let's compare two relatively similar loops through
a list of fruits:

```python
fruits = [
    "apple",
    "orange",
    "watermelon"
]

if __name__ == "__main__":
    for fruit in fruits:
        print(fruit)
```

The above Python is pretty readable: we just iterate through a list of fruits
and print them out to the console.

> I'm not going to even touch on the use of the "dunders."  They're their own
> whole can of `but why`.  You can read more about `__main__` and `__name__`
> [here][dunder-main] if you really want.

```go
package main

import "fmt"

func main() {
	fruits := []string{
		"apple",
		"orange",
		"watermelon",
	}

	for _, fruit := range fruits {
		fmt.Println(fruit)
	}
}
```

This Go application does the same thing as the preceeding Python application.
There are a few important differences, though:

* Go requires you to specify a type for the "list" (in this case, it's a
  `slice` in Go terminology)
* Go requires you to add a comma to the end of the last element when defining
  a multi-line construct, an opinionated behavior that can saving you from the
  annoyance of searching for a syntax error because you forgot to add a comma
  before adding a new item
* Go has a built-in way to "print", but it's part of the built-in `fmt` package
  instead of being a global name
* Go's iteration of collections uses the built-in `range` keyword, which
  generates two values instead of Python's one value
* We can discard the first value (the index) by using an underscore (`_`), a
  pattern seen in plenty of Python codebases as well.

> Incidentally, one of the other lines in the Zen of Python is
> `Namespaces are one honking great idea -- let's do more of those!`, which...
> it then goes on to violate by having basic printing be a global keyword (but
> other kinds of writing, such as to a file or logging, is in a package).

At first glance, this seems like it's quite a bit more work for the same thing.
But what if you actually need the index when iterating through your list/slice?
Well, in Go, it's literally the same code -- just replace the `_` with a
variable name, such as `i` or `index`.  In Python, though, the code is
different.  It's:

```python
fruits = [
    "apple",
    "orange",
    "watermelon"
]

if __name__ == "__main__":
    for index, fruit in enumerate(fruits):
        print(fruit)
```

> I'm not doing anything with the index in this example (which would be an
> error in Go) to keep this simple.

This actually required a PEP -- [PEP 279][pep-279] -- to introduce the
functionality!  Definitely non-obvious.

Okay, let's say you can get past this.  Let's talk about iterating through the
Python `dict` (or the Go `map`).  We'll just print the keys and values.  In
Python first:

```python
foods = {
    "fruits": [
        "apple",
        "orange",
        "watermelon"
    ]
}

if __name__ == "__main__":
    for k, v in foods.items():
        print(k, v)
```

Next, in Go:

```go
package main

import "fmt"

func main() {
	foods := map[string][]string{
		"fruits": {
			"apple",
			"orange",
			"watermelon",
		},
	}

	for k, v := range foods {
		fmt.Println(k, v)
	}
}
```

> Don't let the curly braces (`{` and `}`) in the creation of the `slice` (or
> `list` in Python-speak) confuse you.  Go doesn't define different types by
> using different character syntaxes (i.e., in Python you make a list by
> surrounding values with `[]` and when making a `dict` you use `{}`.  But in
> Go, you do this as part of the type definition (i.e., `map`, and the `[]`
> _before_ the type).

In Go, the way we create the `map` (or `dict` in Python-speak) is a little more
verbose because Go is a static-typed language (whereas Python is "duck-typed").
But this is consistent.  A `map` is a `map` is a `map`.  The important thing
here is that iterating over a `map` is the same as iterating over a `slice` in
Go, unlike Python.  To reduce this down to the things that matter, let's look
at _just_ these three "kinds" of iteration in both Python and Go:

```python
# Iterate through a list of fruits and print them
for fruit in fruits:
    print(fruit)

# Iterate through a list of fruits and print the index number and fruit
for index, fruit in enumerate(fruits):
    print(index, fruit)

# Iterate through a dict and print the key and value:
for key, value in foods.items():
    print(key, value)
```

```go
// Iterate through a slice of fruits and print them
for _, fruit := range fruits {
	fmt.Println(fruit)
}

// Iterate through a slice of fruits and print the index number and fruit
for index, fruit := range fruits {
	fmt.Println(index, fruit)
}

// Iterate through a map and print the key and value:
for key, value := range foods {
	fmt.Println(key, value)
}
```

In Python, I need to learn about the global `enumerate()` function if I want
the index of a list.  If I want to iterate through key:value pairs in a dict,
I need to learn about the dict's `.items()` method (why isn't it a global
function like `enumerate()` for lists?).  In Go, the way I do all three of
these is exactly the same.  I have to remember less, and when I read the code,
it's immediately _far_ more obvious.  Yet again, Go does better.  And we
haven't even addressed Python's comprehensions, another way of
iterating/filtering collections that is somehow Pythonic but is also just more
"stuff" you have to understand when you read the code.

## Conlusion and Final Thoughts

To be honest, if I were talking to someone who had never done any programming
at all, I'd probably tell them to learn the fundamentals in Python and then
immediately switch to immersing his or her self in Go.  Go is a little harder
to learn initially, but saves you from countless pitfalls that you might find
in Python.  Conversely, Python is _much_ quicker to pick up, but isn't _really_
all that opinionated (despite the Zen of Python).  That lack of opinionated
implementation causes quite a few pain points that make some parts of Python
feel "bolted on" or "slapped together."  Looping is one of the biggest areas
where this is felt the most for me, but there are _plenty_ of others that we
may explore in future posts.

So, what are the "fundamentals" as I see them?  If you're a new programmer,
here's a list of things that _I_ think you should learn in Python first, and
then immediately switch to Go.

* Strings and numbers
* Lists
* Dicts
* Functions
* Conditionals (`if` statements)
* Loops (specifically, the `for` loop)

Learning the above in Python can give a new programmer fast feedback loops
without getting caught up on things like "types", "pointers", etc.  This will
help them reason about the logical building blocks of code.  By switching to
Go at this point, a new programmer can start learning a language that will
save them a lot of headache and frustration in the future with a basic
understanding of programming building blocks.

If I'm being honest, this is the comparison I wish existed (or that I knew
about) three or four years ago.  I would've started my journey with Go a lot
sooner.

> One more important note: I still write a _lot_ of Python.  In fact, the vast
> majority of the code I write is currently Python.  I'll probably never _stop_
> writing Python.  It's the right language for a lot of things.  But it's also
> a language that almost daily makes me ask `whyyyyyyyyy`.

[zen-of-python]: https://peps.python.org/pep-0020/
[virtualenv]: https://docs.python.org/3/tutorial/venv.html
[poetry]: https://python-poetry.org/
[pipenv]: https://pipenv.pypa.io/en/latest/
[setup-py]: https://docs.python.org/3/distutils/setupscript.html
[yamlv3]: https://gopkg.in/yaml.v3
[dunder-main]: https://docs.python.org/3/library/__main__.html
[pep-279]: https://peps.python.org/pep-0279/
