# Errors

The GoBoost API uses the following error codes:


Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request is invalid.
401 | Unauthorized -- Your auth credentials are invalid
403 | Forbidden -- You do not have access to requested resource. This is most likely due to the user role you use in your auth header.
404 | Not Found -- The route you requested could not be found.
500 | Internal Server Error -- We had a problem with our server. Try again later.
