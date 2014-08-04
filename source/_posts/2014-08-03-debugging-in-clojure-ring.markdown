---
layout: post
title: "Ring Basics"
date: 2014-08-03 22:32
comments: true
published: false
categories:
- programming
- debugging
- tools
- clojure
- ring
---

Ring is the de facto core of Clojure web services.


* Ring Concepts
  * Request map
  * Response map
  * Handler function
  * Middleware

* Patterns
  * Middleware
  * Specifying vars instead of handlers

* Testing
  * Unit testing handlers
  * Unit testing middleware
  * Integration testing

* Debugging
  * Logging request and response


## Debugging Ring Requests$

{% gist iantruslove/203071cf3385f70c818a %}


I have seen a few questions about debugging

Specifically about your Ring debugging, perhaps creating Ring requests
isn't as hard as you think.  If you're testing a handler, you need to
create a Ring request with as little data as you can get away with,
and inspect the resulting response. The Ring "Concepts" wiki page is
helpful in determining what exactly goes into a request map. I find
that you generally need a :uri and a :request-method to test a
(normally Compojure-generated) handler, and perhaps a :headers map
too.

;; Source:
(defn my-handler [req]
    ;; ... your code here ...
  )

;; Test:
(let [req {:http-method :get
                      :uri "/api/comments"
                      :headers {"accept" "application/json"}}
            resp (my-handler req)]
    (is (= 200 (:status resp)))
      ;; etc
    )


Testing middleware is a little trickier. Middleware generally falls into one of a couple of categories - request modification, response modification, or side-effects (e.g. logging). Middleware implementations generally look like this:


(defn wrap-my-middlware [handler]
    (fn [req]
          ;; Early side-effects:
          (log/info "Wrapping incoming request...")

              ;; modify request here
              (let [resp (handler req)]
                      ;; Late side-effects:
                      (log/info "My middleware is almost done.")

                            ;; modify the response here
                            (assoc resp :some-key "some value"))))

Middleware functions always take a handler function to wrap. To test your middleware, it helps to carefully control and limit what the handler does. Here are a couple of useful ones - one that always just returns a known response, and another that just returns the request passed to it:

;; Some test-time handlers
(def identity-handler (constantly {:status 200 :body "ok"}))
(defn echo-handler "Echo back the request" [req] req)
Since middleware functions always return a function, you can only tell what your middlware did when you call the resulting wrapped handler.
;; Test the middlware:
(let [wrapped-handler (wrap-my-middleware identity-handler)
            req {:http-method :get
                            :uri "/api/comments"
                            :headers {"accept" "application/json"}}
            resp (wrapped-handler req)]
    (is (= 200 (:status resp)))
      ;; check for other middleware effects here
    )

