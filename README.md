
# Introduction
The HSE Energy Management API is REST based on [OpenAPI v2.0](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md) and specified in `yaml` .

The API produces Hypermedia content for client applications to navigate and monitor Energy Assets, based on four energy flows.

  `harvest` - energy *from* renewable source
  `store` - energy *from* batteries
  `enjoy` - energy *to* appliances
  `grid` - energy *from* a public grid

API documentation is available through the *Sundaya Developer Portal* at http://developer.sundaya.com. 

## Versions
The API endpoint host is http://api.sundaya.com. 

All requests to the API endpoint receive the latest version of the API.     

Client applications may request a specific version through the `Accept` header.

    Accept: application/vnd.sundaya.v1+yaml

# Overview
The main request path (`hse/`) returns consolidated data for all four energy flows (*Harvest*, *Store*, *Enjoy*, *Grid*). 

Optionally clients can filter queries (in the request *Body*) by `category`, `subcategory`, and `product-type`, to restrict data to specific energy assets. 

## Date-time Format
Date and time parameters must be expressed in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format and must conform to [RFC3359](https://tools.ietf.org/html/rfc3339) .

    http://api.sundaya.com/hse/period/{periodID}/{finishes}

e.g. http://api.sundaya.com/hse/period/week/20190210

The compressed version of ISO 8601 is required, without semi colons and with `T` as the time designator, as shown int he examples below.


## Timezones
Timezones are not assumed and must be explicitly specified where API parameters allow for a timestamp to be provided. 

The Timezone can be specified in UTC or local time as shown:

- __UTC__, expressed with a trailing `Z` 

    http://api.sundaya.com/hse/period/week/YYYYMMDDThhmmssZ
    e.g. http://api.sundaya.com/hse/period/week/201902091830Z == 18:30 UTC

- __Local__ time with offset 

    http://api.sundaya.com/hse/period/week/YYYYMMDDThhmmss±hhmm
    e.g. http://api.sundaya.com/hse/period/week/201902091500-0330 == 18:30 UTC
## Media Types
Request `Body` parameters and all response objects are sent and received in JSON. 

Clients should use `Accept` and `Content-Type` headers to preserve backward compatibility (in case a new media type or hypermedia scheme is introduced and designated as default).

These media types are supported:

    application/json 
    application/vnd.collection+json

## Headers
This following example shows a sample HTTP request and response.
```
*** REQUEST ***	
GET /hse/period/week/20190209/ HTTP/1.1	
Host: api.sundaya.com	
Accept: application/vnd.collection+json	

*** RESPONSE ***	
200 OK HTTP/1.1	
Content-Type: application/vnd.collection+json	
Content-Length: xxx	

{ "collection" : {...}, ... }
```

## Link-relation Types
Link-relations in Response objects are based on [RFC8288](https://tools.ietf.org/html/rfc8288#page-6). 

The following registered types are referenced in the `rel` attribute of the links in an `application/vnd.collection+json` response. 
- **self**	- Identifies the link's context.
  - In `collection.links` it identifies the collection (name = *week*)            
    - e.g. href=<a>http:/api.sundaya.com/hse/week/20190210</a>

  - In `collection.items.links` it identifies an item in the collection (name = *day*).
    - e.g. href=<a>http:/api.sundaya.com/hse/day/20190204</a>

- **item** - Identifies child resources of the collection represented by the link's context. 
  - In `collection.links` it targets the item series whiich make up the collection (name = *week.days*).
    - e.g. href=<a>http:/api.sundaya.com/hse/day/20190204/20190210</a>

  - In `collection.items.links` it targets subitems of the item in that context (name = *day.hours*).
    - e.g. href=<a>http:/api.sundaya.com/hse/hour/201902050600/201902050500</a>

- **up** - Identifies the parent the collection or item represented by the link's context (name = *week.month*).

- **next** - Identifies the next sibling of the collection or item series represented by the link's context (name = *week.next*).

- **prev** - Identifies the previous siblings of the collection or item series represented by the link's context (name = *week.previous*).