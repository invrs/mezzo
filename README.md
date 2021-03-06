# Mezzo

[![Build Status](https://travis-ci.org/invrs/mezzo.svg?branch=master)](https://travis-ci.org/invrs/mezzo)

Abstract anything into middleware.

## The future

What if you could use middleware for things other than an HTTP request?

Things like:

* configuring a web server
* building your client assets
* extending component functionality
* DRYing up shared project functionality

### Example

From the **[paradiso](https://github.com/invrs/paradiso)** web framework:

```coffee
# Start the web server
#
routes  = require "./routes"
server  = require "paradiso-server"
express = require "paradiso-server-express"

server routes, express
  port:   9000
  static: "public"
```

### Goals

* Abstract library-specific code into small, reusable, and testable middleware.
* Maintain a similar interface for libraries that do the same thing.
* (Change out libraries without changing app code.)
* Piece together and configure middleware easily.

### Configuring middleware

If you only pass options to an adapter, it **does not** run the middleware chain.

```coffee
build option: true
```

However, it does save the options for a later execution:

```coffee
build option: true
build()  # @options.option is still true
```

### Run the workflow

If you pass an adapter or nothing at all, the middleware chain **does** run:

```coffee
build()
build browserify, coffeeify
build browserify, coffeeify, option: true
```

## Write middleware

Skeleton implementation of a mezzo middleware:

```coffee
mezzo = require "mezzo"

module.exports = mezzo class
  constructor: ({
    @adapters  # array of adapters in order of execution
    @options   # any options passed as a parameter (merged)
    @index     # index of this adapter in `@adapters`
  }) ->

  run: ({ env, next }) -> next env
```

### Put it all together

```coffee
mezzo = require "mezzo"

# Build adapters
#
a = mezzo class
  constructor: ({ @options }) ->

  run: ({ env, next }) ->
    console.log "a @options", @options
  	env.a_run = true
  	next()

b = mezzo class
  run: ({ env, next }) ->
  	console.log "b"
  	next b_run: true

c = mezzo class
  run: ({ env, next }) ->
  	env.c_run = true
  	console.log "c env", env
  	next()

# Set options
#
a opt: true

# Execute middleware chain
#
a b, c, opt2: true

# Output:
#
#   a @options { opt: true, opt2: true }
#   b
#   c env { a_run: true, b_run: true, c_run: true }
```
