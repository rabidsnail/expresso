expresso
========

A clojure library for symbolic manipulation of Algebraic Expressions. 


[![Build Status](https://travis-ci.org/clojure-numerics/expresso.png?branch=master)](https://travis-ci.org/clojure-numerics/expresso)

```clojure
(solve 'blue
  (ex (= pencils (+ green white blue red)))
  (ex (= (/ pencils 10) green))
  (ex (= (/ pencils 2) white))
  (ex (= (/ pencils 4) blue))
  (ex (= red 45))) ;=> #{{blue 75N}}
```
### Objectives

expresso aims to be a general library for manipulating mathematical expressions.
This are the key objectives:

 - Enable mathematical expressions to be encoded
 - Provide a powerful facility for general expression manipulation (aka term rewriting)
 - Provida a range of very useful manipulations, which include
   - simplifying mathematical expressions
   - solving a (set of) equations in regard to unknowns
   - differentiating expressions
   - optimizing of an expression for performance
   - compiling an expression to a clojure function
 - Be extensible through adding domain knowledge
 - Full compatibility with [core.matrix](https://github.com/mikera/matrix-api)
 
 
### Getting started

Add the following line to your leiningen dependencies:
```clojure
[expresso "0.1.0"]
```

### Defining expressions

expressos expressions are just normal clojure s-expressions. Expresso has various convenience functions/macros
for creating expressions:

```clojure
;;the ex macro constructs an expression with automatic quoting of variables
(let [x 3]
  (ex (+ x ~x))) ;=> (+ x 3)

;;the ex' macro constructs an expression with explicit quoting of variables
(let [x 3]
  (ex' (+ x 'x))) ;=> (+ 3 x)

;;all functions in the core namespace also support pure s-exp as parameters
(solve '[x] '(= (+ 1 x) 3)) ;=> #{2}
```
### manipulations of algebraic expressions

```clojure
(use 'numeric.expresso.core)

(simplify (ex (+ (* 4 a) (* 3 a) (* -1 (* 7 a))))) 
=> 0

(def F1 (ex (= Y (+ X Z)))
(def F2 (ex (= X [1 2 3]))
(def F3 (ex (= Z (* 2.0 X)))

(solve [Y] F1 F2 F3)
=> #{[3.0 6.0 9.0]}        

(def opt (optimize (ex (+ b (* (+ 5 b) (** y (+ a b)) (** z (+ b a)))))))
opt
=> (let [local478813 (+ a b)] (+ (* b 6) (** y local478813) (** z local478813)))

(def f (compile-expr [a b y z] opt))

(f 1 2 3 4)
=> 103.0
```
The public API is in numeric.expresso.core - go test it out!

### General Manipulation of Expressions
Expresso supports a powerful way to manipulate expressions with rewrite rules, which are built ontop of 
core.logic. Here are a few example rules

```clojure
(use 'numeric.expresso.rules)

;;?&* matches zero or more items
(def r [(rule (ex (* 0 ?&*)) :=> 0)
        (rule (ex (* 1 ?&*)) :=> (ex (* ?&*)))
        ;;supports optional guard with :if
        (rule (ex (/ ?x ?x)) :=> 1 :if (guard (not= ?x 0)))])

;;rules match semantically. because * is commutative the rules match regardless of the order of arguments
(transform-expression r (ex (+ (* 2 (/ 4 4) 3) a (* 4 0))))
=> (+ (* 2 3) a 0)

;;The right hand side and guard of a rule can be 'arbitrary core.logic relations'.
;;The trans and guard macro create suitable core.logic relations out of normal clojure code.
;;==> is a shorthand for trans

(apply-rule (rule (ex (map ?f ?coll)) :==> (map ?f ?coll) :if (guard (coll? ?coll)))
  	    (ex (map ~inc [1 2 3]))) ;=> (2 3 4)
```

### Status 
This library is in a 'very' early state. Don't expect it to be stable yet. Any bug-reports/feature recommendations are very welcome
