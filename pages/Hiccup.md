tags:: programming

- Hiccup is a [[domain-specific language]] for generating HTML used mostly in Clojure community.
- Logseq uses Hiccup to generate HTML from [[clojurescript]] and [[datomic]], the languages Logseq and logseq queries are written in. For every day use you need very little knowledge of Hiccup, but some elements are bot h simple and useful.
- ## Let's Hiccup
	- The most obvious thing you can use Hiccup for is to style the titles of [[Advanced Queries]]:
	- ```clojure
	  #+BEGIN_QUERY
	  {:title [:h3 "This is hiccup!"]
	   :query [:find (pull ?b [*])
	   :where
	      [task ?b #{"LATER"}]
	   ]
	  }
	  #+END_QUERY
	  ```
		- Line 2 is written in Hiccup: `[:h3 "This is hiccup!"]`, this will be converted to: `<h3>This is hiccup!</h3>`
	- `h3`, as used in Hiccup, is the same has an `H3` in HTML, and you can also use other HTML tags, for example: `<p>` or `<b>`, which in Hiccup would be: `[:p "this is an HTML p tag"]` or `[:b "This is an HTML b (bold) tag"]`.
	- For simple titles, that's (almost) all you need. If you would like to add some css divs or classes to the mix, read on:
- ### Basic Hiccup Syntax
	- What is _actually_ happening when you write your title? Hiccup turns Clojure data structures like this:
		- ```clojure
		  [:h3 "This is hiccup!"]
		  ```
	- Into strings of HTML like this:
		- ```html
		  <h3>This is hiccup!</h3>
		  ```
	- Technically speaking, the Clojure data structure is a vector (`[ .... ]`, just like [[Advanced Queries]] uses for
	   searches) that takes one of the following forms:
	- ```clojure
	  [tag & body]
	  [tag attributes & body]
	  ```
	- The first item in the vector is the tag name. It is mandatory, and should be a keyword (`:h3`), a string (`"This is hiccup!"`) or a symbol (`?b` from queries).
	- The second item may optionally be a map of attributes.
	- All subsequent items in the vector are treated as the element body. This can include strings or nested tag vectors, for example:
	- ```clojure
	  [:p "Hello " [:em "World!"]]
	  ```
	- This definition might sound a bit obvious, but later you'll see how you can use the same syntax to create more complicated HTML structures, like lists (`<ul><li>...`) or tables (`<tr><th>...`).
- ### Making it look nice with CSS
	- There are two ways to add **ids** and **classes** to your html elements. The first is the most straight forward, and easy to read, but a bit long:
	- ```clojure
	  [:h3 {:id "mysearch" :class "underlined superpink"} "This is better looking Hiccup!"]
	  ```
	- Hiccup also provides a convenient shorter way of writing:
	- ```clojure
	  [:h3#mysearch.underlined.superpink "This is better looking Hiccup!"]
	  ```
	- As in CSS, the word after the "#" denotes the element's ID, and the word after each "." denotes the element's classes.
	- There may be multiple classes, but there can only be one ID. Additionally, the ID must always come first, so `div#foo.bar` would work, but `div.foo#bar` would not.
	- You can add an ID on its own, or a class on its own:
	- ```clojure
	  [:div#post "..."]
	  [:div.comment "..."]
	  ```
	- The most amazing thing is, you can actually do this _straight inside Logseq_:
		- ```clojure
		  [:h2 {:style {:color "red"}} "h2 title"]
		  [:p "Hello " [:em "World!"]]
		  ```
		- [:h2 {:style {:color "red"}} "h2 title"]
		  [:p "Hello " [:em "World!"]]
- ### Using Hiccup for :views
	- [[Advanced Queries]] support custom-build views for search results. These views are a combination of (a small sub-set of) Clojure and Hiccup. It's not the easiest combination, but without a doubt you can build amazing things with it.
- #+BEGIN_QUERY
  {:title "All pages have a *programming* tag"
   :query [:find ?name
         :in $ ?tag
         :where
         [page-tags ?p #{"page-tag-1"}]
         ;[?t :block/name ?tag]
         ;[?p :block/tags ?t]
         ;[?p :block/name ?name]
         ]
   :inputs ["programming"]
   :view (fn [result]
         [:div.flex.flex-col
          (for [page result]
            [:a {:href (str "#/page/" page)} (clojure.string/capitalize page)])])}
  #+END_QUERY
- ### Expanding seqs
- If you include a Clojure seq in the body of an element vector:
- ```clojure
  [:div (list "Hello" "World")]
  ```
- This is equivalent to:
- ```clojure
  [:div "Hello" "World"]
  - ```
  In other words, the seq is "expanded" out into the body. This is particularly useful for macros like `for`:
  - ```clojure
  [:ul (for [x coll] [:li x])]
  ```
- Note that while lists are considered to be seqs by Clojure, vectors and sets are not.
- ### Additioanl resources
	- [Hiccup Tips](https://ericnormand.me/mini-guide/hiccup-tips)
	- [Tutorial on Medium](https://medium.com/makimo-tech-blog/hiccup-lightning-tutorial-6494e477f3a5)