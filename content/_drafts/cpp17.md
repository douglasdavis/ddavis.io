---
layout: post
title:  "C++17 Early Favorites"
disqustitle: "cpp17fav"
comments: false
---

------

C++ holds a special place in my heart and mind because it was my first
programming language and I still use it heavily today. If you've read
any of my other blog posts you've seen that the main workhorse of high
energy physics is C++. I was lucky enough to _really_ start using the
language when the 2011 standard was in the growing pain stage. I
learned a lot following that process and I continue to keep up with
the developments of the language and the standard library.

C++17 was [formally
approved](https://herbsutter.com/2017/09/06/c17-is-formally-approved/)
a few months ago. These days the GCC and Clang developers do an
awesome job implementing features early on, so a lot of C++17 language
and standard library features have been available for a number of
recent releases of both compilers (and their respective STL
implementations). Unfortunately it'll be a little while before I can
use C++17 in ATLAS production code. Nevertheless, I've still
identified a couple of early favorite features.

## Structured Bindings

When writing Python I always appreciate the simplicity of multiple
function returns and the intuitive loop over a dictionary. Adding
structured bindings to C++ makes it easy to use multiple returns and
intuitive to loop over an
[`std::map`](http://en.cppreference.com/w/cpp/container/map).

For multiple returns in C++ 11 and 14 we had to use
[`std::tie`](http://en.cppreference.com/w/cpp/utility/tuple/tie):

```cpp
auto foo() {
  return std::make_tuple(1,2.0,'3');
}

int main() {
  int i;
  float j;
  char k;
  std::tie(i,j,k) = foo();
  // ...
}
```

Now, with structured bindings, it's as easy as:

```cpp
int main() {
  auto [i, j, k] = foo();
  // ...
}
```

Before C++17, when looping over the `std::map` container, we were
locked into using the `first` and `second` members of
[`std::pair`](http://en.cppreference.com/w/cpp/utility/pair):

```cpp
{% raw %}int main() {
  std::map<int,float> myMap {{1,1.1},{2,2.2}};
  for ( const auto& entry : myMap ) {
    doSomething(entry.first,entry.second);
  }
  // ...
}{% endraw %}
```

With structured bindings, we have something a bit more intuitive with
less boilerplate:

```cpp
{% raw %}int main() {
  std::map<int,float> myMap {{1,1.1},{2,2.2}};
  for ( const auto& [i, j] : myMap ) {
    doSomething(i,j);
  }
  // ...
}{% endraw %}
```

For even more boilerplate, go back to C++03 container looping with
iterators :)

## The Filesystem Library

more later.
