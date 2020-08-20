---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Oh Python"
subtitle: ""
summary: ""
authors: []
tags: ["python"]
categories: []
date: 2020-08-20T13:16:40+01:00
lastmod: 2020-08-20T13:16:40+01:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
Suppose you have a list of objects that you need to iterate over two consecutive items at a time. 

An old [stackoverflow](https://stackoverflow.com/questions/16789776/iterating-over-two-values-of-a-list-at-a-time-in-python) question for this leads to the a quote from the [documentation](https://docs.python.org/2/library/functions.html#zip) that reads:

> This makes possible an idiom for clustering a data series into n-length groups using `zip(*[iter(s)]*n)`.

So the solution would be:
```(python)
k = [1, 2, 3, 4, 5, 6]
list(zip(*[iter(k)]*2))
# [(1, 2), (3, 4), (5, 6)]
```
That is cryptic! Let's break it down to understand why this works.

1. Let's start with the inner most bit. `iter` simply returns an iterator object. For lists we would normally just write `for x in alist` to iterate over the list, but under the hood an iterator is defined with each loop fetching the next item using a `next` call. 
```(python)
>>> iter(k)
<list_iterator object at 0x7fcf654c9f28>
```

2. Next we consider `[iter(k)]*2` - the multiplication here creates a shallow copy of the list.

```(python)
>>> [iter(k)] * 2
[<list_iterator object at 0x7fcf654c9f28>, <list_iterator object at 0x7fcf654c9f28>]
```

3. The star operator `*` then unpacks the collection as positional arguments to a function which is `zip` in this case. `zip` is a handy tool to merge several iterable together.
```(python)
>>> zip(*[iter(k)] * 2)
<zip object at 0x7fcf654de808>
```

4. Finally, the `list` operator just runs through to generate the entire list, giving us the desired output.
```(python)
>>> list(zip(*[iter(k)] * 2))
[(1, 2), (3, 4), (5, 6)]
```

What's strange about all this is that it depends on subtle behaviours of the underlying methods. For example, instead of `zip(*[iter(k)] * 2)` you wrote `list(zip(*[iter(k), iter(k)]))`. You will end up with a different result. The solution depends on the iterators being a shallow copy! Each time any of the iterator is hit, it calls the `next` call to the function. 

### Show, don't tell

I'd hate to encounter snippets like this in the wild as it places significant cognitive load on people trying to read this. Strange it was included in the official 2.x documentation, thankfully removed from the current versions.  
