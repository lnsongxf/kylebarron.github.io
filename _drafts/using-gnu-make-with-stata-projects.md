---
layout: "post"
title: "Using GNU Make with Stata projects"
date: "2018-03-21 13:45"
---


I've thought about this before too. I don't currently have any projects that are large enough to absolutely necessitate using a program like Make (currently I number my files to keep dependencies straight). However, in general I'm of the opinion that I'd rather incorporate a program like Make while my program is small, when it's easiest to add to the Makefile as I go along.

Mauricio created his own Make-like system in Python, but isn't developing it anymore, and I don't want to keep up development of my own version of this when great open-source, well-supported tools exist.

I agree that GNU Make can be confusing, but I think one of the main reasons why it's confusing is that most documentation using it is geared towards C or C++ projects, with very complicated dependency structures. The nice thing about Make is that it's language independent and just uses shell commands. As long as your projects take place on a Linux/Unix or Mac environment, Make will always work.

I just put together a basic example of make with shell commands in a Git repository. Check it out with

```bash
git clone https://github.com/kylebarron/make-test.git
cd make-test
make
make dependency1
```

It should output
```
> make
bash dep1.sh
this is dependency 1
bash dep2.sh
this is dependency 2
# make dependency1
# make dependency2
bash final.sh
This is final.sh
> make dependency1
bash dep1.sh
this is dependency 1
```

Look inside the Makefile and you'll see how simple it is. The Makefile is literally just
```makefile
all: final

dependency1:
	bash dep1.sh

dependency2:
	bash dep2.sh

dependency3:
	bash dep3.sh

final: dependency1 dependency2
	bash final.sh
```

Typing just `make` runs the first "target", the `all` step. Since `all` says to run `final`, Make searches for that
target explanation. The items after `final:` list its dependencies.
Since it lists `dependency1` and `dependency2`, Make runs whatever is specified for each of those, then runs whatever is specified for `final`.
Note that `dependency3` is not run when `final` is run, since it's not listed as a dependency.

You can also ask Make to run a specific target. So `make dependency1` runs `dependency1` and whatever dependencies it has.

Instead of listing dependencies after the colon, you could also list them as `make dependency` inside the clause, like
```makefile
final:
	make dependency1
	make dependency2
	bash final.sh
```

I think this is simple enough to be used for Stata jobs, where instead of `bash dep1.sh`, you use `stata-mp -b do filename.do`.

**Important: you must use tabs for indentation**. I use spaces instead of tabs for most programming, like Stata and Python, but Make will only work with tabs. I use Atom for everything, and it automatically switches to using tabs when I name a file `Makefile`.

Here are a few GNU Make tutorials written with data science in mind; I particularly like the first one:

- [GNU Make for Reproducible Data Analysis](http://zmjones.com/make/)
- [Why Use Make](https://bost.ocks.org/mike/make/)
- [Makefiles for fun and profit](http://www.jonzelner.net/statistics/make/reproducibility/2016/06/01/makefiles/)

A popular modern alternative to Make is luigi, which is developed by Spotify and widely-used, but I think that's actually overkill for Stata jobs.



There are also some language-specific workflow programs like [drake](https://github.com/ropensci/drake) for R, but that's obviously less appealing to me, when you can [use make with R](http://stat545.com/automation00_index.html).






















Make will do that automatically; i.e. rerun automatically if the source timestamp is newer than the output timestamp, if the "target" name is the name of the outputted file. So if you're putting a Stata job in the makefile, you could do something like the following

`helloworld.do`:
```stata
di "hello world"
```

```makefile
helloworld.log: helloworld.do
	stata-mp -b do helloworld.do
```

Then the first time you run `make`, it will do `stata-mp -b do helloworld.do`. The second time (if you haven't changed `helloworld.do`) it will print 
```
make: `helloworld.log' is up to date.
```
and won't run it again. 

Make does this specifically because the file `helloworld.do` is listed as a dependency after the `:` for the `helloworld.log` target.

Kyle
