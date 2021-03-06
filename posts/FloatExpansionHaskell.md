---
author: Stéphane Laurent
date: '2017-06-03'
highlighter: 'pandoc-solarized'
output:
  html_document:
    highlight: zenburn
    keep_md: True
  md_document:
    toc: True
    variant: markdown
prettify: True
prettifycss: minimal
tags: 'haskell, R'
title: 'Calling a Haskell function in R - a float expansion example'
---

-   [Binary (and more) expansion in
    Haskell](#binary-and-more-expansion-in-haskell)
-   [First dynamic linker: string
    output](#first-dynamic-linker-string-output)
    -   [Make the function compatible with
        R](#make-the-function-compatible-with-r)
    -   [Compilation](#compilation)
    -   [Call in R](#call-in-r)
-   [Second dynamic linker: vector
    output](#second-dynamic-linker-vector-output)

In [the previous
article](https://laustep.github.io/stlahblog/posts/DyadicExpansion.html),
I wrote a R function returning the binary expansion of a real number in
$[0,1]$. In the present article, I will:

-   write a similar function in Haskell;
-   write this function in a way compatible with R, inside a module;
-   compile this module in a dynamic linker suitable for R (`dll` for
    Windows, `so` for Linux);
-   call the function from R through the dynamic linker.

The creation of a Haskell function compatible with R is allowed by the
[Foreign Function Interface
(FFI)](https://wiki.haskell.org/Foreign_Function_Interface), in other
words the `Foreign` module.

I learnt how to do such things with the help of [this blog post by Neil
Mitchell](http://neilmitchell.blogspot.be/2011/10/calling-haskell-from-r.html).

Binary (and more) expansion in Haskell
--------------------------------------

Let's go to Haskell. The `floatExpansion` function below is obtained by
a small modification of the `floatToDigits` function of the `Numeric`
module. It returns the expansion of a real number $u \in [0,1]$ in a
given integer base.

``` {.haskell}
import Numeric (floatToDigits)
:{
let floatExpansion :: RealFloat a => Integer -> a -> [Int];
    floatExpansion base u = replicate (- snd expansion) 0 ++ fst expansion
      where expansion = floatToDigits base u
:}
floatExpansion 2 0.125
## [0,0,1]
```

First dynamic linker: string output
-----------------------------------

Firstly, I show how to make this function compatible with R when its
output is a string instead of a list. It is easy to convert a list to a
string in Haskell:

``` {.haskell}
show [0, 0, 1]
## "[0,0,1]"
```

To get the output as a vector in R, more work is needed, and I will do
it in the next section.

### Make the function compatible with R

To make the function compatible with R, there are two rules:

-   Every argument must be a pointer (`Ptr`) to a C compatible type:
    `CInt`, `CDouble` or `CString`.

-   The result must be `IO ()`.

A value of type `Ptr` represents a pointer to an object. This type is
provided by the [`Foreign.Ptr`
module](https://hackage.haskell.org/package/base-4.9.0.0/docs/Foreign-Ptr.html),
which is imported via the `Foreign` module. The types `CInt`, `CDouble`
and `CString` are provided by the [`Foreign.C`
module](https://hackage.haskell.org/package/base-4.9.0.0/docs/Foreign-C.html).

We end up with this module:

``` {.haskell}
-- FloatExpansion1.hs
{-# LANGUAGE ForeignFunctionInterface #-}

module FloatExpansion where

import Foreign
import Foreign.C
import Numeric (floatToDigits)

foreign export ccall floatExpansion :: Ptr CInt -> Ptr CDouble -> Ptr CString -> IO ()

floatExpansion :: Ptr CInt -> Ptr CDouble -> Ptr CString -> IO ()
floatExpansion base u result = do
  base <- peek base
  u <- peek u
  expansion <- newCString $ show $ floatExpansion' (toInteger base) u
  poke result $ expansion

floatExpansion' :: RealFloat a => Integer -> a -> [Int]
floatExpansion' base u = replicate (- snd expansion) 0 ++ fst expansion
  where expansion = floatToDigits base u
```

### Compilation

We need the following C file to do the compilation, as explained in the
[GHC users
guide](https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/win32-dlls.html#making-dlls-to-be-called-from-other-languages).

``` {.c}
// StartEnd.c
#include <Rts.h>

void HsStart()
{
int argc = 1;
char* argv[] = {"ghcDll", NULL}; // argv must end with NULL

// Initialize Haskell runtime
char** args = argv;
hs_init(&argc, &args);
}

void HsEnd()
{
hs_exit();
}
```

Then we compile the library with this command on Linux:

``` {.bash}
ghc -shared -fPIC -dynamic -lHSrts-ghc8.0.2 FloatExpansion1.hs StartEnd.c -o FloatExpansion1.so
```

and this command on Windows:

``` {.bash}
ghc -shared -fPIC FloatExpansion1.hs StartEnd.c -o FloatExpansion1.dll
```

This creates the dynamic linker `FloatExpansion1.so` on Linux,
`FloatExpansion1.dll` on Windows.

In a cabal file, assuming `StartEnd.c` is in the project directory, we
can do:

``` {.cabal}
library
  hs-source-dirs:      src
  exposed-modules:     FloatExpansion
  build-depends:       base >= 4.7 && < 5
  default-language:    Haskell2010
  if os(windows)
    ghc-options:       -O2 -shared -fPIC StartEnd.c -o FloatExpansion1.dll
  else
    ghc-options:       -O2 -shared -fPIC -dynamic StartEnd.c -o FloatExpansion1.so
    extra-libraries:   HSrts-ghc8.0.2
```

### Call in R

We firstly load the library with:

``` {.r}
dyn.load("FloatExpansion1.dll")
.C("HsStart")
## list()
```

And we invoke the function with the help of the `.C` function, as
follows:

``` {.r}
.C("floatExpansion", base=2L, x=0.125, result="")$result
## [1] "[0,0,1]"
```

It works. But it would be better to have a vector as output, rather than
a string.

``` {.r}
dyn.unload("FloatExpansion1.dll")
```

Second dynamic linker: vector output
------------------------------------

To get the output as a vector, the additional modules we need are:
`Foreign.R`, `Foreign.R.Types` and `Data.Vector.SEXP`. They are provided
by the [`inline-r`
package](https://hackage.haskell.org/package/inline-r). The `[Int]` type
of the output list of the `floatExpansion` function must be converted to
`[Int32]`. We write a simple function `intToInt32` to help us to do the
conversion. It works with the help of the `Data.Int` module which is
imported via the `Foreign` module.

We end up with this module:

``` {.haskell}
-- FloatExpansion2.hs
{-# LANGUAGE ForeignFunctionInterface #-}
{-# LANGUAGE DataKinds #-}

module FloatExpansion where

import Foreign
import Foreign.C
import Foreign.R (SEXP)
import qualified Foreign.R.Type as R
import qualified Data.Vector.SEXP as DV
import Numeric (floatToDigits)

foreign export ccall floatExpansion :: Ptr CInt -> Ptr CDouble -> Ptr (SEXP s R.Int) -> IO ()

floatExpansion :: Ptr CInt -> Ptr CDouble -> Ptr (SEXP s R.Int) -> IO ()
floatExpansion base u result = do
  base <- peek base
  u <- peek u
  let expansion = map intToInt32 $ floatExpansion' (toInteger base) u
  poke result $ DV.toSEXP $ DV.fromList expansion

intToInt32 :: Int -> Int32
intToInt32 = fromIntegral

floatExpansion' :: RealFloat a => Integer -> a -> [Int]
floatExpansion' base u = replicate (- snd expansion) 0 ++ fst expansion
  where expansion = floatToDigits base u
```

We compile the library as before. And we load it in R as before:

``` {.r}
dyn.load("FloatExpansion2.dll")
.C("HsStart")
## list()
```

And we invoke the function with the help of the `.C` function, as
follows:

``` {.r}
.C("floatExpansion", base=2L, x=0.125, result=list(0L))$result
## [[1]]
## [1] 0 0 1
```

In fact, the output is a list with one element, the desired vector.

Let's write a user-friendly function:

``` {.r}
floatExpand <- function(x, base=2L){
  .C("floatExpansion", base=as.integer(base), x=as.double(x), result=list(integer(1)))$result[[1]]  
}
```

Let's compare it with my R function `num2dyadic`:

``` {.r}
library(microbenchmark)
microbenchmark(
  floatExpand = floatExpand(runif(1)),
  num2dyadic  = num2dyadic(runif(1)),
  times = 5000
)
## Unit: microseconds
##         expr    min      lq      mean  median      uq      max neval cld
##  floatExpand 12.049 16.9580  43.44155  23.205  35.253 8006.062  5000  a 
##   num2dyadic 43.286 98.8435 150.81104 128.964 167.788 3451.697  5000   b
```

It is faster. And I have checked that the two functions always return
the same results.

Moreover the "RHaskell" function allows more than the binary expansion,
for example the ternary expansion:

``` {.r}
floatExpand(1/3+1/27, base=3)
## [1] 1 0 1
```

Quite nice, isn't it ?

``` {.r}
dyn.unload("FloatExpansion2.dll")
```
