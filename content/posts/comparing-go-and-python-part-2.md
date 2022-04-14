---
title: "Comparing the Basics of Go and Python for New Programmers -- Part 2"
tags:
  - python
  - go
  - series
  - series-go-vs-python
publishdate: "2022-04-14"
---

Part One of this series comparing [Go and Python can be found here][part1].

In Part One, I covered some of the differences between Python and Go when it
comes to packaging and looping.  This time, I want to cover a (slightly?) more
advanced topic: blocking vs. non-blocking HTTP calls.  To do this, we'll write
the same application in both Python and Go.  This tiny demo application will
retrieve the names of the original 150 Pokemon from a public API.

> Technically, Mew is #151 and is "original."  But we're going to leave Mew
> out today.

## Getting Pokemon One at a Time

First, we'll write our application to make its API calls in serial.  This is
also known as "blocking" code: the code cannot continue until the previous
call is completed because it's waiting for network I/O.

### Python

For our Python implementation, we'll ues the very popular (and with good
reason) [requests][requests] library.  I don't want to fight with virtualenv,
so I'm going to use this `Dockerfile`:

```docker
FROM python:3.8-slim-buster

WORKDIR /app
RUN pip3 install requests
COPY app.py .

CMD ["python3", "app.py"]
```

Our Python code for this is:

```python
#!/usr/bin/env python
import requests
import time

def main():
    # Loop from 1 to 150
    for number in range(1, 151):
        pokemon_url = f'https://pokeapi.co/api/v2/pokemon/{number}'
        # Perform an HTTP GET on the Pokemon API
        response = requests.get(pokemon_url)
        # Parse the response as JSON, get the "name" key, and print it
        print(response.json()["name"])

if __name__ == "__main__":
    start_time = time.time()
    main()
    print("Total Time: %s Seconds" % (time.time() - start_time))
```

We can build and run it with:

```bash
$ podman build -t blog .
$ podman run --rm -it blog

<SNIP -- A whole lotta Pokemon!>

Total Time: 25.06453824043274 Seconds
```

> Note: you can replace `podman` with `docker` if you're a Docker user.

Okay, 25 seconds to get all of the Pokemon.  Well, all of the ones that matter,
anyway.

> I will fight you.  There are only 150 Pokemon.  Plus Mew.

### Go

Let's have a look at doing the same thing in Go:

```go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"strconv"
	"net/http"
	"time"
)

func main() {
	startTime := time.Now()
	// Create an http.Client with a 2 second timeout, otherwise it will
	// never timeout
	client := http.Client{
		Timeout: time.Second * 2,
	}
	// Loop from 1 to 150
	for i := 1; i < 151; i++ {
		pokemon_url := "https://pokeapi.co/api/v2/pokemon/" + strconv.Itoa(i)
		// Create a map to unmarshal (parse) JSON into
		// We create a map that contains type `interface{}` so we don't
		// have to define the structure of the JSON
		var pokemon map[string]interface{}

		// Perform an HTTP GET on the Pokemon API
		res, _ := client.Get(pokemon_url)
		defer res.Body.Close()
	
		// Read the response and unmarshal (parse) it into our map
		body, _ := ioutil.ReadAll(res.Body)
		json.Unmarshal(body, &pokemon)

		fmt.Println(pokemon["name"])
	}
	fmt.Printf("Total Time: %s\n", time.Since(startTime))
}
```

And then, let's run it:

```bash
$ go run main.go

<SNIP -- A whole lotta Pokemon!>

Total Time: 6.877103447s
```

Both of these applications do roughly the same thing: they iterate over a
sequence of numbers from `1` to `150`, using those numbers to perform an
HTTP GET against the Pokemon API.  That API endpoint returns a bunch of data
as a JSON response, so each application must also parse JSON and turn it into a
native data structure -- a `dict` for Python and a `map` for Go.

Run both the Python and Go version a few times and record their total times.
This will provide a baseline of performance so that we can understand the
gains from executing multiple requests "at the same time."

> Okay, I'm not even going to lie here: I actually thought I'd see comparable
> time taken.  I tried to put it in the same container that the Python app
> runs in just to be sure it wasn't a weird thing about my environment, and...
> it only added half of a second.  I _seriously_ contemplated binning this
> post because of how much faster the Go implementation was because I knew it
> would make the concurrency gains seem so tiny for Go compared to Python.

## Refactoring to Catch 'Em All!

Ok, so our serial API calls are costing us quite a bit of time.  Our Python
codebase is taking around 25 seconds, and our Go application is clocking in at
6 or 7 seconds.  We'll start by refactoring our Python code to make multiple
API calls "at the same time."

> "At the same time" is misleading, since we'll be using asynchronous
> programming and not parallel programming.  Basically, HTTP calls will be
> fired off and then "sent to the background."  While they're blocking on
> things like network I/O, others will be kicked off.  At the end, the results
> are gathered up and handled syncrhonously.

### Python

When it's time to reap the rewards of asynchronous programming in Python, we
run into our first problem: the very popular and frequently-used `requests`
library doesn't really support Python's `asyncio`.  This means that if you want
any sort of ability to do multiple HTTP requests simultaneously, you need to
either write multi-threading or multi-processing code or use a library other
than `requests`.  I'm going to take the latter option, because multi-processing
in Python is more frustrating than `asyncio`.

> Spoiler: I picked `requests` on purpose, knowing it didn't support `asyncio`.
> I did it to intentionally exaggerate the comparison between Python and Go,
> but I _also_ did it to showcase a very real problem when trying to add
> asynchronous code later in your application when you identify a bottleneck:
> it can be tricky and painful.

The package we'll use is `aiohttp`, which is an `asyncio`-compatible HTTP
library.  To adapt our code, we will now need to...completely rewrite our
code and introduce new Python keywords in front of our function definitions and
context managers.  If you've never used `asyncio`, be prepared for your Python
code to look a whole lot less elegant.

```python
#!/usr/bin/env python
import aiohttp
import asyncio
import time

# Use the "async" Python keyword to tell Python that this function is async
async def main():
    # Use the "async" keyword to create an async-compatible context manager
    async with aiohttp.ClientSession() as session:
        for number in range(1, 151):
            pokemon_url = f'https://pokeapi.co/api/v2/pokemon/{number}'
            # Use the "async" keyword to create an async-compatible context
            # manager and HTTP GET request
            async with session.get(pokemon_url) as response:
                # Block the request until we have received the response and can
                # parse it as JSON using the `await` Python keyword
                pokemon = await response.json()
                print(pokemon["name"])

if __name__ == "__main__":
    start_time = time.time()
    # Run the `main` function in the `asyncio` loop
    asyncio.run(main())
    print("Total Time: %s Seconds" % (time.time() - start_time))
```

Update your `Dockerfile`:

```docker
FROM python:3.8-slim-buster

WORKDIR /app
RUN pip3 install aiohttp
COPY app.py .

CMD ["python3", "app.py"]
```

And rebuild and run your newly asynchronous Python code:

```bash
$ podman build -t blog .
$ podman run --rm -it blog

<SNIP -- A whole lotta Pokemon!>

Total Time: 7.189948081970215 Seconds
```

As you can see, it's a _lot_ faster -- around 350% faster!  But to get this
huge performance gain, we had to either know in advance that our script or
application would need us to implement some sort of simultaneous coding or we'd
have to do a whole lot of work refactoring and picking different packages.
These sorts of gains are even more obvious the longer you're blocked on network
I/O -- say, when you're configuring network devices over SSH 100ms away.

> It's important to note here that it is possible to make the Python code
> perform even faster by refactoring even more.  In fact, the problem is that
> this code isn't _actually_ all that concurrent and has a sneaky, deceptive
> bug caused by a naive first pass at the implementation.  I'll show how to
> fix this at the _very_ end of the blog post, after all of the conclusions,
> just to be fair.  However, I was very much trying to tackle the "easiest" way
> to refactor for concurrency in Python -- it gets even more complex when you
> want to get better performance (especially when compared to the simplicity
> of Go).

### Go

Look, I thought it wasn't going to be worth showing the time savings in Go when
I started writing the code.  I mean, our serial, synchronous, "slow" Go
implementation was already as fast as our asynchronous "fast" Python code!  But
I've gotta say -- it's hilariously worth it.  Enough with the waiting, though.
On to the code!

```go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"strconv"
	"net/http"
	"time"
	// Import the sync package so we can wait for all requests to finish
	// before exiting the program
	"sync"
)

func main() {
	startTime := time.Now()
	client := http.Client{
		Timeout: time.Second * 2,
	}
	// Create a WaitGroup, effectively treated as a queue that must finish
	// before we can exit
	var wg sync.WaitGroup
	for i := 1; i < 151; i++ {
		// Add one item to the WaitGroup ("queue") per iteration
		wg.Add(1)
		// Reset the `i` var to prevent scoping issues
		// (this is covered in the Go FAQ)
		i := i
		// Create an anonymous function/closure and run it as a
		// goroutine
		go func() {
			// Remove one piece of work from the WaitGroup "queue"
			// _after_ the rest of the anonymous function finishes
			defer wg.Done()
			pokemon_url := "https://pokeapi.co/api/v2/pokemon/" + strconv.Itoa(i)
			var pokemon map[string]interface{}

			res, _ := client.Get(pokemon_url)
			defer res.Body.Close()
	
			body, _ := ioutil.ReadAll(res.Body)
			json.Unmarshal(body, &pokemon)

			fmt.Println(pokemon["name"])
		}()
	}
	// Block until the WaitGroup is decremented to 0 (no items left in the "queue")
	wg.Wait()
	fmt.Printf("Total Time: %s\n", time.Since(startTime))
}
```

So, what changes were really made?  Well, we made our code asyncrhonous by
only doing one thing: we wrapped our logic in an anonymous function (the
`func(){}`; told it to run as a [goroutine][goroutine] by typing `go` in front
of it;  and told it to execute immediately by adding the `()` to the end.  In
other words, to make our code asynchronous, we did this:

```go
go func() {
	// our original code here
}()
```

The rest of the changes -- adding the `sync` import, all of the `wg` lines, and
re-defining the `i` var (for reasons [explained in the FAQ][faq]) were to
resolve some "bugs" or "unexpected behaviors."  In particular, the `sync` and
`wg` packages let us perform our network-based calls asynchronously but then
wait for all of them to finish before exiting the program.

While you could argue that adding `go func(){}()` around code isn't all that
different from adding `async` in front of everything in Python, and that using
`sync.WaitGroup` isn't all that different from `await` in Python, the bigger
picture here is that we didn't swap out our HTTP client library.  We didn't
have to meaningfully rewrite any of our Go code to achieve asynchronous code.

This is a _huge_ difference between Python and Go: you can't assume that Python
code and libraries support async or non-blocking activities, but with Go, you
usually can.  If something is synchronous or blocking in Go in a way that is
meaningful, you'll usually find that it has the word `sync` in the package
or function.  This doesn't mean that asynchronous Go code is bug-free -- you
still have the problem of locking that any language does.  If two things try
to modify the same file on disk at roughly the same time, you still need to
deal with file locks.  But that's true in other languages, too.

Okay, enough talking about how much easier it is to refactor code to be
concurrent in Go.  Let's see if we actually get similar performance gains in
Go as we did in Python!

```bash
$ go run main.go

<SNIP -- A whole lotta Pokemon!>

Total Time: 763.827633ms
```

...I told you it was going to be hilarious.  If we round a bit and say that our
original Go code averaged 7 seconds and our new, concurrent code took around
800ms on average, we have a performance gain of **875%**.  Based on that, we
can also say that our concurrent Go code is _also_ around 875% faster than our
concurrent Python code.  If we consider the extremes -- our slow Python code
and our fast Go code -- we find that the concurrent Go code is an absolutely
staggering 3125% faster than our syncrhonous Python code.  Obviously not a fair
comparison, but hopefully this shows how some very simple changes in Go can
vastly improve performance -- whereas in Python the gains are _less_ in terms
of percentage while being more challenging to implement than in Go.

## Conclusions

I'm regularly surprised by Go.  I try not to buy into hype, which frequently
leads me to be pretty late to the game on things.  I'm a slow burn, I don't
chase "new shiny," and I prefer some maturity in the technologies I dedicate my
time to learn.

The idea for this blog post started before I had any idea how any of the code
would perform.  I already knew that concurrent would be faster than
syncrhonous, and I expected _some_ performance gains in Go compared to Python.
I didn't expect it to be such a severe difference.  It's possible that
something about my environment contributed to how slow the Python code ran.
I tried to eliminate that by running the Go code in the exact same container as
the Python code, but that only made a difference of around 500ms on the
synchronous code, so I didn't even bother trying for the concurrent version.

I almost stopped writing this blog post.  I wrote the synchronous Go code, and
it was so much faster than the Python code that I thought rewriting it to be
concurrent would only shave one or two seconds off -- maybe three.  And then I
wrote the async Python code and became excited again: I knew it would be
faster, but it had caught up to the synchronous Go code.  Since I figured the
concurrent Go implementation would only be one or two seconds faster, I thought
it would be an interesting showcase that concurrent Go was only slightly faster
than concurrent Python.  And then I ran it.  And then I was convinced it was a
fluke, so I ran it five more times.  Then I thought I must have had a bug.
Maybe somehow my time calculation was off, so I grabbed an external clock.  It
was the same (within reason -- I'm not a robot, I can't react within
miliseconds).  So then I thought that somehow it wasn't fetching all 150
Pokemon, so I counted them.  150.

Python performed according to my expectations, which makes sense since I've
been writing Python in some capacity for the past 8 years or so.  But Go?  Go
shattered my expectations in an amazing way.  Is Go the perfect language?
Nope, definitely not.  It has some things about it that I hate, like having
to read an HTTP body and parse it as JSON (or "unmarshal" in Go-speak) as two
separate instructions.  Remembering to `defer response.Body.Close()` is also
kind of annoying.  I expect that these are things that will just become natural
to me as I gain more experience with the language, similar to remembering how
nested comprehensions in Python work.

But is Go a great language?  Yes, hands-down.  Every bit of Go code in this
blog post is part of the standard Go library.  Go is intended to be concurrent
by default, making it extremely easy to take syncrhonous code and make it
asynchronous.  Go gives me most of the batteries that I need most frequently
and only makes me go to the store when I need "D" batteries.  Python, on the
other hand, feels like it's giving me around half of the most common batteries
-- they give me "AA"s, and "9V"s, and (for some reason) "D"s, but then it tells
me `go fish` for "AAA"s and "CR2032"s.

As a final note, I'd like to mention that the idea for using the Pokemon API
wasn't mine.  In fact, a lot of the async Python code isn't even originally
mine!  I was plumbing the depths of the Internet trying to find a simple, easy
snippet of Python `asyncio` + `aiohttp` that would be able to scale quite
well both up and down in terms of number of requests.  I wanted something that
would be simple and easy to follow, but that a reader could easily adjust
simply by changing the number of loop iterations.  I came upon a
[Twilio blog post][twilio-post] that checked all the boxes and used the Pokemon
API, which scales admirably well since it's simple and has hundreds of Pokemon.
If you want to see the impacts with fewer API calls, just tune the number of
iterations in the loop down.  Want more?  There are 898 Pokemon at the time of
this writing, each available via the Pokemon API (though you might need better
error handling and/or rate-limit checking to get that high).

## One More Thing

If you're curious, I tested all 898.  In Go, I averaged around 1.5 to 3 seconds
to catch 'em all.  In our original async Python code , it was a (much more)
painful 35 seconds...which, honestly, still a lot better than if it was done in
serial!  But we can, in fact, do better if we refactor just a little bit and
introduce a little more complexity.  After all, I do want to be fair to Python.

For this, we're going to refactor the code (again) to pull out the call actual
`HTTP GET`s into their own `async` function.  We do this because our `await` is
actually a problem in our code right now: it's sneakily blocking and isn't
_quite_ giving us the full benefits of concurrency.  See the new and improved
code below!

```python
#!/usr/bin/env python
import aiohttp
import asyncio
import time

# Refactor our HTTP GET code to deal with the sneaky blocking nature of
# await in the previous code.
async def get_pokemon(session, url):
    async with session.get(url) as resp:
        pokemon = await resp.json()
        print(pokemon['name'])

async def main():
    async with aiohttp.ClientSession() as session:
        # Create a "task queue", similar to our Go code's sync.WaitGroup
        tasks = []
        for number in range(1, 899):
            pokemon_url = f'https://pokeapi.co/api/v2/pokemon/{number}'
            # Add an item to the task queue, similar to our use of
            # wg.Add(1) in Go, except it's an actual function call instead
            # of just a number.
            tasks.append(
                get_pokemon(session, pokemon_url)
            )
        # Wait for everything to finish, similar to our Go code's wg.Wait()
        await asyncio.gather(*tasks)

if __name__ == "__main__":
    start_time = time.time()
    asyncio.run(main())
    print("Total Time: %s Seconds" % (time.time() - start_time))
```

This gives me a much more respectable average of 3-5 seconds.  The Go code is
still noticeably faster, but the difference is demonstrably smaller.
Interestingly, if we compare this to our Go implementation, we can see that it
is incredibly similar in its ideas.

So, did I give Python an unfair disadvantage?  Possibly.  But I think that in
the end, this really shows how Python's implementations aren't "obvious."
I had to refactor twice: once for the seemingly obvious concurrency approach,
and again to deal with a non-obvious blocker, each time making my code slightly
less readable and introducing new patterns.  In Go, there was really only one
way to accomplish this, and it was incredibly obvious.  Once again, Go is
better at the Zen of Python than Python is.

Finally, there's a lot about concurrency in Python that changes quite
frequently over time.  There's an absolutely [phenomenal blog post][pyblog]
that goes into a _lot_ of depth on Python concurrency.  In it, there are
several notes about how things have evolved from one minor version to another
in a way that changes what the best practices are for code written in those
versions.  I'm happy for improvements, but one of the things I like about Go
is that the fundamentals don't change all that much from one version to the
next.

[part1]: /posts/comparing-go-and-python/
[faq]: https://golang.org/doc/faq#closures_and_goroutines
[requests]: https://docs.python-requests.org/
[goroutine]: https://gobyexample.com/goroutines
[twilio-post]: https://www.twilio.com/blog/asynchronous-http-requests-in-python-with-aiohttp
[pyblog]: https://www.integralist.co.uk/posts/python-asyncio/
