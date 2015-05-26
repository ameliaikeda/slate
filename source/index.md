---
title: R/a/dio API v2 Docs

language_tabs:
  - shell

toc_footers:
  - <a href='https://docs.r-a-d.io/v1'>Version 1 Documentation</a>

includes:
  - errors

search: true
---

# Introduction

R/a/dio's API is a plain REST API. We don't give out authentication keys unless you have an obvious need for it. Aside from that, requests are essentially limitless, and there is a global 1 second cache on all requests to avoid hitting a database more than needed when our highest resolution timestamps are in seconds.

<aside class="warning">
Unless explicitly stated, all endpoints are <code>GET</code> only unless authenticated.
</aside>

<aside class="notice">
  Should you need PUT/DELETE/PATCH capabilities and can;t make native requests, add the method to the <code>_method</code> query parameter.
</aside>

The base URL for the v2 API is `https://api.r-a-d.io/v2`, and GET examples are accessible in a browser.

# Overview

## Current Info

```shell
curl "https://api.r-a-d.io/v2/current"
```

> The above command returns JSON structured like this:

```json
{
  "now_playing": "tripflag - #comiket",
  "listeners": 125,
  "bot_stream": true,
  "start": 1432643119,
  "end": 1432643358,
  "current": 1432643325,
  "thread_url": "https://example.com/thread/1",
  "requests": true,
  "song": {
    "id": 1337,
    "artist": "tripflag",
    "title": "#comiket",
    "album": "Tesudo Musume - Collection",
    "tags": "gatta goto chuu chuu tetsu musuu"
  },
  "dj": {
    "id": "17",
    "name": "Hanyuu-sama",
    "image_url": "https://static.r-a-d.io/images/dj/17"
  }
}
```

This endpoint retrieves all kittens.

### HTTP Request

`GET https://api.r-a-d.io/v2/current`

### Response

Will always return `200 OK` unless we've really screwed up, in which case you'll see a `500 Internal Server Error`.

Key | Type | Description
--- | ---- | -----------
now_playing | string  | The currently playing song's metadata. This can be an empty string.
listeners   | integer | The number of listeners currently tuned into R/a/dio.
bot_stream  | boolean | If true, then our [AFK Streamer](https://github.com/R-a-dio/Hanyuu-sama) is playing. This does not mean you can request.
start       | long    | Unix timestamp for the time the current song started
end         | long    | Unix timestamp for the time the current song ended. `null` if unknown.
current     | long    | The current server time as a unix timestamp, used for syncing progress bars.
thread      | string  | A url to the current thread. Can be `string` or `null`
requests    | boolean | If `true`, requests are allowed to the `/request` endpoint.
song        | object  | A [Songs](#songs) object or `null`, describing the current playing song. If `null`, we don't have the song in our database, or a DJ is playing with different metadata. 
dj          | object  | A [DJs](#djs) object, or `null` if there is no currently connected DJ (stream is offline)
online      | boolean | `true` if the stream is currently online (A DJ is connected). Note this value can be unreliable as the stream has a fallback.

# Songs

## Get a specific song by ID


```shell
curl "https://api.r-a-d.io/v2/songs/1337"
```

> The above command returns JSON structured like this:

```json
{
  "artist": "tripflag",
  "title": "#comiket",
  "album": "Tesudo Musume - Collection",
  "tags": ["gatta", "goto", "chuu", "chuu", "tetsu", "musuu"],
  "requestable": true,
  "cooldown": 12341,
  "request_hash": "abcdef1234567890",
  "last_played": 1432643119,
  "last_requested": 1432643117,
  "plays": 9001
}
```

This endpoint retrieves a specific song by ID. See below for searching, which uses elasticsearch.


### HTTP Request

`GET https://api.r-a-d.io/v2/songs/:id`

### Response

Key | Type | Description
--- | ---- | -----------
artist         | string  | The song's artist metadata
title          | string  | The song's title metadata
album          | string  | The song's album metadata
tags           | array   | Tags for the song provided by R/a/dio admins, each tag is a string
requestable    | boolean | `true` if this song is requestable (no problems with the file, off cooldown)
cooldown       | long    | The number of seconds until a song becomes requestable (if negative, assume never). Requestable songs will show `0` as the cooldown
request_hash   | string  | A unique identifier used to request the song using the `/song/:id/request` endpoint
last_played    | long    | The unix timestamp of when this song was last played
last_requested | long    | The unix timestamp of when this song was last requested
plays          | integer | The number of times this song has been last_played

<aside class="notice">
  Be careful - cooldowns and other times use the value of the <code>current</code> key on the <code>GET /current</code> endpoint as their canonical time.
</aside>

## Search for a song


```shell
curl "https://api.r-a-d.io/v2/songs/search?q=comiket"
```

> The above command returns JSON structured like this:

```json
[
  {
    "artist": "tripflag",
    "title": "#comiket",
    "album": "Tesudo Musume - Collection",
    "tags": ["gatta", "goto", "chuu", "chuu", "tetsu", "musuu"],
    "requestable": true,
    "cooldown": 12341,
    "request_hash": "abcdef1234567890",
    "last_played": 1432643119,
    "last_requested": 1432643117,
    "plays": 9001
  },
  {
    "artist": "tripflag",
    "title": "#comiket - another one",
    "album": "Tesudo Musume - Collection",
    "tags": ["gatta", "goto", "chuu", "chuu", "tetsu", "musuu"],
    "requestable": true,
    "cooldown": 12341,
    "request_hash": "abcdef1234567890",
    "last_played": 1432643119,
    "last_requested": 1432643117,
    "plays": 9001
  }
]
```

This endpoint retrieves a specific song by ID. See below for searching, which uses elasticsearch.


### HTTP Request

`GET https://api.r-a-d.io/v2/songs/search`

### Response

See above. Returns an empty array (`[]`) when no results are found.

### URL Parameters

Parameter | Type | Description
--------- | ---- | -----------
q         | string | The primary search key (string); use this for search boxes
artist    | string | Fuzzy-matched against the `artist` key of a `Song`
title     | string | Fuzzy-matched against the `title` key of a `Song`
album     | string | Fuzzy-matched against the `album` key of a `Song`
tags      | string | Space-delimited string of tags to exact-match against

## Request a song

```shell
curl -XPATCH "https://api.r-a-d.io/v2/songs/1337/request" \
      -d "hash=abcdef1234567890123412512351235"
```

> The above command returns JSON structured like this:

```json
{
  "requested": false,
  "reason": {
    "human": "You need to wait longer before you can request",
    "robot": "cooldown"
  }
}
```

### HTTP Request

`PATCH https://api.r-a-d.io/v2/songs/1337/request`

<aside class="notice">
  If you can't make a PATCH request, append <code>?_method=PATCH</code> to the and use GET/POST
</aside>

### Response

Key | Type | Description
--- | ---- | -----------
requested    | boolean  | `true` if the song was successfully requested, `false` otherwise
reason.human | string  | A human-readable success/failure message
reason.robot | string  | An error code that should be easier to parse. See below.

### Error Codes

Code | Description
---- | -----------
cooldown | The requesting client has to wait longer before requesting again
timeout  | The request could not be completed because the request backend was unreachable
success  | The request completed successfully
banned   | The requesting client has been banned from making requests
invalid  | The hash provided was invalid, and the request was discarded


