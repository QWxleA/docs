- Hiccup is a domain-specific language for generating HTML used mostly in Clojure community.
- Check out [this](https://medium.com/makimo-tech-blog/hiccup-lightning-tutorial-6494e477f3a5) tutorial for a quick introduction.
- You can embed Hiccup inside block same as if you want to embed HTML.
- Example:
	- [:h2 {:style {:color "red"}} "h2 title"]
	    [:p "Hello " [:em "World!"]]
- Logseq uses Hiccup to generate HTML from clojurescript and datomic, the languages Logseq and logseq queries are written in. For every day use you need very little knowledge of Hiccup, but some elements are bot h simple and useful.
- ## Absolute simplest use of Hiccup
	- The very first thing you can use Hiccup for is to style **query-titles**
	- ```clojure
	  ```
### Basic Syntax

Hiccup turns Clojure data structures like this:

```clojure
[:a {:href "http://github.com"} "GitHub"]
```

Into strings of HTML like this:

```html
<a href="http://github.com">GitHub</a>
```

Using the [hiccup.core/html][1] macro.

The Clojure data structure is a vector that takes one of the following forms:

```clojure
[tag & body]
[tag attributes & body]
```

The first item in the vector is the tag name. It is mandatory, and should be a keyword, string or symbol.

The second item may optionally be a map of attributes.

All subsequent items in the vector are treated as the element body. This can include strings or nested tag vectors, for example:

```clojure
[:p "Hello " [:em "World!"]]
```

[1]: http://weavejester.github.com/hiccup/hiccup.core.html#var-html
### CSS-style sugar

Hiccup provides a convenient shortcut for adding `id` and `class` attributes to an element. Instead of writing:

```clojure
[:div {:id "email" :class "selected starred"} "..."]
```

You can write:

```clojure
[:div#email.selected.starred "..."]
```

As in CSS, the word after the "#" denotes the element's ID, and the word after each "." denotes the element's classes.

There may be multiple classes, but there can only be one ID. Additionally, the ID must always come first, so `div#foo.bar` would work, but `div.foo#bar` would not.

You can add an ID on its own, or a class on its own:

```clojure
[:div#post "..."]
[:div.comment "..."]
```
### Expanding seqs

If you include a Clojure seq in the body of an element vector:

```clojure
[:div (list "Hello" "World")]
```

This is equivalent to:

```clojure
[:div "Hello" "World"]
```

In other words, the seq is "expanded" out into the body. This is particularly useful for macros like `for`:

```clojure
[:ul (for [x coll] [:li x])]
```

Note that while lists are considered to be seqs by Clojure, vectors and sets are not.