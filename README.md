% [pander: An R Pandoc writer](https://github.com/daroczig/pander)

**pander** is an [R](http://r-project.org) package containing [helpers](#helper-functions) to return [Pandoc](http://johnmacfarlane.net/pandoc/) markdown of user specified format or *automatically* of several type of [**R objects**](#generic-pander-method).

The package is also capable of exporting/converting complex Pandoc documents (reports) in two ways ATM:

  * users might write some reports in [brew](http://cran.r-project.org/web/packages/brew/index.html) syntax resulting in a pretty *Pandoc* document (where `cat`ed R object are transformed to table, list etc.) and also in a **bunch of formats** automatically.

    *Example*: this [`README.md`](https://github.com/daroczig/pander/blob/master/README.md) is cooked with `Pandoc.brew` based on [`inst/README.brew`](https://github.com/daroczig/pander/blob/master/inst/README.brew) and also exported to [HTML](http://daroczig.github.com/pander/). Details can be found [below](#brew-to-pandoc) or head directly to [examples](#examples).

<!-- endlist -->

 * or users might create a report in a live R session by adding some R objects and paragraphs to a `Pandoc` reference class object. Details can be found [below](#live-report-generation).

**How it is differ from Sweave, brew, knitr, R2HTML etc.?**

  * `pander` results in Pandoc's *markdown* which can be converted to almost any text document format (like: pdf, HTML, odt, docx, textile etc.). Conversion can be done automatically after calling `pander` reporting functions ([Pander.brew](#brew-to-pandoc) or [Pandoc](#live-report-generation)).
  * based on the above *no "traditional" R console output* is shown in the resulting document (nor in markdown, nor in exported docs) but **all R objects are transformed to tables, list etc**.
  * *graphs/plots* are recognized in blocks of R commands without any special setting or marks around code block and saved to disk in a `png` file linked in the resulting document. This means if you create a report (e.g. `brew` a text file) and export it to pdf/docx etc. all the plots/images would be there.
  * can use `brew`'s caching mechanism
  * `knitr` *support* is coming too, for details see my [TODO list](https://github.com/daroczig/pander/blob/master/TODO.md)

# Installation

Currently, this package is hosted only on [GitHub](https://github.com/daroczig/pander).

Until this gets on [CRAN](cran.r-project.org), you can install it via nifty function from `devtools` package:

```
library(devtools)
install_github('pander', 'daroczig')
```

Or download the [sources in a zip file](https://github.com/daroczig/pander/zipball/master) and build manually. If you're running R on Windows, you need to install [Rtools](http://cran.stat.ucla.edu/bin/windows/Rtools/).

## Depends

`pander` heavily builds on [Pandoc](http://johnmacfarlane.net/pandoc) which should be **pre-installed** before trying to convert your reports to [different formats](http://johnmacfarlane.net/pandoc/). Although main functions work without Pandoc, e.g. you can generate a markdown formatted report via [Pandoc.brew](#brew-to-pandoc) or the custom [reference class](#live-report-generation).

And as `pander` and `rapport` are quite Siamese twins, you would need an **up-to-date** version of [rapport](http://rapport-package.info) most likely installed from [Github](https://github.com/aL3xa/rapport).

# Helper functions

There are a bunch of helper functions in `pander` which return user specified inputs in Pandoc format. You could find these functions starting with `pandoc.`. For example `pandoc.strong` would return the passed characters with strong emphasis. E.g.:

```rout
> pandoc.strong('FOO')
**FOO**>
> pandoc.strong.return('FOO')
[1] "**FOO**"
```

As it can be seen here `pandoc` functions generally prints to console and do not return anything. If you want the opposite (get the Pandoc format as a string): call each function ending in `.return` - like the second call in the above example. For details please check documentation, e.g. `?pandoc.strong`.

Of course there are more complex functions too. Besides verbatim texts, (image) links or footnotes (among others) there are a helper e.g. for **lists**:

```rout
> l <- list("First list element", paste0(1:5, '. subelement'), "Second element", list('F', 'B', 'I', c('phone', 'pad', 'talics')))
> pandoc.list(l, 'roman')
```

Which returns:

```
<%
l <- list("First list element", paste0(1:5, '. subelement'), "Second element", list('F', 'B', 'I', c('phone', 'pad', 'talics')))
pandoc.list(l, 'roman')
%>
```

`pandoc` can return **tables** in [three formats](http://johnmacfarlane.net/pandoc/README.html#tables):

  * The default style is the `multiline` format as most features (e.g. multi-line cells and alignment) are available in there.

```rout
> m <- mtcars[1:2, 1:3]
> pandoc.table(m)
<%
pandoc.table(mtcars[1:2, 1:3])
%>
```

  * `simple` tables do not support line breaks in cells:

```rout
> pandoc.table(m, style = "simple")
<%
pandoc.table(mtcars[1:2, 1:3], style = "simple")
%>
```

  * `grid` format are really handy for [emacs](http://emacswiki.org/emacs/TableMode) users but do support line breaks inside cells, but do not tolerate cell alignment:

```rout
> pandoc.table(m, style = "grid")
<%
pandoc.table(mtcars[1:2, 1:3], style = "grid")
%>
```

Besides the `style` parameter there are several other ways to tweak the output like `decimal.mark` or `digits`. And of course it's really easy to add a **caption**:

```rout
> pandoc.table(m, style = "grid", caption = "Hello caption!")
<%
pandoc.table(mtcars[1:2, 1:3], style = "grid", caption = 'Hello caption!')
%>
```

`pandoc.table`˙can deal with the problem of really **wide tables**. Ever had an issue in LaTeX or MS Word when tried to print a correlation matrix of 40 variables? Not a problem any more as `pandoc.table` splits up the table if wider then 80 characters and handles caption too:

```rout
> pandoc.table(mtcars[1:2, ], style = "grid", caption = "Hello caption!")
<%
pandoc.table(mtcars[1:2, ], style = "grid", caption = 'Hello caption!')
%>
```

# Generic pander method

`pander` or `pandoc` (call as you wish) can deal with a bunch of R object types as being a pandocized `S3` method with a variety of classes.

Besides simple types (vectors, matrices, tables or data frames) lists might be interesting for you:

```rout
> pander(list(a=1, b=2, c=table(mtcars$am), x=list(myname=1,2), 56))
<%pander(list(a=1, b=2, c=table(mtcars$am), x=list(myname=1,2), 56))%>
```

A nested list can be seen above with a table and all (optional) list names inside. As a matter of fact `pander.list` is the default method of `pander` too, see:

```rout
> x <- chisq.test(table(mtcars$am, mtcars$gear))
> class(x) <- "I've never heard of!"
> pander(x)
<%
x <- chisq.test(table(mtcars$am, mtcars$gear))
class(x) <- 'I\'ve never heard of!'
pander(x)
%>
```

So `pander` showed a not known class in an (almost) user-friendly way. And we got some warnings too styled with [Pandoc **footnote**](http://johnmacfarlane.net/pandoc/README.html#footnotes)! If that document is exported to e.g. `HTML` or `pdf`, then the error/warning message could be found on the bottom of the page with a link. *Note*: there were two warnings in the above call - both captured and returned! Well, this is the feature of `Pandoc.brew`, see [below](#brew-to-pandoc).

The output of different **statistical methods** are tried to be prettyfied. Some examples:

```rout
> pander(ks.test(runif(50), runif(50)))
<%pander(ks.test(runif(50), runif(50)))%>

> pander(chisq.test(table(mtcars$am, mtcars$gear)))
<%pander(chisq.test(table(mtcars$am, mtcars$gear)))%>

> pander(t.test(extra ~ group, data = sleep))
<%pander(t.test(extra ~ group, data = sleep))%>

> ## Dobson (1990) Page 93: Randomized Controlled Trial (examples from: ?glm)
> counts <- c(18,17,15,20,10,20,25,13,12)
> outcome <- gl(3,1,9)
> treatment <- gl(3,3)
> m <- glm(counts ~ outcome + treatment, family=poisson())
> pander(m)
<%
counts <- c(18,17,15,20,10,20,25,13,12)
outcome <- gl(3,1,9)
treatment <- gl(3,3)
m <- glm(counts ~ outcome + treatment, family=poisson())
pander(m)
%>

> pander(anova(m))
<%pander(anova(m))%>

> pander(aov(m))
<%pander(aov(m))%>

> pander(prcomp(USArrests))
<%pander(prcomp(USArrests))%>

> pander(density(mtcars$hp))
<%pander(density(mtcars$hp))%>

```

# Brew to Pandoc

Everyone knows and uses [brew](http://cran.r-project.org/web/packages/brew/index.html) but if you would need some smack, the following links really worth visiting:

  * TODO

**In short**: a `brew` document is a simple text file with some special tags. `Pandoc.brew` uses only two of them (sorry, no brew templates here):

  * `<\% ... \%>` (without the backslash) stand for running R calls
  * `<\%= ... \%>` (without the backslash) does pretty the same but applies `pander` to the returning R object (instead of `cat` like the original `brew` function does). So putting there any R object would return is a nice Pandoc markdown format.

This latter tries to be smart in some ways:

  * a block (R commands between the tags) could return a value in the middle of the block and do something else without any output in the rest (but only one returned value per block!)
  * plots and images are grabbed in the document, rendered to a `png` file and `pander` method would result in a Pandoc markdown formatted image link (so the image would be shown/included in the exported document).
  * all warnings/messages and errors are recorded in the blocks and returned in the document as a footnote

This document was generated by `Pandoc.brew` based on [`inst/README.brew`](https://github.com/daroczig/pander/blob/master/inst/README.brew) so the above examples were generated automatically - which could be handy while writing some nifty statistical reports :)

```
Pandoc.brew(system.file('README.brew', package='pander'))
```

`Pandoc.brew` could cook a file (default) or work with a character vector provided in the `text` argument. The output is set to `stdout` by default, it could be tweaked to write result to a text file and run Pandoc on that to create a `HTML`, `odt`, `docx` or other document.

To export a brewed file to other then Pandoc's markdown, please use the `convert` parameter! For example (please disregard the backslash in front of the percent sign):

```r
text <- paste('# Header', '', '<\%=as.list(runif(10))\%>', '<\%=mtcars[1:3, ]\%>', '<\%=plot(1:10)\%>', sep = '\n')
Pandoc.brew(text = text, output = tempfile(), convert = 'html')
Pandoc.brew(text = text, output = tempfile(), convert = 'pdf')
```

Of course a text file could work as input (by default) the above example uses `text` parameter as a reproducible example. For example brewing this README with all R chunks run to html, pleae run:

```r
Pandoc.brew(system.file('README.brew', package='pander'), output = tempfile(), convert = 'html')
Pandoc.brew(system.file('README.brew', package='pander'), output = tempfile(), convert = 'pdf')
```

## Examples

The package comes bundled with some examples for `Pandoc.brew` to let users check out its features pretty fast. These are:

  * [minimal.brew](https://github.com/daroczig/pander/blob/master/inst/examples/minimal.brew)
  * [short-code-long-report.brew](https://github.com/daroczig/pander/blob/master/inst/examples/short-code-long-report.brew)

To `brew` these examples on your machine try to run e.g.:

```r
Pandoc.brew(system.file('examples/minimal.brew', package='pander'), output = tempfile(), convert = 'html')
```

For easy access I have uploaded some exported documents of the above examples:

  * minimal.brew: [html](minimal.html) [pdf](minimal.pdf) [odt](minimal.odt) [docx](minimal.docx)
  * short-code-long-report.brew: [html](short-code-long-report.html) [html](short-code-long-report.html) [html](short-code-long-report.html) [html](short-code-long-report.docx)

Please check out `pdf`, `docx`, `odt` and other formats (changing the above `convert` option) on your machine too and do not forget to [give some feedback](https://github.com/daroczig/pander/issues)!

# Live report generation

`pander` package has a special reference class called `Pandoc` which could collect some blocks in a live R session and export the whole document to Pandoc/pdf/HTML etc.

Without any serious comments, please check out the below (self-commenting) example:

```r
## Initialize a new Pandoc object
myReport <- Pandoc$new()

## Add author, title and date of document
myReport$author <- 'Gergely Daróczi'
myReport$title  <- 'Demo'

## Or it could be done while initializing
myReport <- Pandoc$new('Gergely Daróczi', 'Demo')

## Add some free text
myReport$add.paragraph('Hello there, this is a really short tutorial!')

## Add maybe a header for later stuff
myReport$add.paragraph('# Showing some raw R objects below')

## Adding a short matrix
myReport$add(matrix(5,5,5))

## Or a table with even
myReport$add.paragraph('Hello table:')
myReport$add(table(mtcars$am, mtcars$gear))

## Or a "large" dataframe which barely fits on a page
myReport$add(mtcars)

## And a simple linear model with Anova tables
ml <- with(lm(mpg ~ hp + wt), data = mtcars)
myReport$add(ml)
myReport$add(anova(ml))
myReport$add(aov(ml))

## And do some principal component analysis at last
myReport$add(prcomp(USArrests))

## Sorry, I did not show how Pandoc deals with plots:
myReport$add(plot(1:10))

## Want to see the report? Just print it:
myReport

## Exporting to pdf (default)
myReport$export()

## Or to docx in tempdir():
myReport$format <- 'docx'
myReport$export(tempfile())

## You do not want to see the generated report after generation?
myReport$export(open = FALSE)
```

-------
This report was generated with [R](http://www.r-project.org/) (2.15.0) and [pander](https://github.com/daroczig/pander) (0.1) in 0.071 sec on x86_64-unknown-linux-gnu platform.