---
layout: post
title: "Contiguous stacks in Go"
date: 2014-03-25 11:51
---

Lately I've been messing with Go and I really like it. The 1.3 release is [due in June 2014](http://talks.golang.org/2014/go1.3.slide#2) and among other [performance improvements](https://docs.google.com/document/d/1s6MxBsLyBG45SRS60a-aM01DmZ4hD1nMzGAKdTopKGY/edit), the runtime will implement [*contiguous* stacks](http://golang.org/s/contigstacks). Let's see what this is about.

## Segmented stacks

The 1.2 runtime uses *segmented stacks*, also known as *split stacks*.

In this approach, stacks are discontiguous and grow incrementally.

Each stack starts with a single *segment*. When the stack needs to grow, another segment is allocated and linked to the previous one, and so forth. Each stack is effectively a doubly linked list of one or more segments.

![Segmented stacks in Go](/assets/segmented-stacks.png)

The advantage of this approach is that stacks can start small and grow or shrink as needed. For example they start at 4kb in 1.1 and [8kb in 1.2](https://codereview.appspot.com/14317043/#ps6001).

However a certain issue arises.

Imagine a function call happening when the stack is almost full. The call will force the stack to grow and a new segment will be allocated. When that function returns, the segment will be freed and the stack will shrink again.

Now imagine that call happening very frequently:

{% highlight c %}
func main() {
    for {
        big()
    }
}

func big() {
    var x [8180]byte
    // do something with x

    return
}
{% endhighlight %}

The call to `big()` will cause a new segment to be allocated, which will be freed upon return. Only this will happen repeatedly since we are inside the loop.

In such cases where the stack boundary happens to fall in a tight loop, the overhead of creating and destroying segments repeatedly becomes significant. This is called the "hot split" problem inside the Go community. Rust folks call it ["stack thrashing"](https://mail.mozilla.org/pipermail/rust-dev/2013-November/006314.html).

## Contiguous stacks

The "hot split" will be addressed in Go 1.3 by making the stacks *contiguous*.

Now when a stack needs to grow, instead of allocating a new segment the runtime will:

1. create a new, somewhat larger stack
2. copy the contents of the old stack to the new stack
3. re-adjust every copied pointer to point to the new addresses
4. destroy the old stack

What's interesting here is the re-adjusting part.

As it turns out, that part heavily depends on the compiler's escape analysis which provides us with an important invariant: *the only pointers to data in a stack, are in that stack themselves* (there are some exceptions though). If any pointer escapes (eg. the pointer is returned to the caller) it means that the pointed data are allocated on the heap instead.

There is a certain challenge though. The 1.2 runtime doesn't *know* if a pointer-sized word in the stack is an actual pointer or not. It is possible that there are floats and most rarely integers that if interpreted as pointers, would actually point to data.

Due to the lack of such knowledge the garbage collector has to *conservatively* consider all the locations in the stack frames to be *GC roots*. This opens the possibility for [memory leaks](https://groups.google.com/forum/#!topic/golang-nuts/5BB4bnsJrvA), especially on 32-bit architectures since their address pool is much smaller.

When copying stacks however, such cases *have* to be avoided and only *real* pointers should be taken into account when re-adjusting.

[Work was done though](https://docs.google.com/document/d/1lyPIbmsYbXnpNj57a261hgOYVpNRcgydurVQIyZOz_o/pub) and [information about live stack pointers](https://docs.google.com/document/d/13v_u3UrN2pgUtPnH4y-qfmlXwEEryikFu0SQiwk35SA/pub) is now embedded in the binaries and is available to the runtime. This means not only that re-adjusting all stack pointers is now possible, but also that the collector in 1.3 can [*precisely*](http://talks.golang.org/2014/go1.3.slide#4) collect stack data.

The initial stack size will be conservatively set to 4kb in 1.3 but will probably be further reduced in 1.4. As for the shrinking strategy, when the collector runs, stacks using less than 1/4 of their size will be shrinked by half.

Even though *contiguous stacks* cause a trivial amount of memory fragmentation, benchmarks comparing both implementations using `json` and `html/template` show some nice performance improvements.

![json benchmark contiguous vs segmented stacks](/assets/json-benchmark.png)
![html/template benchmark contiguous vs segmented stacks](/assets/html-benchmark.png)

<div style="text-align: center; font-size: 15px;">
  source: <a href="https://docs.google.com/document/d/1wAaf1rYoM4S4gtnPh0zOlGzWtrZFQ5suE8qr2sD8uWQ/pub" target="_blank">contiguous stacks design document</a>
</div>

## Conclusion

Go 1.3 is going to be a great release with a lot of [performance improvements](http://talks.golang.org/2014/go1.3.slide#7) and [other great things](http://talks.golang.org/2014/go1.3.slide#16) going on. I'm [looking forward](http://isgo1point3.outyet.org) to it.

[There's an interesting discussion going on in Reddit](http://www.reddit.com/r/rust/comments/21ffem/contiguous_stacks_in_go/).
