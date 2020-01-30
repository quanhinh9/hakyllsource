---
author: Stéphane Laurent
date: '2020-01-30'
highlighter: 'pandoc-solarized'
linenums: True
output:
  html_document:
    highlight: kate
    keep_md: False
  md_document:
    preserve_yaml: True
    variant: markdown
prettify: True
prettifycss: minimal
tags: 'R, maths, special-functions'
title: Complex incomplete elliptic integral of the second kind
---

The *incomplete elliptic integral of the second kind* is usually defined
by $$
E(\phi \mid k) = 
\int_0^\phi \sqrt{1 - k^2\sin^2t}\,\mathrm{d}t
$$ for $-\frac{\pi}{2} \leqslant \phi \leqslant \frac{\pi}{2}$ and
$k^2 \leqslant \frac{1}{\sin^2\phi}$.

It can be evaluated in R with the `gsl` package:

``` {.r}
f <- function(t, k) sqrt(1 - k^2*sin(t)^2)
phi <- 1; k <- 0.5
integrate(f, 0, phi, k = k)$value
## [1] 0.9648765
gsl::ellint_E(phi, k)
## [1] 0.9648765
```

That said, this elliptic integral is more generally defined by $$
E(\phi \mid m) = 
\int_0^\phi \sqrt{1 - m\sin^2t}\,\mathrm{d}t
$$ for $-\frac{\pi}{2} \leqslant \phi \leqslant \frac{\pi}{2}$ and
$-\infty < m \leqslant \frac{1}{\sin^2\phi}$.

Evaluation of $E(\phi \mid m)$
==============================

Thus, `gsl` allows to evaluate $E(\phi \mid m)$ for a positive value of
the parameter $m$ only, with the `ellint_E` function. However it is
possible to use `gsl` to evaluate $E(\phi \mid m)$ for a negative
parameter $m$, with the help of the *Carlson elliptic integrals* $R_F$
and $R_D$. These elliptic integrals are evaluated in `gsl` by the
functions `ellint_RF` and `ellint_RD`.

``` {.r}
E <- function(phi, m){
  if(phi == 0){
    0
  }else if(phi >= -pi/2 && phi <= pi/2){
    sine <- sin(phi)
    sine2 <- sine*sine
    cosine2 <- 1 - sine2
    oneminusmsine2 <- 1 - m*sine2
    sine * (gsl::ellint_RF(cosine2, oneminusmsine2, 1) -
              m * sine2 * gsl::ellint_RD(cosine2, oneminusmsine2, 1) / 3)
  }else{
    NaN
  }
}
```

Let's try it:

``` {.r}
E(phi = 1, m = -5)
## [1] 1.493736
```

You can compare with
[`EllipticE[1,-5]`](https://www.wolframalpha.com/input/?i=EllipticE%5B1%2C+-5%5D)
on WolframAlpha.

Here is how the function looks like for a fixed value of $\phi$:

``` {.r}
opar <- par(mar = c(5,4,1,1))
phi <- pi/4
curve(
  E(phi, x), from = -5, to = 1/sin(phi)^2, lwd = 2, 
  xlab = expression(italic(m)), 
  ylab = expression(paste(italic(E), "(", italic(phi1), " | ", italic(m), ")"))
)
```

![](figures/EllipticII-curveE-1.png)

``` {.r}
par(opar)
```

Extending $E(\phi \mid m)$ to arbitrary real $\phi$
===================================================

There is a continuous extension of $E(\phi \mid m)$. It is given by the
formula $$
E(\phi + k\pi \mid m) = 2 k E\Bigl(\frac{\pi}{2} \mid m\Bigr) + E(\phi \mid m)
$$ for any $k \in \mathbb{Z}$.

``` {.r}
E <- function(phi, m){
  if(phi == 0){
    0
  }else if(phi >= -pi/2 && phi <= pi/2){
    sine <- sin(phi)
    sine2 <- sine*sine
    cosine2 <- 1 - sine2
    oneminusmsine2 <- 1 - m*sine2
    sine * (gsl::ellint_RF(cosine2, oneminusmsine2, 1) -
              m * sine2 * gsl::ellint_RD(cosine2, oneminusmsine2, 1) / 3)
  }else if(phi > pi/2){
    k <- 0
    while(phi > pi/2){
      phi <- phi - pi
      k <- k + 1
    }
    2*k*E(pi/2, m) + E(phi, m)
  }else{
    k <- 0
    while(phi < -pi/2){
      phi <- phi + pi
      k <- k - 1
    }
    2*k*E(pi/2, m) + E(phi, m)
  }
}
```

Let's try it:

``` {.r}
E(phi = 7, m = -5)
## [1] 12.25899
```

and compare with
[`EllipticE[7,-5]`](https://www.wolframalpha.com/input/?i=EllipticE%5B7%2C+-5%5D)
on WolframAlpha.

Extending $E(\phi \mid m)$ to complex $\phi$
============================================

Now we will construct the analytical continuation of $E(\phi \mid m)$.

I found a nice implementation of the Carlson elliptic integrals
[here](https://www.codeproject.com/Articles/566614/Elliptic-integrals),
which can easily be generalized to complex values of the parameters:

``` {.r}
RF <- function(x, y, z, minerror = 1e-7){
  x <- as.complex(x); y <- as.complex(y); z <- as.complex(z)
  dx <- dy <- dz <- Inf
  while(max(dx,dy,dz) > minerror){
    lambda <- sqrt(x*y) + sqrt(y*z) + sqrt(z*x)
    x <- (x + lambda) / 4
    y <- (y + lambda) / 4
    z <- (z + lambda) / 4
    A <- (x+y+z) / 3
    dx <- Mod(1 - x/A)
    dy <- Mod(1 - y/A)
    dz <- Mod(1 - z/A)
  }
  E2 <- dx*dy + dy*dz + dz*dx
  E3 <- dy*dx*dz
  (1 - E2/10 + E3/14 + E2*E2/24 - 3*E2*E3/44 - 5*E2*E2*E2/208 + 
      3*E3*E3/104 + E2*E2*E3/16) / sqrt(A)
}
RD <- function(x, y, z, minerror = 1e-7){
  x <- as.complex(x); y <- as.complex(y); z <- as.complex(z)
  dx <- dy <- dz <- Inf
  s <- 0; fac <- 1
  while(max(dx,dy,dz) > minerror){
    lambda <- sqrt(x*y) + sqrt(y*z) + sqrt(z*x)
    s <- s + fac / (sqrt(z) * (z + lambda))
    fac <- fac/4
    x <- (x + lambda) / 4
    y <- (y + lambda) / 4
    z <- (z + lambda) / 4
    A <- (x + y + 3*z) / 5
    dx <- Mod(1 - x/A)
    dy <- Mod(1 - y/A)
    dz <- Mod(1 - z/A)
  }
  E2 <- dx*dy + dy*dz + 3*dz*dz + 2*dz*dx + dx*dz + 2*dy*dz
  E3 <- dz*dz*dz + dx*dz*dz + 3*dx*dy*dz + 2*dy*dz*dz + dy*dz*dz + 2*dx*dz*dz
  E4 <- dy*dz*dz*dz + dx*dz*dz*dz + dx*dy*dz*dz + 2*dx*dy*dz*dz
  E5 <- dx*dy*dz*dz*dz
  3*s +
    fac * (1 - 3*E2/14 + E3/6 + 9*E2*E2/88 - 3*E4/22 - 9*E2*E3/52 + 3*E5/26 -
             E2*E2*E2/16 + 3*E3*E3/40 + 3*E2*E4/20 + 45*E2*E2*E3/272 -
             9*(E3*E4 + E2*E5)/68) / A / sqrt(A)
}
```

Now here is the implementation of the incomplete elliptic integral of
the second kind allowing complex values of $\phi$:

``` {.r}
E <- function(phi, m){
  if(phi == 0){
    0
  }else if(Re(phi) >= -pi/2 && Re(phi) <= pi/2){
    sine <- sin(phi)
    sine2 <- sine*sine
    cosine2 <- 1 - sine2
    oneminusmsine2 <- 1 - m*sine2
    sine * (RF(cosine2, oneminusmsine2, 1) -
              m * sine2 * RD(cosine2, oneminusmsine2, 1) / 3)
  }else if(Re(phi) > pi/2){
    k <- 0
    while(Re(phi) > pi/2){
      phi <- phi - pi
      k <- k + 1
    }
    2*k*E(pi/2, m) + E(phi, m)
  }else{
    k <- 0
    while(Re(phi) < -pi/2){
      phi <- phi + pi
      k <- k - 1
    }
    2*k*E(pi/2, m) + E(phi, m)
  }
}
```

Let's try it:

``` {.r}
E(phi = 7+2i, m = -5)
## [1] 7.763389+5.631026i
```

and compare with
[`EllipticE[7+2I,-5]`](https://www.wolframalpha.com/input/?i=EllipticE%5B7%2B2I%2C+-5%5D)
on WolframAlpha.

I also find the same results as the ones of WolframAlpha when $\phi$ is
real and $m$ is complex, but not always when both are complex.

``` {.r}
E(phi = 1+1i, m = -5i) # same as Wolfram
## [1] -0.66421+1.746623i
E(phi = 7+2i, m = -5i) # not the same as Wolfram
## [1] -0.78406+2.057178i
```