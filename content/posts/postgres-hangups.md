---
title: "Postgres Hangups"
date: 2020-05-10
author: ntbloom
draft: true
---

Postgresql is amazing. Like, I want to go back in time and give the person (people? had
to have been people, right?) who wrote it a hug. I started my software engineering
journey as an excel monkey, and after cutting my teeth on `vlookup` hell, I was blown
away by the power, simplicity, and safety guaranteed by a full-fledged database engine.

> _Microsoft Excel is the world's greatest GUI-based non-transactional relational
> database with no type safety, user permissions layer, or logging. Throw it on the
> shared company fileserver and have a ball! Need a backup? Just save it as the same
> file and location but add "version \[n\]" to the end of the filename._
>
> -- "The Computer Guy" at your workplace

Needless to say, there's a reason that relational databases are one of the most popular
ways to store content, especially when the data are well-structured.

I'll grant that Postgres is not the _only_ relational database solution, and it's not
even the only _open-source_ relational database solution (shoutout to Libre Office
Base?), but for hackers like me who aren't managing 500,000,000 stock transactions per
second (is that a real number? I don't know), it's a fantastic solution.

However, there are some hangups that continue to get me time and time again, especially
after being away from SQL for awhile and coming back. I'll share a few of my biggest
annoyances, not because they are legitimate complaints, but if I write a blog post
about it then I will forever cement them into my brain and I won't have to think about
them any more. Or maybe some other SQL-curious excel poweruser will come to the dark
side and find these warnings to be helpful. Either way, these are some of the biggest
problems I've been having lately.

### Fun with NULL

I get it. `NULL` is
[the worst thing that's ever happened to computer science](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/ "Null References: The Billion Dollar Mistake"),
which is only tangentially related to my complaints about how Postgresql treats `NULL`,
but posting links referencing people who are 1,000x smarter than I am makes me look 2x
smarter than I really am which in turn makes you think I'm at least 1.5x the engineer I
really am. Thanks for the morale boost.

My issue with Postgresql and `NULL`, and I suppose the whole SQL standard in general,
is the idea that `NULL` itself cannot be a value. As in, the absence of a value does
not constitute a criterion for query matching. Which is fine and likely relates to
avoiding null pointer errors which I guess makes my Tony Hoare reference more relevant
after all.

It's all academic until you start writing `WHERE` clauses on rows that may or may not
contain `NULL` values. Which might happen a lot if you're mapping objects in an OO
language like Python where `None` is a perfectly reasonable value for an instance
variable but happens to convert to `NULL` in your database adapter:

```python3
# whales.py
# let's go whaling today!

white_whale = Whale(
    name="Moby Dick",
    species="Sperm Whale",
    dead=False
    killed_by=None
)

whale2 = Whale(
    name="whale2",
    species="Sperm Whale",
    dead=True,
    killed_by="Captain Ahab"
)

whale3 = Whale(
    name="whale3",
    species="Grey Whale",
    dead=True,
    killed_by="Starbuck"
)
```

Let's put these into our whaling database, because we all have one of those:

```postgresql
DROP TABLE IF EXISTS whales; --it definitely doesn't
CREATE TABLE whales (
  name TEXT NOT NULL,
  species TEXT NOT NULL,
  dead BOOL NOT NULL,
  killed_by TEXT);

INSERT INTO whales
VALUES
  ('Moby Dick', 'Sperm Whale', FALSE),
  ('whale2', 'Sperm Whale', TRUE, 'Captain Ahab'),
  ('whale3', 'Grey Whale', TRUE, 'Starbuck');
```

So far so good. We can now use a simple `SELECT` query to find out who is still alive.
Let's see how the results look:

```postgresql
SELECT name
FROM whales
WHERE dead = FALSE;

--------

--> Moby Dick
--> (1 row)
```

Great work, our database is doing its job! And Moby Dick is still alive (spoiler
alert?). This is way better than excel. Now let's see how Captain Ahab did on his
whaling trip (aside from the last 24 hours or so):

```postgresql
SELECT name
FROM whales
WHERE killed_by = 'Captain Ahab';

--------

--> whale2
--> (1 row)
```

Awesome stuff. This is really going great. I can't imagine a scenario where these
queries don't work like they're supposed to. For instance, if there are 3 whales in the
database, and a query on `=` returns 1 whale, the same query with a `!=` should return
the other 2, right?

```postgresql
SELECT name
FROM whales
WHERE killed_by != 'Captain Ahab';

--------

--> whale3
--> (1 row)
```

WTF, postgres? Captain Ahab did not kill Moby Dick (did you even read the book?), so
Moby Dick should clearly get returned by the query that contains all whales not killed
by Captain Ahab.

In reality, Postgresql cannot make such a check because `NULL` itself is the _absence
of value_, not a value equalling nothing like it does in Python (at least at the higher
level). So we're forced to make a small sacrifice to the SQL syntax gods and rephrase
our query:

```postgresql
SELECT name
FROM whales
WHERE
  killed_by != 'Captain Ahab'
  OR killed_by is NULL;

--------

--> Moby Dick
--> whale3
--> (2 rows)
```

Phew, got it, but this is pretty tedious to have to think about on every query. Again,
the annoyance of this doesn't come from the fact that I don't know how to hand-write
the proper SQL queries to extract the information that I need. It's that when dealing
with layers and layers of abstractions like an ORM or other glue code, there are some
extra steps that need to be taken to make sure that the results are correct. Missing a
single row or two when you're writing a higher-level function can produce some
insidious bugs if you're not ready for it.

### Single vs double quotes

I do not claim to be a particularly expert programmer, but I am comfortable working in
numerous different languages. For the most part, languages are agnostic about the use
of single versus double quotes for strings. I am also an adamant user of autoformatters
if they exist. My favorites are
[Black](https://github.com/psf/black "github:psf/black"),
[Prettier](https://prettier.io/ "prettier.io"),
[Clang-format](https://clang.llvm.org/docs/ClangFormat.html "Clang 11 Documentation"),
and [Rustfmt](https://github.com/rust-lang/rustfmt "github:rust-lang/rustfmt") (special
kudos to Rustfmt, not because I know much about Rust, but for being officially
supported by the language foundation itself to ensure that 95% of Rust code ever
written will follow the standard). All of these tools convert strings (among other
things) to whatever format is either preferred or makes the most sense (Black, for
example will default to `"` for simple cases but will use `'` if it means fewer escape
characters). Long story short, I always use the double quotes because that's how my
American brain works and I assume that the autoformatter will take care of converting
it to whatever it's supposed to be.

Postgresql on the other hand is very particular about its quotes. Double quotes refer
exclusively to column names and single quotes refer to strings. Trying to reference a
row value with a double quote typically results in a very confusing and not very
helpful error message:

```postgresql
SELECT name
FROM whales
WHERE killed_by = "Captain Ahab";

--------

--> ERROR: column "CAPTAIN AHAB" does not exist
--> Line 1: SELECT name FROM whales WHERE killed_by = "Captain Ahab";
-->                                                   ^
```

I'm not asking Postgresql to not enforce the `'` vs. `"` rule (okay I am -- can we get
support for double quoted strings when the user clearly means a string?), but I would
like to at least see a more useful error. The `^` pointing to the double quote is kind
of helpful, but not really. I would much prefer something along the lines of
`ERROR: column "CAPTAIN AHAB" does not exist. Did you mean to use single quotes: 'Captain Ahab'?`
This basically what the Rust compiler does, and referencing Rust in a blog post (two
times!) definitely gets your blog post lots of hits. This one took me 45 minutes of
head banging and returning to the problem the next day the first time I ran into it
several years ago and I'm sure I wasn't the only newcomer to face such hardships.

By now I know exactly what this error means and I can fix the mistake within seconds of
making it, but I don't feel like memorizing foggy error messages should be considered a
prerequisite for being a good engineer. I still make this error all of the time,
including today as I was writing this post.

### Error messages

Speaking of error messages, Postgreql is known for the most terse, confusing, and
unhelpful error messages. For example, throw a trailing comma into a query and you will
be met with an error that might as well be written `ERROR: QUERY BAD`:

```postgresql
--looks innocuous...
INSERT INTO whales
VALUES
  ('whale4', 'Finback Whale', FALSE),
  ('whale5', 'Sperm Whale', FALSE),
  ('whale6', 'Grey Whale', FALSE),
;

--------

--> ERROR: syntax error at or near ";"
```

Technically Postgres is correct that the error is either near or at the semicolon (and
so cruel to use `";"` to refer to the problem spot, as if wrapping words in double
quotes is acceptable), but again this is not helpful. It would be quite easy for the
query parser to filter these sorts of problems and respond with a helpful message. For
example:

- `ERROR: trailing comma found in LINE 4`. Shows the line number and the exact mistake.
  This can't be very hard to add.

- `ERROR: expected 4 VALUES but only 3 provided`. Indicates that you might have an
  extra comma somewhere. Very common pattern in compilers and interpreters.

- `ERROR: you're not very good at this are you?`. They're not wrong.

- `ERROR: perhaps you should try this in Excel`. Oof.

Of course these kind of errors usually happen when deleting lines from existing code
when you forget to remove the trailing comma. The
[javascript community](https://gist.github.com/isaacs/357981 "gist.github.com/isaacs/comma-first-var.js")
has produced some "comma-firsters" who think they have this problem all figured out
"FROM AN INSTITUTIONAL PERSPECTIVE FOR ALL PROGRAMMING LANGUAGES!", but I'm not
convinced:

<!-- prettier-ignore -->
```javascript
// who codes like this?
const numbers = [
  1
  , 2
  , 3
  , 4
  , 5
  , 6
];
// they're probably wearing toe-shoes
```

### PostgreSQL is still pretty awesome

Okay, who am I kidding? Postgresql is totally amazing and the quirks described above
are so insignificant that I will simply learn them and move on. And along my journey
I'm finding a whole bunch of other amazing tools in the processs that make it way
easier to do otherwise tedious tasks.

For example, I'm really digging the `RETURNING` keyword when used alongside an
auto-incrementing primary key:

```postgresql
CREATE TABLE tasks (
  task_id SERIAL PRIMARY KEY,
  task TEXT);

/*...insert 1,233 tasks...*/

INSERT INTO tasks
VALUES
  ("take a shower and call your mom")
RETURNING task_id;

--------

--> 1234
```

Other newer features include full-text searching to provide an alternative to
Solr/Elastic for users who already have their data stored in a Postgres database. The
example below shows how the search engine converts different variants of the English
word 'walk' to the same root stem for fuzzy matching:

```postgresql
SELECT to_tsvector(
  'english',
  'walk walks walked walking');

--------

--> 'walk':1,2,3,4
```

There are some other (weird?) use cases that make me really happy that they exist even
though I will never use them. For example there's the
[geometric types](https://www.postgresql.org/docs/12/datatype-geometric.html "PostgresSQL Documentation 12: 8.8 Geometric Types")
that I can only hope is used by teenage hackers to try to cheat on their high school
geometry homework only to learn geometry in the process:

> _Confession: I tried to write an example for how to cheat on high school geometry
> homework using Postgresql's geometric data types and can't figure out how to use
> them. Good luck, kids!_

### The Solution? Become a Better Engineer!

This post was written mostly in jest and as a way to solidify workarounds for problems
that trip me up so I can start writing better SQL. But the point remains, that our
tools are not always a perfect match for our brains, and we as engineers are
responsible for figuring out how to use them for our needs.

I say a perfect match for our brains because the architects of Postgresql likely
designed it in such a way that they had no choice but to make the decisions the way
that they did. For example, does the average consumer of data from Postgres even
interact directly with the database as opposed to an ORM like Sqlalchemy or
ActiveRecord? If the answer is no, then issues like better error messaging is totally
moot because the users who require more explicit error messages are probably getting
them from Python or Ruby. Likewise, my guess is that your average seasoned DBA is well
aware that `NULL` is not a value and probably prefers it that way (I would love an
explanation of this reasoning if that person is you). I'm sure there is a major
safety/performance reason in there somewhere.

So how do I move forward? Get over it? Learn why things are the way that they are?
Practice, practice, practice until I don't think about it any more? All of those are
likely the right explanation, and how I'm going to go about doing it. At the very least
I now have something on the public record that I can google for to remember how to
convert the storyline of _Moby Dick_ into an ~~excel spreadsheet~~ Postgresql database.
