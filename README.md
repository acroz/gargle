
<!-- README.md is generated from README.Rmd. Please edit that file -->

# gargle

<!-- badges: start -->

[![CRAN
status](https://www.r-pkg.org/badges/version/gargle)](https://cran.r-project.org/package=gargle)
[![Build
Status](https://travis-ci.org/r-lib/gargle.svg?branch=master)](https://travis-ci.org/r-lib/gargle)
[![AppVeyor Build
Status](https://ci.appveyor.com/api/projects/status/github/r-lib/gargle?branch=master&svg=true)](https://ci.appveyor.com/project/r-lib/gargle)
[![codecov.io](https://codecov.io/github/r-lib/gargle/coverage.svg?branch=master)](https://codecov.io/github/r-lib/gargle?branch=master)
[![lifecycle](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
<!-- badges: end -->

The goal of gargle is to take some of the agonizing pain out of working
with Google APIs. This includes functions and classes for handling
common credential types and for preparing, executing, and processing
HTTP requests.

The target user of gargle is an *R package author* who is wrapping one
of the ~250 Google APIs listed in the [APIs
Explorer](https://developers.google.com/apis-explorer). gargle aims to
play roughly the same role as [Google’s official client
libraries](https://developers.google.com/api-client-library/), but for
R. gargle may also be useful to useRs making direct calls to Google
APIs, who are prepared to navigate the details of low-level API access.

gargle’s functionality falls into two main domains:

  - **Auth.** The `token_fetch()` function calls a series of concrete
    credential-fetching functions to obtain a valid access token (or it
    quietly dies trying).
      - This covers explicit service accounts, application default
        credentials, Google Compute Engine, and the standard OAuth2
        browser flow.
      - gargle offers the `Gargle2.0` class, which extends
        `httr::Token2.0`. It is the default class for user OAuth 2.0
        credentials. There are two main differences from
        `httr::Token2.0`: greater emphasis on the user’s email
        (e.g. Google identity) and default token caching is at the user
        level.
  - **Requests and responses**. A family of functions helps to prepare
    HTTP requests, (possibly with reference to an API spec derived from
    a Discovery Document), make requests, and process the response.

See the [articles](https://gargle.r-lib.org/articles/) for holistic
advice on how to use gargle.

## Installation

You can install the released version of gargle from
[CRAN](https://CRAN.R-project.org) with:

``` r
install.packages("gargle")
```

And the development version from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("r-lib/gargle")
```

## Basic usage

gargle is a low-level package and does not do anything visibly exciting
on its own. But here’s a bit of usage in an interactive scenario where a
user confirms they want to use a specific Google identity and loads an
OAuth2 token.

``` r
library(gargle)

token <- token_fetch()
#> The gargle package is requesting access to your Google account. Select a
#> pre-authorised account or enter '0' to obtain a new token. Press
#> Esc/Ctrl + C to abort.

#> 1: janedoe_personal@gmail.com
#> 2: janedoe@example.com

#> Selection: 1

token
#> <Token (via gargle)>
#>   <oauth_endpoint> google
#>              <app> gargle-demo
#>            <email> janedoe_personal@gmail.com
#>           <scopes> ...userinfo.email
#>      <credentials> access_token, expires_in, refresh_token, scope, ...
```

Here’s an example of using request and response helpers to make a
one-off request to the [Places
API](https://developers.google.com/places/web-service/intro). We’ll find
the restaurants in a certain part of Vancouver, BC.

``` r
library(gargle)

req <- request_build(
  method = "GET",
  path = "maps/api/place/nearbysearch/json",
  params = list(
    location = "49.268682,-123.167117",
    radius = 100,
    type = "restaurant"
  ),
  key = gargle_api_key(),
  base_url = "https://maps.googleapis.com"
)
resp <- request_make(req)
out <- response_process(resp)
vapply(out$results, function(x) x$name, character(1))
#> [1] "The Naam"                    "The Oakwood Canadian Bistro"
#> [3] "Healthy Noodle House"        "Tapatio Mexican Cafe & Bar" 
#> [5] "FreshFast"
```

Please note that the ‘gargle’ project is released with a [Contributor
Code of Conduct](https://gargle.r-lib.org/CODE_OF_CONDUCT.html). By
contributing to this project, you agree to abide by its terms.
