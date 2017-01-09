---
layout: post
title: Some numerology of repeating decimals
---

It’s been a very busy summer. I have been doing a lot of interesting things, but
none of them are quite ready to write about. Instead, here’s some numerology I
came across while programming.

When working with computers and mathematics, we are often looking for patterns
to make sure we understand what is going on. One small, simple example, is
recognizing fractions when the computer prints out numbers. A factor like 0.0625
could be a clue that we missed a factor of 16 somewhere, or perhaps called a
function too many times in a loop.

Here are the reciprocals of the first twenty integers in decimal form, courtesy
of Python:

~~~
In [1]: for n in range(20): print (n + 1, 1 / (n + 1))
1 1.0
2 0.5
3 0.3333333333333333
4 0.25
5 0.2
6 0.16666666666666666
7 0.14285714285714285
8 0.125
9 0.1111111111111111
10 0.1
11 0.09090909090909091
12 0.08333333333333333
13 0.07692307692307693
14 0.07142857142857142
15 0.06666666666666667
16 0.0625
17 0.058823529411764705
18 0.05555555555555555
19 0.05263157894736842
20 0.05
~~~

As taught in middle school mathematics, you can see that decimal representations
of fractions either truncate or they have a repeating pattern. Non-repeating,
infinite series of numbers are reserved for irrational numbers like π.

One thing I was thinking of yesterday is that most of the repeating decimals in
fractions repeat the same digit (like 1/3 = 0.333…), or they repeat a pattern of
six digits (like 1/7 = 0.142857142857…). This is great for computers, since
floating point numbers typically have about fifteen significant digits, as seen
in the Python output above. A pattern of six repeating digits is pretty easy to
recognize.

Why are we so lucky? Why don’t common fractions have longer patterns of
repeating digits? For example, 1/17 repeats after fifteen digits, so its hard to
recognize that the pattern is repeating in the double precision floating point
format above. Yet, all the reciprocals of numbers smaller than 17 and many other
common fractions have repeating sequences of 1, 2, or 6 digits. What is special
about six repeating digits?

The answer turns out to be a fluke in the base-10 number system. As we learned
in school, the way to convert a repeating decimal back into a fraction is to use
a little algebra. Let x = 1/7, or 0.14285714… in decimal. Then 1 000 000  x =
142857.14285714…. Subtracting the two equations gives an integer equation: 999
999 x = 142857. Thus x = 142 857 / 999 999, which simplifies to x = 1/7.

This number 999 999 (and ones like it, like 9 and 99) are special to base-10
numbers. Its factorization (a general property of integers in any number system)
is 999 999 = 3² · 7 · 11 · 13 · 111. That’s cool! In base 10, we already know
that fractions with denominators that factor into 2’s and 5’s don’t repeat. Now
we see that if the denominators contain a couple factors of 3, or a factor of 7,
11, or 13, the factor repeats in six digits (or less, for special cases like 1/9
or 1/11). We also see what is special about 1/17: since 17 is not a factor of
999 999, the repeating decimal for 1/17 has many more digits.

In summary, thanks to our ten fingers, we use the base-10 number system. Ten is
a product of two primes 2 and 5, so many fractions have finite, non-repeating
decimal representations. Next, 10 – 1 = 9, which is 3 · 3, so fractions with up
to two factors of three in their denominators along with factors of 2 and 5 end
in a single, infinitely repeating digit. And the very lucky factorization of the
number 999 999  into the other small primes guarantees that numbers with single
factors of 7, 11, or 13 in their denominators repeat in patterns of six digits.
