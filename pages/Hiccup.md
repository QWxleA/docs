tags:: programming

- Hiccup is a [[domain-specific language]] for generating HTML, used mostly in Clojure community.
- For every day use you need very little knowledge of Hiccup, and fortunately (most of it) is surprisingly simple.
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
- ## Basic Hiccup Syntax
	- What is _actually_ happening when you write your title? Hiccup turns Clojure data structures like this:
		- ```clojure
		  [:h3 "This is hiccup!"]
		  ```
	- Into strings of HTML like this:
		- ```html
		  <h3>This is hiccup!</h3>
		  ```
	- Technically speaking, the Clojure data structure is a vector (`[ .... ]`, just like [[Advanced Queries]] uses for searches, that takes one of the following two forms:
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
	- This definition might sound a bit obvious, or unimportant, but later you'll see how you can use the same syntax to create more complicated HTML structures, like lists (`<ul><li>...`) or tables (`<tr><th>...`).
- ## Making it look nice with CSS
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
- ## Using Hiccup for :views
	- [[Advanced Queries]] support custom-build views for search results. These views are a combination of (a small sub-set of) Clojure and Hiccup. It's not the easiest combination, but without a doubt you can build amazing things with it.
	- ``` clojure
	  #+BEGIN_QUERY
	  {:title [:b "All pages with a " [:em "programming"] " tag"]
	   :query [:find ?name
	   :in $ ?tag
	   :where
	    [?t :block/name ?tag]
	    [?p :block/tags ?t]
	    [?p :block/name ?name]]
	   :inputs ["programming"]
	   :view (fn [result]
	  	         [:div.flex.flex-col
	  	          (for [page result]
	  	            [:a {:href (str "#/page/" page)} (clojure.string/capitalize page)]
	                  )
	                ]
	           )}
	  #+END_QUERY
	  ```
	- Let's examine one of the [[Advanced Queries]], we are only interested in lines **11** to **15**. It is an excellent example how search results, clojure and hiccup can represent search results:
	- These lines will create the following:
	- ```html
	  <div class="flex flex-col">
	  <a href="#/page/advanced queries">Advanced queries</a>
	  <a href="#/page/hiccup">Hiccup</a>
	  </div>
	  ```
	- **Line 11 (query):** creates the div with the two classes flex and flex-col, that div closes in **line 15** (**4** in the html)
	- **Line12:** is interesting _clojure_, it loops over _result_, and every single value is stored in _page_
	- **Line 13:** is the most complex Hiccup line so far:
		- `[:a {:href (str "#/page/" page)} (clojure.string/capitalize page)]`
	- which translates to:
		- `<a href="#/page/hiccup">Hiccup</a>`
	- And if you look carefully, they look very much alike:
		- The whole thing is wrapped in an `<a>` -> `[:a .... ]`
		- Then you have the address `href="#/page/hiccup"` -> `(str "#/page/" page)`, which is Clojure for: **str** returns a string, in our case it concatenates (glues together)  `"#/page/"` and the value of `page`.
		- Last you have the visible part of the link `Hiccup` -> `(clojure.string/capitalize page)`, which is not hard to guess, it capitalizes the value of `page`.
		- **Note:** if you ever use this code, and `page` is a journal-page (which is a bunch of numbers), the whole thing will fail, so don't capitalize numbers, they don't like that.
	- And this is what it looks like:
		- query-table:: false
		  #+BEGIN_QUERY
		  {:title [:b "All pages with a " [:em "programming"] " tag"]
		   :query [:find ?name
		   :in $ ?tag
		   :where
		    [?t :block/name ?tag]
		    [?p :block/tags ?t]
		    [?p :block/name ?name]]
		   :inputs ["programming"]
		   :view (fn [result]
		  	         [:div.flex.flex-col
		  	          (for [page result]
		  	            [:a {:href (str "#/page/" page)} (clojure.string/capitalize page)]
		                  )
		                ]
		           )}
		  #+END_QUERY
- ### Table views
	- A regular table is a great way to show data, this is how it's done in hiccup:
	- **Query to create a table with page and todo count**
	  link:: [Discord](https://discord.com/channels/725182569297215569/743139225746145311/921337299164356658)
	  date:: [[2021-12-17]]
		- query-table:: false
		  ```clojure
		  #+BEGIN_QUERY 
		  {:title "TODO by page"
		    :query     [:find (pull ?b [:block/marker :block/parent {:block/page
		       [:db/id :block/name]}])
		    :where
		             [?b :block/marker ?marker]
		             [(= "TODO" ?marker)] 
		    ]
		  :result-transform (fn [result]
		                          (map (fn [[key value]] {:page (get key :block/name) :count (count value)}) (group-by :block/page result))
		                  )
		  :view (fn [rows] [:table 
		   [:thead 
		    [:tr 
		     [:th "Page"] 
		     [:th "Count"] ] ] 
		   [:tbody 
		  (for [r rows] [:tr 
		     [:td [:a {:href (str "#/page/" (get r :page))} (get r :page)] ] 
		     [:td (get r :count)] ])
		     ]]
		  )
		  }
		  #+END_QUERY
		  ```
		- #+BEGIN_QUERY 
		  {:title "TODO by page"
		    :query     [:find (pull ?b [:block/marker :block/parent {:block/page
		       [:db/id :block/name]}])
		    :where
		             [?b :block/marker ?marker]
		             [(= "TODO" ?marker)] 
		    ]
		  :result-transform (fn [result]
		                          (map (fn [[key value]] {:page (get key :block/name) :count (count value)}) (group-by :block/page result))
		                  )
		  :view (fn [rows] [:table 
		   [:thead 
		    [:tr 
		     [:th "Page"] 
		     [:th "Count"] ] ] 
		   [:tbody 
		  (for [r rows] [:tr 
		     [:td [:a {:href (str "#/page/" (get r :page))} (get r :page)] ] 
		     [:td (get r :count)] ])
		     ]]
		  )
		  }
		  #+END_QUERY
		- #### This is the data used in the previous query:
			- TODO first todo
			- TODO second todo
	- ### A more complete table
- query-table:: false
  #+BEGIN_QUERY
  {
   :query [:find (pull ?b [{:block/page
       [:block/name :block/journal-day]} :block/properties])
        :where
        [property ?b :type "programming_lang"]
        ;[(get ?bprops :distanse "nil") ?bs]
        ;[(not= ?bs "nil")]
         ]
  :result-transform (fn [result]
                       (sort-by (fn [s]
                          (get-in s [:block/page :block/journal-day])) (fn [a b] (compare b a)) result)) 
  :view (fn [rows] [:table 
   [:thead 
    [:tr 
     [:th "Dato"] 
     [:th "Distanse"]
     [:th "Økt"] ] ] 
   [:tbody 
  (for [r rows] [:tr 
     [:td (get-in r [:block/page :block/name])] 
     [:td (get-in r [:block/properties :distanse])]
     [:td (get-in r [:block/properties :økt])] ])
     ]]
  )
  }
  #+END_QUERY
- Pascal
  type:: programming_lang
  creator:: me
  description:: none
-
- ### Additional resources
	- [Hiccup Tips](https://ericnormand.me/mini-guide/hiccup-tips)
	- [Tutorial on Medium](https://medium.com/makimo-tech-blog/hiccup-lightning-tutorial-6494e477f3a5)