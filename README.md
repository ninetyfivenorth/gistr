gistr
=======



[![Build Status](https://api.travis-ci.org/ropensci/gistr.png)](https://travis-ci.org/ropensci/gistr)
[![Build status](https://ci.appveyor.com/api/projects/status/4jmuxbbv8qg4139t/branch/master?svg=true)](https://ci.appveyor.com/project/sckott/gistr/branch/master)
[![Coverage Status](https://coveralls.io/repos/ropensci/gistr/badge.svg)](https://coveralls.io/r/ropensci/gistr)

`gistr` is a light interface to GitHub's gists for R.

## See also:

* [rgithub](https://github.com/cscheid/rgithub) an R client for the Github API by Carlos Scheidegger
* [git2r](https://github.com/ropensci/git2r) an R client for the libgit2 C library by Stefan Widgren

## Quick start

### Install

Stable version from CRAN


```r
install.packages("gistr")
```

Or dev version from GitHub. 


```r
devtools::install_github("ropensci/gistr")
```


```r
library("gistr")
```

### Authentication

There are two ways to authorise gistr to work with your GitHub account:

* Generate a personal access token (PAT) at [https://help.github.com/articles/creating-an-access-token-for-command-line-use](https://help.github.com/articles/creating-an-access-token-for-command-line-use) and record it in the `GITHUB_PAT` environment variable.
  - To test out this approach, execute this in R: `Sys.setenv(GITHUB_PAT = "blahblahblah")`, where "blahblahblah" is the PAT you got from GitHub. Then take `gistr` out for a test drive.
  - If that works, you will probably want to define the GITHUB_PAT environment variable in a file such as `~/.bash_profile` or `~/.Renviron`.
* Interactively login into your GitHub account and authorise with OAuth.

Using the PAT is recommended.

Using the `gist_auth()` function you can authenticate separately first, or if you're not authenticated, this function will run internally with each function call. If you have a PAT, that will be used, if not, OAuth will be used.


```r
gist_auth()
```

### Workflow

In `gistr` you can use pipes, introduced perhaps first in R in the package `magrittr`, to pass outputs from one function to another. If you have used `dplyr` with pipes you can see the difference, and perhaps the utility, of this workflow over the traditional workflow in R. You can use a non-piping or a piping workflow with `gistr`. Examples below use a mix of both workflows. Here is an example of a piping workflow (with some explanation):


```r
file <- system.file("examples", "alm.md", package = "gistr")
gists(what = "minepublic")[[1]] %>% # List my public gists, and index to get just the 1st one
  add_files(file) %>% # Add a new file to that gist
  update() # update sends a PATCH command to the Gists API to add the file to your gist online
```

And a non-piping workflow that does the same exact thing:


```r
file <- system.file("examples", "alm.md", package = "gistr")
g <- gists(what = "minepublic")[[1]]
g <- add_files(g, file)
update(g)
```

Or you could string them all together in one line (but it's rather difficult to follow what's going on because you have to read from the inside out)


```r
file <- system.file("examples", "alm.md", package = "gistr")
update(add_files(gists(what = "minepublic")[[1]], file))
```

### Rate limit information


```r
rate_limit()
#> Rate limit: 5000
#> Remaining:  4584
#> Resets in:  25 minutes
```

### List gists

Limiting to a few results here to keep it brief


```r
gists(per_page = 2)
#> [[1]]
#> <gist>5f2a44cf020285cd888a
#>   URL: https://gist.github.com/5f2a44cf020285cd888a
#>   Description: Medusa collective data
#>   Public: TRUE
#>   Created/Edited: 2015-03-19T00:21:28Z / 2015-03-19T00:21:28Z
#>   Files: results.js
#> 
#> [[2]]
#> <gist>ec9987ad245bbc668c72
#>   URL: https://gist.github.com/ec9987ad245bbc668c72
#>   Description: 
#>   Public: TRUE
#>   Created/Edited: 2015-03-19T00:21:13Z / 2015-03-19T00:21:13Z
#>   Files: plots.md
```

Since a certain date/time


```r
gists(since='2014-05-26T00:00:00Z', per_page = 2)
#> [[1]]
#> <gist>5f2a44cf020285cd888a
#>   URL: https://gist.github.com/5f2a44cf020285cd888a
#>   Description: Medusa collective data
#>   Public: TRUE
#>   Created/Edited: 2015-03-19T00:21:28Z / 2015-03-19T00:21:28Z
#>   Files: results.js
#> 
#> [[2]]
#> <gist>ec9987ad245bbc668c72
#>   URL: https://gist.github.com/ec9987ad245bbc668c72
#>   Description: 
#>   Public: TRUE
#>   Created/Edited: 2015-03-19T00:21:13Z / 2015-03-19T00:21:13Z
#>   Files: plots.md
```

Request different types of gists, one of public, minepublic, mineall, or starred.


```r
gists('minepublic', per_page = 2)
#> [[1]]
#> <gist>ec9987ad245bbc668c72
#>   URL: https://gist.github.com/ec9987ad245bbc668c72
#>   Description: 
#>   Public: TRUE
#>   Created/Edited: 2015-03-19T00:21:13Z / 2015-03-19T00:21:13Z
#>   Files: plots.md
#> 
#> [[2]]
#> <gist>1a6e7f7d6ddb739fce0b
#>   URL: https://gist.github.com/1a6e7f7d6ddb739fce0b
#>   Description: 
#>   Public: TRUE
#>   Created/Edited: 2015-03-19T00:20:48Z / 2015-03-19T00:20:48Z
#>   Files: plots_imgur.md
```


### List a single commit


```r
gist(id = 'f1403260eb92f5dfa7e1')
#> <gist>f1403260eb92f5dfa7e1
#>   URL: https://gist.github.com/f1403260eb92f5dfa7e1
#>   Description: Querying bitly from R 
#>   Public: TRUE
#>   Created/Edited: 2014-10-15T20:40:12Z / 2014-10-15T21:54:29Z
#>   Files: bitly_r.md
```

### Create gist

You can pass in files


```r
file <- system.file("examples", "stuff.md", package = "gistr")
gist_create(file, description='a new cool gist', browse = FALSE)
#> <gist>eff66901203ed68cef01
#>   URL: https://gist.github.com/eff66901203ed68cef01
#>   Description: a new cool gist
#>   Public: TRUE
#>   Created/Edited: 2015-03-19T00:21:34Z / 2015-03-19T00:21:34Z
#>   Files: stuff.md
```

Or, wrap `gist_create()` around some code in your R session/IDE, with just the function name, and a `{'` at the start and a `}'` at the end.


```r
gist_create(code={'
x <- letters
numbers <- runif(8)
numbers

[1] 0.3229318 0.5933054 0.7778408 0.3898947 0.1309717 0.7501378 0.3206379 0.3379005
'})
```


```r
gist_create(code={'
x <- letters
numbers <- runif(8)
numbers

[1] 0.3229318 0.5933054 0.7778408 0.3898947 0.1309717 0.7501378 0.3206379 0.3379005
'}, browse=FALSE)
#> <gist>413c09eb9791b0598070
#>   URL: https://gist.github.com/413c09eb9791b0598070
#>   Description: 
#>   Public: TRUE
#>   Created/Edited: 2015-03-19T00:21:35Z / 2015-03-19T00:21:35Z
#>   Files: code.R
```

#### knit and create

You can also knit an input file before posting as a gist:


```r
file <- system.file("examples", "stuff.Rmd", package = "gistr")
gist_create(file, description='a new cool gist', knit=TRUE)
#> <gist>4162b9c53479fbc298db
#>   URL: https://gist.github.com/4162b9c53479fbc298db
#>   Description: a new cool gist
#>   Public: TRUE
#>   Created/Edited: 2014-10-27T16:07:31Z / 2014-10-27T16:07:31Z
#>   Files: stuff.md
```

Or code blocks before (note that code blocks without knitr block demarcations will result in unexecuted code):


```r
gist_create(code={'
x <- letters
(numbers <- runif(8))
'}, knit=TRUE)
#> <gist>ec45c396dee4aa492139
#>   URL: https://gist.github.com/ec45c396dee4aa492139
#>   Description:
#>   Public: TRUE
#>   Created/Edited: 2014-10-27T16:09:09Z / 2014-10-27T16:09:09Z
#>   Files: file81720d1ceff.md
```

### knit code from file path, code block, or gist file

knit a local file


```r
file <- system.file("examples", "stuff.Rmd", package = "gistr")
run(file, knitopts = list(quiet=TRUE)) %>% gist_create(browse = FALSE)
#> <gist>68941c9a914ef3a27bfa
#>   URL: https://gist.github.com/68941c9a914ef3a27bfa
#>   Description: 
#>   Public: TRUE
#>   Created/Edited: 2015-03-19T00:21:35Z / 2015-03-19T00:21:35Z
#>   Files: stuff.md
```



knit a code block (knitr code block notation missing, do add that in) (result not shown)


```r
run({'
x <- letters
(numbers <- runif(8))
'}) %>% gist_create
```

knit a file from a gist, has to get file first (result not shown)


```r
gists('minepublic')[[1]] %>% run() %>% update()
```

### working with images

The GitHub API doesn't let you upload binary files (e.g., images) via their HTTP API, which we use
in `gistr`. There is a workaround.

If you are using `.Rmd` or `.Rnw` files, you can set `imgur_inject = TRUE` in `gistr_create()` so 
that imgur knit options are injected at the top of your file so that images will be uploaded to imgur. 
Alternatively, you can do this yourself, setting knit options to use imgur.

A file already using imgur


```r
file <- system.file("examples", "plots_imgur.Rmd", package = "gistr")
gist_create(file, knit=TRUE)
#> <gist>1a6e7f7d6ddb739fce0b
#>   URL: https://gist.github.com/1a6e7f7d6ddb739fce0b
#>   Description: 
#>   Public: TRUE
#>   Created/Edited: 2015-03-19T00:20:48Z / 2015-03-19T00:20:48Z
#>   Files: plots_imgur.md
```

A file _NOT_ already using imgur


```r
file <- system.file("examples", "plots.Rmd", package = "gistr")
gist_create(file, knit=TRUE, imgur_inject = TRUE)
#> <gist>ec9987ad245bbc668c72
#>   URL: https://gist.github.com/ec9987ad245bbc668c72
#>   Description: 
#>   Public: TRUE
#>   Created/Edited: 2015-03-19T00:21:13Z / 2015-03-19T00:21:13Z
#>   Files: plots.md
```

### List commits on a gist


```r
gists()[[1]] %>% commits()
#> [[1]]
#> <commit>
#>   Version: 24ae478d01413e190a32be5159ffdc97edac5630
#>   User: sckott
#>   Commited: 2015-03-19T00:21:35Z
#>   Commits [total, additions, deletions]: [5,5,0]
```

### Star a gist

Star


```r
gist('7ddb9810fc99c84c65ec') %>% star()
#> <gist>7ddb9810fc99c84c65ec
#>   URL: https://gist.github.com/7ddb9810fc99c84c65ec
#>   Description: 
#>   Public: TRUE
#>   Created/Edited: 2014-06-27T17:50:37Z / 2014-06-27T17:50:37Z
#>   Files: code.R, manifest.yml, rrt_manifest.yml
```

Unstar


```r
gist('7ddb9810fc99c84c65ec') %>% unstar()
#> <gist>7ddb9810fc99c84c65ec
#>   URL: https://gist.github.com/7ddb9810fc99c84c65ec
#>   Description: 
#>   Public: TRUE
#>   Created/Edited: 2014-06-27T17:50:37Z / 2014-06-27T17:50:37Z
#>   Files: code.R, manifest.yml, rrt_manifest.yml
```

### Edit a gist

Add files


```r
file <- system.file("examples", "alm.md", package = "gistr")
gists(what = "minepublic")[[1]] %>%
  add_files(file) %>%
  update()
#> <gist>413c09eb9791b0598070
#>   URL: https://gist.github.com/413c09eb9791b0598070
#>   Description: 
#>   Public: TRUE
#>   Created/Edited: 2015-03-19T00:21:35Z / 2015-03-19T00:21:37Z
#>   Files: alm.md, code.R
```

Delete files


```r
file <- system.file("examples", "alm.md", package = "gistr")
gists(what = "minepublic")[[1]] %>%
  delete_files(file) %>%
  update()
#> <gist>413c09eb9791b0598070
#>   URL: https://gist.github.com/413c09eb9791b0598070
#>   Description: 
#>   Public: TRUE
#>   Created/Edited: 2015-03-19T00:21:35Z / 2015-03-19T00:21:38Z
#>   Files: code.R
```

### Open a gist in your default browser


```r
gists()[[1]] %>% browse()
```

> Opens the gist in your default browser

### Get embed script


```r
gists()[[1]] %>% embed()
#> [1] "<script src=\"https://gist.github.com/sckott/413c09eb9791b0598070.js\"></script>"
```

### List forks

Returns a list of `gist` objects, just like `gists()`


```r
gist(id='1642874') %>% forks(per_page=2)
#> [[1]]
#> <gist>1642989
#>   URL: https://gist.github.com/1642989
#>   Description: Spline Transition
#>   Public: TRUE
#>   Created/Edited: 2012-01-19T21:45:20Z / 2015-02-20T13:50:25Z
#>   Files: 
#> 
#> [[2]]
#> <gist>1643051
#>   URL: https://gist.github.com/1643051
#>   Description: Line Transition (Broken)
#>   Public: TRUE
#>   Created/Edited: 2012-01-19T21:51:30Z / 2015-01-15T16:50:51Z
#>   Files:
```

### Fork a gist

Returns a `gist` object


```r
g <- gists()
(forked <- g[[ sample(seq_along(g), 1) ]] %>% fork())
#> <gist>515b565e220d07d30025
#>   URL: https://gist.github.com/515b565e220d07d30025
#>   Description: Medusa collective data
#>   Public: TRUE
#>   Created/Edited: 2015-03-19T00:21:39Z / 2015-03-19T00:21:39Z
#>   Files: results.js
```



## Example use case

_Working with the Mapzen Pelias geocoding API_

The API is described at https://github.com/pelias/pelias, and is still in alpha they say. The steps: get data, make a gist. The data is returned from Mapzen as geojson, so all we have to do is literally push it up to GitHub gists and we're done b/c GitHub renders the map.


```r
library('httr')
base <- "http://pelias.mapzen.com/search"
res <- GET(base, query = list(input = 'coffee shop', lat = 45.5, lon = -122.6))
json <- content(res, as = "text")
gist_create(code = json, filename = "pelias_test.geojson")
#> <gist>017214637bcfeb198070
#>   URL: https://gist.github.com/017214637bcfeb198070
#>   Description:
#>   Public: TRUE
#>   Created/Edited: 2014-10-28T14:42:36Z / 2014-10-28T14:42:36Z
#>   Files: pelias_test.geojson
```

And here's that [gist](https://gist.github.com/sckott/017214637bcfeb198070)

![pelias img](inst/img/gistr_ss.png)

## Meta

* Please [report any issues or bugs](https://github.com/ropensci/gistr/issues).
* License: MIT
* Get citation information for `gistr` in R doing `citation(package = 'gistr')`

[![rofooter](http://ropensci.org/public_images/github_footer.png)](http://ropensci.org)
