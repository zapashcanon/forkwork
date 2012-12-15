# [ForkWork](https://github.com/mlin/forkwork)

A simple OCaml library for forking child processes to perform work on multiple cores.

ForkWork is intended for workloads that a master process can partition into independent jobs, each of which will typically take a while to execute (several seconds, or more). Also, the resulting values should not be too massive, since they must be marshalled for transmission back to the master process.

Among the numerous tools for multicore parallelism available in the OCaml ecosystem, ForkWork fits somewhere in between [Netmcore](http://projects.camlcity.org/projects/dl/ocamlnet-3.6.1/doc/html-main/Netmcore.html) and [Parmap](http://www.dicosmo.org/code/parmap/). It's a bit easier to use than the former, and a bit more flexible than the latter.

## API documentation

See [ocamldoc:ForkWork](http://mlin.github.com/forkwork/ForkWork.html)

## Installation

ForkWork is mainly tested on Linux and Mac OS X with OCaml 3.12 and above. It should work on most flavors of Unix. The easiest way to install it is by using [OPAM](http://opam.ocamlpro.com):
`opam install forkwork`.

If you don't use OPAM, set up [findlib](http://projects.camlcity.org/projects/findlib.html),
define the `OCAMLFIND_DESTDIR` environment variable if necessary and

```git clone https://github.com/mlin/forkwork.git && cd forkwork && make && make install```

You can then use `ocamlfind` as usual to include the `forkwork` package when
compiling your program (making sure `OCAMLPATH` includes `OCAMLFIND_DESTDIR` if
you changed it).

## High-level example

ForkWork provides a high-level interface consisting of parallel map functions for lists and arrays. Here's a program that forks four worker processes to compute a Monte Carlo estimate of π:

```
let worker k =
  Random.self_init ();
  let inside = ref 0 in
  let outside = ref 0 in begin
    for i = 1 to k do
      let x = Random.float 1.0 in
      let y = Random.float 1.0 in
      incr (if x *. x +. y *. y < 1.0 then inside else outside)
    done;
    (!inside,!outside)
  end
;;

let estimate_pi n k =
  let results = ForkWork.map_array worker (Array.make n k) in
  let insides, outsides = List.split (Array.to_list results) in
  let inside = float (List.fold_left (+) 0 insides) in
  let outside = float (List.fold_left (+) 0 outsides) in
  4.0 *. (inside /. (inside +. outside))
;;

Printf.printf "%f\n" (estimate_pi 4 25_000_000)
;;
```

ForkWork tries pretty hard to deal with exceptions in the forked child processes in a reasonable way, which is difficult because they [cannot be marshalled reliably](http://caml.inria.fr/mantis/view.php?id=1961). If an exception occurs during a parallel map operation, ForkWork will stop launching new child processes, wait for others to exit, and raise an exception to the caller.

## Lower-level example

There is also a lower-level interface providing much more control over the scheduling of child processes and retrieval of their results. 

TODO: example

## Running tests

The tests use [kaputt](http://kaputt.x9c.fr). To run them, `./configure --enable-tests` and `make test`.
