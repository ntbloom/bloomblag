---
title: "When Is It Okay to Kiss Your Cousin?"
date: 2020-05-14
author: ntbloom
draft: true
---

"Kissing cousins" remains one of my all-time favorite American folk-humor memes. I'm
still not exactly sure what it refers to, but I have some ideas, all hilarious:

> _Possible definitions:_
>
> - _cousins far enough apart where fornication is socially acceptable (kissing is a
>   euphemism)_
> - _cousins far enough apart where fornication is not socially acceptable but kissing
>   is (kissing is_ not _a euphemism)_
> - _two people who haven't spoken since the awkward family reunion when they were both
>   15 (she had a different last name!)_
> - _one in every five_ Arrested Development _jokes (best show ever?)_

While we'll never know exactly what happened at the lake house that summer, I think we
can agree that kissing cousins should refer in the most generalist sense to two people
who are a little too closely related to one another to be quite so friendly with one
another.

This contrasts nicely with marriage where they say that opposites attract and that the
two partners should be only similar enough that they agree in perpetuity not to murder
one another (see
[Henny Youngman](https://en.wikipedia.org/wiki/Henny_Youngman "take my wife...please")).

The problem of course is that finding "that perfect someone" can be a really daunting
task, and well, sometimes you gotta stick (wrong verb?) with what's right in front of
you. Is it ever okay to break social convention in such a manner?

### So what does this have to do with software?

If a person is just somebody with a genetic history and a prediliction for hanky panky,
then a kissing cousin is another person with a similar genetic history with whom hanky
panky may or may not be considered taboo, so you should probably think about it really
hard first. Likewise, if a computer program is just a collection of instructions to
accomplish certain tasks, then you should probably stay away from simultaneously using
two programs with the same capabilities. Whereas the first case produces hemophilia and
other genetic diseases, the latter can create performance bottlenecks, dependency hell,
and all sorts of other problems, or at least make you look bad on Stack Overflow:

```docker
# why does my container take so long to build?
FROM debian:latest
RUN apt-get update; \
  apt-get install -y --no-install-recommends \
# for the API
    python3-django \
    mysql-server \
# the other API
    python3-flask \
    python3-sqlalchemy \
    postgresql-common
...
```

This moral hygiene is in fact the basis of the so-called
["Unix Philosophy"](https://en.wikipedia.org/wiki/Unix_philosophy "Unix Philosophy - Wikipedia"),
where every program was supposed to do just one thing and do it well before passing
along the updated content to the next microtool.

For the most part this line of thinking has remained sound, as we tend to stay away
from using multiple programs and runtimes that are too closely related in order to
avoid peeing in the communal genetic pool. When was the last time you deployed a web
app that used React _and_ Vue?

Obviously the above examples are pretty clear violations of the single responsibility
principle, and no sane programmer would bring in more than one javascript framework on
the front end (jk -- please send me examples).  
But my original question still remains, is there a scenario where it's okay to do
something you would never admit to your programming friends?

### New Jersey's finest contribution to the world

Besides being the US state furthest removed from American folk culture, New Jersey is
known for its rock star programmers at Bell Labs in the 1970s and beyond. The value of
this contribution to the world of software and beyond is incalculable. In fact, let's
observe a short moment of silence to thank the developers of the UNIX operating system
that we don't connect to our remote servers using Powershell...

...

...

... _insert token Micro\$oft joke_ ...

...

...

... _for real, thank you!_ ...

...

...

Okay we're back.

In fact, while the authors of the C programming language, awk, sed, and others were
hammering on figuring out how to manipulate text on a computer that weighed 1,100 lbs
but only had 9kb of RAM, they were also very busy reinventing wheels everywhere they
went. Need to print the first line of a file? Gotcha covered five times over:

```fish
#!/usr/bin/sh
# print me the first line, please

# one way!
head -1 filename.txt

# yet another!
sed -n '1p' filename.txt

# I could keep going!
awk 'NR==1' filename.txt

# works if there aren't any numbers in the file...
cat -n filename.txt | grep 1 | cut -f 2
```

Of course our UNIX forefathers also gave us the tools to sort through the muck and find
out which of these solutions is the most performant and, by extension, the "right and
true" way forward. Using `time` we can determine which of the above choices makes the
most sense.

Let's run the same commands on a file that contains precisely 425,083 lines of text
which is a totally legitimate number of lines that I did not come up with by accident
in the process of attempting to make a file with 100,000 lines and forgetting how to do
it the first 5 times:

| command             | real time | user time | sys time |
| ------------------- | --------: | --------: | -------: |
| head                |    0.004s |    0.002s |   0.002s |
| sed                 |    0.079s |    0.070s |   0.009s |
| awk                 |    0.173s |    0.162s |   0.010s |
| cat with grep & cut |    0.886s |    0.384s |   0.633s |

So we have an obvious winner in `head`, which makes sense since of all the tools in the
shed, `head` is the only one whose primary job is to print n-lines of text starting at
the beginning of the file. The huge gap between the `cat` command and the rest also
makes sense, since only `cat` reads the file in its entirety before trying to find the
first line.

It's pretty clear at this point that at least in terms of this particular task, `head`
is a close cousin of `sed` and `awk` who are also close cousins of `grep` and `cut`.
It's hard to generalize from this very specific (dumb) task, but let's just say that
all of these tools should not be at the same 4th of July keg party at the river this
year if you know what I mean.

### But what if they could?

Let's imagine a hypothetical scenario where a very non-technical friend emails you a
CSV file containing some accounting data that he wants "to make some data science" on.
They've been told you are a "computer person" and might be able to help them "crunch
some numbers":

```plaintext
"Type","Date","Num","Name","Memo","Class","Clr","Split","Amount","Balance"
"Sales Receipt","01/03/2019","9054",,"DELIVERY CHARGE",,,"Undeposited Funds",35.00,35.00
"Sales Receipt","01/11/2019","9085",,"DELIVERY CHARGE",,,"Undeposited Funds",40.00,225.00
"Sales Receipt","01/21/2019","9100",,"DELIVERY CHARGE",,,"Undeposited Funds",60.00,320.00
"Sales Receipt","01/21/2019","9099",,"DELIVERY CHARGE",,,"Undeposited Funds",45.00,365.00
"Sales Receipt","01/22/2019","9102",,"DELIVERY CHARGE",,,"Undeposited Funds",35.00,435.00
"Sales Receipt","01/24/2019","9104",,"DELIVERY CHARGE",,,"Undeposited Funds",40.00,475.00
"Sales Receipt","02/04/2019","9123",,"DELIVERY CHARGE",,,"Undeposited Funds",45.00,520.00

...8530 more rows...
```

"The second to last column is the amount we charge for delivery," he tells you. "It's a
variable amount starting at \$35 based on your neighborhood. We're trying to find out
how many deliveries we made last year at the minimum rate. Can you help?"

You grind your teeth together. Thoughts swirl through your head. Do you drive over to
his house and show him how to open the CSV file in Excel on his laptop and teach him
the `sumif` method? Do you write a Python script to bring the file into a pandas
dataframe and print out a nice statistical analysis of his failing business model? Do
you spin up a quick Postgres container and put the thing into a format that's actually
worth something?

You remember you're on a Linux box. You should probably do this in `sed` because you
read on Hacker News that it was Turing Complete, and that means it's the correct tool
for any job. Then you remember that `awk` can split files into columns based on a
delimiter, and how hard can it be to learn how to declare and increment a variable
based on a single conditional for the rest of it?

You start to sweat and realize that you can do whatever the hell you want because this
project doesn't matter. You crack your knuckles, `head` the file to get a second look
at the format (it's so fast!), and type your command from pure memory. You'll `time` it
of course because who doesn't like to look at themselves in the mirror when they work
out?

```fish
$ time awk -F, '{ print $9 }' accounts.csv | grep ^35\.00 | wc -l
#> 2503
#>
#> real   0m0.026s
#> user   0m0.023s
#> sys    0m0.008s
```

After you email him the answer, your elation in realizing you solved his problem in
less than 2 minutes is quickly overwhelmed by the most painful variety of of kissing
cousin shame as you wallow in the fact that you really _should_ learn to use `awk`
properly by itself. Comingling regex engines is fine if your last name is Lannister but
this is America in 2020 and we believe in decency, people. So you work for 2 hours
reading crusty web pages written in plain html explaining the ins and outs of 1970s-era
text manipulators, only to come up with the following stinker:

```fish
$ time awk -F, '$9 ~ /35\.00/ { count++ } END { print(count) }' accounts.csv
#> 2503
#>
#> real    0m0.047s
#> user    0m0.044s
#> sys     0m0.003s
$ echo "well that was disappointing"
```

"sed is probably a much better tool for the job", you say, so you decide to try your
hand at `awk`'s kissing cousin. You look into it for 30 minutes or so before you find
[an amazing SO answer](https://stackoverflow.com/a/1782426/11269758 "never say never")
with a tagline "never say never" and decide that maybe `sed` needs a little help from
an old friend or two:

```fish
$ time sed -n '/,35.00/p' accounts.csv | wc -l
#> 2503
#>
#> real    0m0.018s
#> user    0m0.011s
#> sys     0m0.014s
```

That's a whole lot more reasonable, and in the process of your research you remember
that `grep` has a `-c` option to count occurrences (isn't that what most UNIX
programming is anyway, just remembering obscure flags you haven't used in 6 months?),
so you circle back to your first example:

```fish
$ time awk -F, '{ print $9 }' accounts.csv | grep -c ^35\.00
#> 2503
#>
#> real    0m0.050s
#> user    0m0.045s
#> sys     0m0.008s
```

Shit, it's 2am and you seem to be moving backwards. Didn't you solve this problem
already? Okay, one more time with `sed` and `grep -c` and then you're going to bed:

```fish
$ time sed -n '/,35\.00/p' accounts.csv | grep -c
#> 2503
#>
#> real    0m0.008s
#> user    0m0.006s
#> sys     0m0.008s
```

Hooray, you found a winner! You get a UNIX trophy! Let's tally up the scores:

| variant            |                      time saved | cumulative developer time |
| ------------------ | ------------------------------: | ------------------------: |
| awk with grep & wc |                        baseline |             1 min, 30 sec |
| pure awk           | <span class="neg">--21ms</span> |      2 hrs, 1 min, 30 sec |
| sed with wc        |    <span class="pos">8ms</span> |    2 hrs, 31 mins, 30 sec |
| awk with grep -c   | <span class="neg">--24ms</span> |    2 hrs, 32 mins, 30 sec |
| sed with grep -c   |   <span class="pos">18ms</span> |    2 hrs, 33 mins, 30 sec |

The good news is that you're way better at `sed`, `awk`, and `grep`, at least until you
forget all the flags again in 30 days. The bad news is that you just wasted 2 and a
half hours of your life to save less time than it takes the knot to form in the back
your throat when you see the guest list for your Uncle's birthday party next month. And
all for a project that you will never participate in again.

You shamefully clear your bash history, not so gently close the lid on your laptop, and
go to bed. Let's never talk about this night again to anybody.
