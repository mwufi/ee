---
title: "Learning Ocaml"
date: 2018-12-22T23:13:36-05:00
tags: ["Ocaml"]
---

> Programming languages matter. 

This is the beginning of the OCaml book. It's been a while since I've learned a new language to code in, and now, perhaps, is as good a time as any.

## the plan

> Maybe if I spend an hour a day learning (reading and making programs) I'll be able to "think" in OCaml by the end of the break! Wouldn't that be great? I think OCaml is a great language to write compilers in, like Haskell. It also looks really neat - in fact, this is the reason I'm bothering to do this in the first place!

# Introduction

The installation and management of these third-party libraries is made much easier via a package management tool known as OPAM. We'll explain more about OPAM as the book unfolds, but it forms the basis of **the Platform**, which is a set of tools and libraries that, along with the OCaml compiler, lets you build real-world applications quickly and effectively.

# Install OCaml

The first step is to install! I followed the guide [here](https://github.com/janestreet/install-ocaml), which told me that I could do this:
```
brew install -y opam m4
```

# Our first program

```ocaml
open Base;;

8 / 3;;
- : int = 2
3.5 +. 6.;;
- : float = 9.5
let x = 3 + 4;;
val x : int = 7
let x' = 3. + 4.;;
val x' : float = 7.
```

* I like how the metaphor is `open`, and it ends with two semi-colons. 
* OCaml distinguishes between floats and ints! You need different operators: `+.` instead of `+`
* variaaables! Punctuation is excluded, except for _ and ', and variables must start with a lowercase letter or an underscore. `FuNNyBusiness` won't cut it!

We can also create functions.
```ocaml
let square x = x * x;;
square 2;;
- : int = 4
```

and import *modules* (in this case, `of_int` is a method which lives in the `Float` module)
```ocaml
let ratio x y =
    Float.of_int x /. Float.of_int y
;;
val ratio : int -> int -> float =

ratio 4 7;;
- : float = 0.57142857
```

Modules can be opened to make their contents available without explicitly typing the module name. This is what we did with `Base` earlier. To make it easier to read:
```ocaml
let ratio x y =
    let open Float.O in
    of_int x / of_int y
```
or even better:
```ocaml
let ratio x y = 
    Float.O.(of_int x / of_int y)
```

What does this function do?
```ocaml
let sum_if_true test first second =
    (if test first then first else 0)
    + (if test second then second else 0)
;;
val sum_if_true : (int -> bool) -> int -> int -> int =
```

* What's with the arrows? You can read them as "everything but the last one is an argument, and the last one is the return value"

Equal signs are weird here:
```ocaml
let even x =
x % 2 = 0
;;
val even : int -> bool =
```

* We used `=` in two different ways: once as part of the `even` definition; and once as an equality test, when comparing `x % 2` to `0`.

# Type Inference

OCaml does some clever type inference when you write your programs. Imagine the following function:
```ocaml
let first test x y =
    if test x then x else y
;;
val first_if_true : ('a -> bool) -> 'a -> 'a -> 'a =
```

How does it know what to return? `x` and `y` can be anything! But OCaml enforces a single type, since a function can't return two different types (unlike in Python). So there is one constraint now, that `x` and `y` have to share a common type.

# Lists of data

Sooner or later, you'll want to deal with more interesting things than numbers, and that's where collections come in.

Tuples
```ocaml
let a_tuple = (3, "three");;
val a_tuple : int * string = (3, "three")
let (x,y) = a_tuple;;
val x : int = 3
val y : string = "three"

x + String.length y
- : int = 8

let distance (x1, y1) (x2, y2) = 
    Float.sqrt ((x1 -. x2)) **. 2 +. ((y1 -. y2)) **. 2
;;
val distance : float * float -> float * float -> float =
```  

Lists
```ocaml
let languages = ["Spanish"; "Esperanto"];;
val languages : string list = ["Spanish"; "Esperanto"]

List.length languages :: "No";;
- : int = 3

List.map languages ~f:String.length;;
- : int list = [7; 9; 2]

[1;2;3] @ [4;5;6];;
- : int list = [1; 2; 3; 4; 5; 6]
```

* We can add elements to the front of a list with `::`:
* It's important to remember that `@` is not a constant time operation. (*wait, it's not? Are you sure they're not using linked lists?*)

Special note

* `~f` is a *labeled argument*! (for `List.map` above). Labeled arguments are specified by name rather than by position (and hence, can be specified not in any order)

Like in Lisp, you can do things like "extract the first element from a list":
```ocaml
let my_fav (fav :: rest) =
    fav
;;
val my_favorite_language : 'a list -> 'a =

my_favorite_language ["English";"Spanish";"French"];;
- : string = "English"
my_favorite_language [];;
Exception: Match_failure ("//toplevel//", 1, 26).
```

But uh-oh. What happened? (*what if the list has no elements?!*) New version uses `match` to handle all the cases.
```ocaml
let my_fav languages =
    match languages with
    | first :: rest -> first
    | [] -> "Ocaml is the best!!!" (*A good default!*)
;;
```

* I think this is the first function that I'm happy with (is distinctively OCaml)

How to make a function recursive (just add `rec`!)
```ocaml
let rec sum everything =
    match everything with
    | [] -> 0
    | head :: tail -> head + sum tail
;;
sum [1;2;3];;
- : int 6
```

Options
```ocaml
let divide x y =
if y = 0 then None else Some (x / y)
;;
val divide : int -> int -> int option =
```

* This function returns an Option, not a real value. what can we do with options? A: can use pattern matching, as we did with tuples and lists

```ocaml
let f x y =
    match divide x y  with
    | None -> 100 (*divide by 0 error!*)
    | Some (i) -> i
;;
```

Why do options help? In languages like Java and C#, variables might be nullable. But here, if we want to make that the case, we have to explicitly change the type to an option. 

Defining new types
```ocaml
type point2d  = {x: float, y: float};;

let magnitude { x = x_pos; y = y_pos } =
Float.sqrt (x_pos **. 2. +. y_pos **. 2.)
;;
val magnitude : point2d -> float =
```

Finally, have a code snippet. Recognize all the things that are being used here?
```
let is_inside_scene_element point scene_element =
  let open Float.O in
  match scene_element with
  | Circle { center; radius } ->
    distance center point < radius
  | Rect { lower_left; width; height } ->
    point.x    > lower_left.x && point.x < lower_left.x + width
    && point.y > lower_left.y && point.y < lower_left.y + height
  | Segment { endpoint1; endpoint2 } -> false
```

And now it's time for bed!!
