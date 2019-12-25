# Introduction
The HSY Energy Management API is based on REST / Hypermedia and specified in [OpenAPI v2.0](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md). 

The API provides energy metrics for four **Flow** types:`harvest`, `store`, `yield`, `grid`. 

The API response consolidates all four energy flows in each request and reponse, based on a time period.

- Clients use the API to manage **Energy Assets** through flexible views of energy metrics and by scheduling use at preferred times.

- Vendor device integrations use the API to monitor and control **Energy Management Devices**.

Flow | Energy Asset | Energy Management Device 
--- | --- | ---
`harvest` | Renewables | PV Modules, PV Grid-interactive Inverter
`store` | Storage | Battery Management System (BMS), Energy Storage System (ESS)
`yield` | Appliances | Multicore Cable Sensors, Switchboard Clamp Sensors, Socket Sensors
`grid` | Mains Electricity | Smart Meters

The API and documentation is available through the *Sundaya Developer Portal* at https://developer.sundaya.com. 

# Overview

## Data element names

The following provides names of all data elements used in the API, in the context of each flow. 

- `store.in` and `store.out` indicate *charge* and *discharge* flows for batteries.

- `grid.out` and `grid.in` indicate mains use, and feed-in flows into the public grid.

- `harvest` indicates renewable generation and always implies `.out`. 

- `yield` indicates energy use and always implies `.in`. 

## Double-entry format 

Energy flows are based on a 'double entry' format (each flow has an opposite flow). 

These equal and opposite flows are summarised in the following table: 

Flow | From / To   
    --- |--- 
`harvest` |`store.in` `yield` `grid.in`
`store.out` | `yield`
`yield`  |  `harvest` `store.out` `grid.out`
    
To query specific assets, clients can filter requests by `category`, `subcategory`, and `product-type` (in the request *Body*).


## Versions
The API endpoint host is http://api.endpoints.sundaya.cloud.goog. 

All requests to the API endpoint receive the latest version of the API.     

Client applications may request a specific version through the `Accept` header.

    Accept: application/vnd.sundaya.v1+yaml

## Date-time Format
Date and time parameters must be expressed in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format and must conform to [RFC3359](https://tools.ietf.org/html/rfc3339) .

    http://api.endpoints.sundaya.cloud.goog/hsy/{periodID}/{end}

e.g. http://api.endpoints.sundaya.cloud.goog/hsy/week/20190210

The compressed version of ISO 8601 is required, without semi colons and with `T` as the time designator, as shown int he examples below.


## Timezones
Timezones are not assumed and must be explicitly specified where API parameters allow for a timestamp to be provided. 

The Timezone can be specified in UTC or local time as shown:

- __UTC__, expressed with a trailing `Z` 

    http://api.endpoints.sundaya.cloud.goog/hsy/week/YYYYMMDDThhmmssZ
    e.g. http://api.endpoints.sundaya.cloud.goog/hsy/week/201902091830Z == 18:30 UTC

- __Local__ time with offset 

    http://api.endpoints.sundaya.cloud.goog/hsy/week/YYYYMMDDThhmmss±hhmm
    e.g. http://api.endpoints.sundaya.cloud.goog/hsy/week/201902091500-0330 == 18:30 UTC
## Media types
Request `Body` parameters and all response objects are sent and received in JSON. 

Clients should use `Accept` and `Content-Type` headers to preserve backward compatibility (in case a new media type or hypermedia scheme is introduced and designated as default).

These media types are supported:

    application/json 
    application/vnd.collection+json

## Headers
This following example shows a sample HTTP request and response.
```
*** REQUEST ***	
GET /hsy/week/20190209/ HTTP/1.1	
Host: api.endpoints.sundaya.cloud.goog	
Accept: application/vnd.collection+json	
    
*** RESPONSE ***	
200 OK HTTP/1.1	
Content-Type: application/vnd.collection+json	
Content-Length: xxx	
    
{ "collection" : {...}, ... }
```

## Link-relation types
Link-relations in Response objects are based on [RFC8288](https://tools.ietf.org/html/rfc8288#page-6). 

The following registered types are referenced in the `rel` attribute of the links in an `application/vnd.collection+json` response. 
- **self**	- Identifies the link's context.

    In `collection.links` it identifies the collection (name = *week*)            

    - e.g. href=<a>http:/api.endpoints.sundaya.cloud.goog/hsy/week/20190210</a>

    In `collection.items.links` it identifies an item in the collection (name = *day*).
    - e.g. href=<a>http:/api.endpoints.sundaya.cloud.goog/hsy/day/20190204</a>

- **item** - Identifies child resources of the collection represented by the link's context. 

    In `collection.links` it targets the item series whiich make up the collection (name = *week.days*).
    - e.g. href=<a>http:/api.endpoints.sundaya.cloud.goog/hsy/day/20190204/20190210</a>

    In `collection.items.links` it targets subitems of the item in that context (name = *day.hours*).
    - e.g. href=<a>http:/api.endpoints.sundaya.cloud.goog/hsy/hour/201902050600/201902050500</a>

- **up** - Identifies the parent the collection or item represented by the link's context (name = *week.month*).
    
- **next** - Identifies the next sibling of the collection or item series represented by the link's context (name = *week.next*).

- **prev** - Identifies the previous siblings of the collection or item series represented by the link's context (name = *week.previous*).