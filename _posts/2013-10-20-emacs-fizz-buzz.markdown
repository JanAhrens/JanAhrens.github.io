---
title: FizzBuzz in Emacs
layout: post
date: 2013-10-20
---

I've been playing around with
[GNU Emacs](http://www.gnu.org/software/emacs/) for some time, but
never really got myself to use it on a daily basis.  To change this I
decided that it's time to learn a little about
[Emacs Lisp](http://en.wikipedia.org/wiki/Emacs_Lisp) and hopefully
gain the ability to bend the editor to my needs. I choose the
[FizzBuzz Kata](http://codingdojo.org/cgi-bin/wiki.pl?KataFizzBuzz) as
a starting point and want to share what I learned along the way.

Emacs is not only an editor, it's in fact a powerful Lisp
interpreter.  Most of the built-in text editing functionality of Emacs
is written in Emacs Lisp.

The first thing that you'll see when you start Emacs is the
[`*scratch*` buffer](http://www.gnu.org/software/emacs/manual/html_node/emacs/Lisp-Interaction.html).
In that buffer you can directly evaluate any Lisp expression.  To use
Emacs as a calculator for example you only have to type `(+ 23 19)` in
that buffer, put your cursor after the closing parenthesis
and press `Control + j` - voila there's "the answer"&trade;.

Emacs also ships with a built in testing framework called
[`ERT` - Emacs Regression Testing](http://www.gnu.org/software/emacs/manual/html_node/ert/),
which I choose for my quest to implement FizzBuzz.

My first task was to write a failing test and learn how to execute it,
so I came up with this beautiful crafted test-case.

```cl
(ert-deftest the-truth ()
  "the truth should be working"
  (should (equal t 0)))
```

Since I'm a lazy person, I tried only to read the
[necessary parts](http://www.gnu.org/software/emacs/manual/html_node/ert/Running-Tests-Interactively.html#Running-Tests-Interactively)
of the ERT documentation. My first attempt to run the test was
executing the `ert` command with `M-x ert RET`. The command was
fitting, but I received a prompt report that there were zero passed
and zero failed tests.

After a little more digging I found out that I first had to evaluate
the test-case.  Typing the text into the `*scratch*` buffer doesn't
tell Emacs to evaluate the expression and ERT will only run the
globally defined test-cases.  I had to go to the end of the test-case
(after the closing parenthesis) and type `C-x C-e`. A second `M-x ert
RET` gave me the familiar red "F" output.

The rest of the Kata was straight forward. My result looks like this:

```cl
(defun translate-fizz-buzz ()
  "translate a number into it's fizz-buzz value"
  (if (equal 0 (mod x 15))
      "FizzBuzz"
    (if (equal 0 (mod x 3))
        "Fizz"
      (if (equal 0 (mod x 5))
          "Buzz"
        x))))

(ert-deftest normal-number ()
  "normal numbers should not be translated"
  (should (equal 2 (translate-fizz-buzz 2))))

(ert-deftest divisible-by-three ()
  "numbers divisible by three should be replaced with Fizz"
  (should (equal "Fizz" (translate-fizz-buzz 3))))

(ert-deftest divisible-by-five ()
  "numbers divisible by five should be replaced with Buzz"
  (should (equal "Buzz" (translate-fizz-buzz 5))))

(ert-deftest divisible-by-three-and-five ()
  "numbers divisible by five and three should be replaced with FizzBuzz"
  (should (equal "FizzBuzz" (translate-fizz-buzz 15))))
```

If I got you interested and you also want to experiment with ELT, keep
in mind that you have to evaluate every function again as soon as you
make changes (at least when you use the `*scratch*` buffer).

This small exercises got me hooked to spent more time learning about
Emacs and Lisp. I really like the concept that you have a powerful
programming environment inside my editor and the possibility to bend
the editor to my needs. The next weeks will show if the excitement
will last. If you're interested in my Emacs configuration (which is
not really advanced at the time of writing this post), have a look at
my
[dotfiles repository](https://github.com/JanAhrens/dotfiles/tree/master/.emacs.d/).
