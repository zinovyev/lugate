# Lugate
Lugate is a library for building JSON-RPC 2.0 Gateway API just inside of your NGINX configuration file

[![Build Status](https://travis-ci.org/zinovyev/lugate.svg?branch=master)](https://travis-ci.org/zinovyev/lugate)
[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/zinovyev/lugate/master/LICENSE)
[![GitHub version](https://badge.fury.io/gh/zinovyev%2Flugate.svg)](https://badge.fury.io/gh/zinovyev%2Flugate)

## About
When we talk about Microservices Architecture pattern there is a thing there called
[API Gateway](http://microservices.io/patterns/apigateway.html) that is the single entry point into the system.
Lugate is a binding over OpenResty's [ngx\_http\_lua\_module](https://github.com/openresty/lua-nginx-module) module.
The library provides features for request parsing, validating and routing. Lugate acts like a caching proxy over the
JSON-RPC 2.0 protocol and is meant to be used like an API Gateway for your system.

## Install
Lugate can be installed via the luarocks package manager. Just run:
```bash
luarocks install lugate
```

## Synopsis
```lua
    location / {
          # MIME type determined by default_type:
          default_type 'application/json';

          content_by_lua_block {
              -- Load lugate module
              local Lugate = require "lugate"

              -- Get new lugate instance
                  local lugate = Lugate:init({
                    json = require "rapidjson",
                    ngx = ngx,
                    cache = {'redis', '127.0.0.1', 6379},
                    routes = {
                      ['v1%.([^%.]+).*'] = '/v1', -- v1.math.subtract -> /v1.math
                      ['v2%.([^%.]+).*'] = '/v2', -- v2.math.addition -> /v2.math
                    }
                  })

              -- Send multi requst and get multi response
              lugate:run()

              -- Print responses out
              lugate:print_responses()
          }
    }
```

## Lugate proxy call
Lugate brings its own protocol (wrapper for JSON-RPC 2.0) to live.
It's used for transferring, routing and bringing cache control logic over
JSON-RPC 2.0 request.
The *params* member of the [request object](http://www.jsonrpc.org/specification#request_object)
is exteded with additional nested member fields:

* **route** - The routing destination
* **cache** - Cache life time
* **key** - Chache key
* **params** - The regular array of parameter values should be moved here for now

When the request is processed by the lugate proxy, the **route** and **cache** values are removed from the
*params* member and the nested **params** value is expanded to fill the whole parent *params* field.

```json
--> {"jsonrpc": "2.0", "method": "subtract", "params": {"cache":3600, "key": "foobar", "route": "v2.substract", "params": [42, 23]}, "id": 1}
<-- {"jsonrpc": "2.0", "result": -19, "id": 2}
```

### Route
### Params
### Ttl
### Key

## API reference
To get the API reference use this [autogenerated doc](http://zinovyev.github.io/lugate).

## Testing
Use `busted` framework for running tests.

## Running example
The example project uses vagrant provision.
So you need to have vagrant and virtualbox installed on your PC.
Launch project by running this command from the example directory:
```bash
vagrant up
```
Add this line to your local `/etc/hosts` file:
```
192.168.50.47 gateway.lugate.loc service01.lugate.loc service02.lugate.loc service03.lugate.loc
```
