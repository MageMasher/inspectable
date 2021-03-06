# Inspectable

In your spec table.

A bunch of tools to help improve your repl experience when using clojure.spec with clojure and clojurescript.

## Installation

For clojure make sure you are using `[org.clojure/clojure "1.9.0-alpha17"]` or newer.

For clojurescript make sure you are using `[org.clojure/clojurescript "1.9.854"]` or newer.

To include the library add the following to your `:dependencies`.

    [inspectable "0.2.2"]
    
## Usage

For clojure :

```clojure
user> (require '[inspectable.repl :refer [why browse-spec]])
```

For clojurescript :

```clojure
;; In the clojure repl 
user> (use 'inspectable.repl)
user> (start-cljs)


;; In your clojurescript repl
cljs.user> (require '[inspectable.cljs-repl :refer-macros [why] :refer [browse-spec]])
```

Currently inspectable provides two tools: `browse-spec` and `why`. See below to understand how they can help you.


## The spec browser (browse-spec)

The spec browser lets you explore your spec registry through a java swing graphical interface.
You can invoke it with different type of arguments:

```clojure
(browse-spec) ;; open the browser showing all specs in the registry
(browse-spec "ring") ;; open the browser showing specs matching "ring" regex
(browse-spec :ring/request) ;; open the browser pointing to :ring/request spec
(browse-spec 'clojure.core/let) ;; open the browser pointing to 'clojure.core/let spec
```

For example, suppose you are working with [ring](https://github.com/ring-clojure/ring) and have [ring-spec](https://github.com/ring-clojure/ring-spec) loaded,
then you can use ```(browse-spec "ring")``` to see a list of all specs in the registry that matches "ring" in their names. 

<img src="/doc/images/browser-all-ring.png?raw=true"/>

Once in the browser, you can click on the specs to navigate down to sub specs, or use the top bar 
to navigate back to previous visited specs:

<img src="/doc/images/browser-ring-request.png?raw=true"/>

As you can see, browsing a spec shows you a pretty print of the spec form together with a generated spec sample.

<img src="/doc/images/browser-ring-server-port.png?raw=true"/>

Inspectable spec browser also supports browsing multi-specs: 

<img src="/doc/images/browser-multi.png?raw=true"/>

In case a sample can't be generated the exception will be displayed intead of the example

<img src="/doc/images/browser-sample-not.png?raw=true"/>

## Specs fail explain (why)

Another useful tool provided by inspectable is `why`.
You can use `why` to understand why clojure.spec is complaining about a spec in three different situations.

First, you can use `why` to help you understand the output of `s/explain-data`

```clojure
(def bad-request (ring.mock.request/request :get "htp://localhost:69000/test"))

(s/valid? :ring/request bad-request) ;; => false

(why (s/explain-data :ring/request bad-request))
```

<img src="/doc/images/ring-req-fail-pp.png?raw=true"/>

In this case the pprint is enough, but if the data structure is a big and deeply nested one, try
using the collapsible tree:

<img src="/doc/images/ring-req-fail-tree.png?raw=true"/>

Second, `why` can help in situations like calling an instrumented function that fails.

See [Integrating inspectable with your repl](#integrating-inspectable-with-your-repl) for a way of automatically
calling why on this situations.

Suppose we are working with events as defined in [clojure spec guide](https://clojure.org/guides/spec#_multi_spec)
and some function :

```clojure

(s/fdef only-with-code
        :args (s/cat :code :error/code
                     :events (s/coll-of :event/event))
        :ret :event/event)

(defn only-with-code [code events]
  ...)

;; instrument it
(stest/instrument)

;; try to call it with some args
(why
 (only-with-code 4
                  [{:event/type :event/search
                    :event/timestamp 0
                    :search/url "/home"}
                   {:event/type :event/search
                    :event/timestamp 5.5
                    :search/url "/home"}
                   {:event/type :event/error
                    :error/code 4
                    :search/url "/home"}
                   {:event/type :event/search
                    :search/url "/about"}]))

```

and you will get :

<img src="/doc/images/fn-instrument-fail.png?raw=true"/>

Third, `why` can also be used to catch macro expansion exceptions like in the case of:

```clojure
(why (let [a 1
           b 2
           3]
       (+ a b c)))
```

will show you :

<img src="/doc/images/let-fail.png?raw=true"/>

See [Integrating inspectable with your repl](#integrating-inspectable-with-your-repl) for a way of automatically
calling why on this situations.



## Integrating inspectable with your repl

You can call (inspectable.repl/install) to install inspectable on your current repl so every time spec
throws a exceptions `why` will be automatically applied over it.

```clojure
(inspectable.repl/install)
```

There is a function `inspectable.repl/repl-caught` you can use for the same purpose if you are starting
your own sub repl.

Starting a new repl :

```clojure
(clojure.main/repl :caught inspectable.repl/repl-caught)
```

## Workflows integration ideas

### Re-frame

If you are using re-frame and you have specs for your db you can modify your spec check middleware like :

```clojure

(defn check-and-throw
  "throw an exception if db doesn't match the spec"
  [a-spec db]
  (when-not (s/valid? a-spec db)
    (why (s/explain-data a-spec db)))) ;; <---- use why here

(def check-spec (after (partial check-and-throw :my-app.db/db)))

```

### Cider

If you are using Cider and Clojurescript 

```elisp
(setq cider-cljs-lein-repl "(do (use 'inspectable.repl) (start-cljs))")
```

so after your `cider-jack-in-clojurescript` everything is ready.

## Related work

If you are interested in understanding specs when they fail also checkout [expound](https://github.com/bhb/expound)!

## Roadmap

- Multiple themes
- Reference to the caller for instrumented functions fails.
- Test and add instructions for React Native
