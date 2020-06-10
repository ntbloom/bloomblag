---
title: "What If We Used npm as a Python Runner?"
date: 2020-06-10
author: ntbloom
draft: true
---

Python is an amazing language and its interpreted nature makes for super easy
development workflows. Without the need for a compilation step or complicated builds,
running your code can be as simple as running `python app.py` in your project's root
directory.

However, like every problem that is infinitely more complicated than that blog post you
read last week, it's usually not a simple as that. You're probably using a virtual
environment. Maybe (hopefully?) you're practicing TDD and your code is largely run
using a tool like pytest that has a million flags that you have to keep straight:

```bash
$ source venv/bin/activate
(venv) $ python3 -m pytest tests/good_tests/ -x -ff -m "not slow"
(venv) $ python3 -m pytest tests/shitty_tests/ --runxfail
(venv) $ python3 -m pytest --cov=project tests/ --runxfail
(venv) $ deactivate
$
```

None of these complications are deal-breakers but together they make for maybe 5 or 10
different variations of `run code` that would be nice to keep track of in a meaningful,
shareable way.

Of course you get all of this for free with a good IDE like Pycharm, but what about the
poor Linux user who refuses to adopt modern development practices and insists that vim
and tmux is all the IDE anybody ever needs?

There has to be a better way to keep all of this straight. What if there were a tool
that many developers already have installed on their machines that was capable of
mapping prickly one liners to easily remembered reusable and shareable commands?

Good news, hackers: npm is here to save the day.

### Bring out the beast

I won't lie; npm is kind of a nightmare. Between the number of dependencies you have no
idea what the hell they do and innumerable
[left-pad](https://www.davidhaney.io/npm-left-pad-have-we-forgotten-how-to-program/ "NPM & left-pad: Have We Forgotten How To Program?")
lookalikes in the public repos, staying on top of a complicated production javascript
installation -- at least in terms of dependency management -- can be a treacherous
chore. However, as a script runner, npm in my opinion can't be beat. Don't believe me?
Run these three commands from the root of a non-javascript project and see how quickly
you can get running with reusable bash one-liners:

```bash
$ echo "loglevel=silent" > .npmrc
$ echo "{\"scripts\":{\"hello\":\"echo 'Hello, World!'\"}}" > package.json
$ npm run hello
#> Hello, World!
```

What's happening of course is that you just wrote the world's simplest javascript
application (tentative clickbait blog post title: _"10 Reasons To Eliminate All
Javascript From Your Next Javascript App")_ and are now using it to replace bash as
your main command-line runner.

So how does this work for Python? Well, the beauty of npm is that it executes scripts
from the same directory where it finds a package.json file, so no matter where the
calling directory of an npm command, you don't have to fuss around with keeping
relative references straight. On top of that, since you have access to the same amazing
tools that your command-line has, you aren't limited in what you can accomplish. Want
to be done with activating and deactivating virtual environments? Just invoke the
virtual interpreter directly:

```json
{
  "scripts": {
    "py-app": "venv/bin/python3 app.py"
  }
}
```

With this in your `package.json` you can now run your app from any location in the
project directory by calling `npm run py-app` without having to activate or deactivate
the virtual environment. You can even append command-line flags to the end of your
commands to mimick as if you were calling directly from the command line.
`npm run py-app arg1 arg2 arg3` in this case is equivalent to
`python3 app.py arg1 arg2 arg3` but with the convenience of being able to store base
configurations in source control.

### Some practical examples?

This is more of a proof of concept than a tried-and-true method, but here are some
other interesting hacks. Here are a few that I just thought of, but I'm sure there are
dozens more.

---

**Leave yourself a comment:**

```json
{
  "py-app": "venv/bin/python3 app.py # runs the main app",
  "python": "venv/bin/python3 # get yourself a REPL"
}
```

Note that this prevents you from adding additional args at the end of the command since
the `#` is greedy in this case. In my opinion this is the biggest drawback to this
pattern as we're now forced to choose between extensibility and documentation. Of
course as with most problems in the software world, we can blame this almost
exclusively on javascript for having institutionally prevented us from writing comments
in configuration files.

---

**Manage dependencies with pip:**

```json
{
  "pip": "venv/bin/pip # avoid installing on system python",
  "pip-save": "venv/bin/pip freeze > requirements.txt",
  "make-venv": "python3 -m venv venv && venv/bin/pip install -r requirements.txt"
}
```

Potential commands include `npm run pip install requests` or `npm run pip-save`.

---

**Specify different pytest configurations:**

```json
{
  "pytest": "venv/bin/pytest # extensible template",
  "pytest-all": "venv/bin/pytest --cov=myproject tests/ --runxfail",
  "pytest-api": "venv/bin/pytest tests/api --maxfail=1 --failed-first",
  "pytest-database": "venv/bin/pytest tests/database --maxfail=1 --failed-first",
  "pytest-ff": "venv/bin/pytest --maxfail=1 --failed-first"
}
```

Invoke pytest manually with `npm run pytest tests/ --ff -x` or use a preconfigured
option like `npm run pytest-all`.

---

**Run linting or autoformatting tools:**

```json
{
  "lint": "venv/bin/mypy --config-file mypy.ini src/",
  "autoformat": "venv/bin/black -l 80 ."
}
```

---

You can even expand the pattern to automate git workflows if you like living
dangerously:

```json
{
  "commit": "git add . && git commit -m",
  "yolo-commit": "git add . && git commit -m 'runs on my machine...' && git push --force origin master"
}
```

### A warning before jumping in

Let's be abundantly clear: this is a **hack**. This will not work on a remote server or
in a container where nodejs is not a dependency, for example. It is also not as robust
and configurable as _real_ build tools like make, which is probably a way better tool
for the job and you (I?) should probably learn it.

Nevertheless, there is a beautiful elegance in this sort of design pattern where there
is literally nothing lost by having a single json file in a project directory. If node
isn't on a particular system, run the commands another way. Worst case scenario you
have to manually type _one line_ to get your program or tests to run, which is also
fine because all of the flags are already written down in your handy `package.json`
file even if you won't call on it directly.

So will I be using this on a daily basis? I don't know, probably not. But it's cool to
know that it's there. And it can't be any more fragile than all of the left-pad cousins
holding up the internet.
