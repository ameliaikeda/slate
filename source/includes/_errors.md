# Status Codes

The R/a/dio API uses the following error codes:


Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request sucks
401 | Unauthorized -- Your API key is wrong when trying to make authenticated requests
403 | Forbidden -- You aren't authenticated and are trying to access a protected resource
404 | Not Found -- The specified resource could not be found
405 | Method Not Allowed -- You tried to use a method we don't respond to 
406 | Not Acceptable -- You requested a format that isn't json
410 | Gone -- A resource was removed (rarely used)
418 | I'm a teapot -- You will know why you get this response
429 | Too Many Requests -- Stop spamming already (Time until unbanned: `X-Radio-Timeout` header)
500 | Internal Server Error -- We had a problem with our server. It'll maybe be fixed
503 | Service Unavailable -- We're temporarially offline for maintenance
