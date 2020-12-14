# payloads_for_request_smuggling

Note worthy Points:

* According to RFC 7320, If a message is received with both a Transfer-Encoding and a Content-Length header field, the Transfer-Encoding overrides the Content-Length.

* The `Transfer-Encoding: chunked` waits to bytes in the post body until it find `0`. `0` means the end of the request.


## When CL & TE were using:

In this case: The front end server parses CL header, and forward the request to the backend, the backend supports the TE header, so it will ignore the CL and check the request where `0` ends the request and the rest of the data is for the second request, so when the second request goes to the server, it will respond as `GPOST` or `GGET` whatever the second request will use. Because the server will wait for the rest of the query, as the second request will start from 'GET' or 'POST'

```
Transfer-Encoding: chunked
Content-Type: application/x-www-form-urlencoded
Content-Length: 6

0

G
```

## When TE & CL were using:

In this case: The front end server will use the TE and find the `0` because this indicates the end of the chunked request, hence this means the front end server will send this whole request to the backend, now backend is not supporting the Transfer-Encoding header so it will check the Content-Length header and will find that there are only 4 Bytes of the content body which is `12\r\n` (`\r` will consider as 1 byte hence 4 bytes) and rest of it is a HTTP request as the syntax says with method `GPOST`, so it will send another request as `GPOST` method, which is a non standard method, resulting an error when we hit second request to the server.

```
Content-Length: 4
Transfer-Encoding: chunked

12
GPOST / HTTP/1.1

0

```

## When TE & TE were using:

In this case: The front end server prioritise the first header which is valid and send the whole request until `0` (basically line 45) the backend server might prioritize the second header which is a non standard header which the server will ignore and move to Content-Length header which will check 4 bytes and rest of the thing will happen as same as above.

```
Transfer-Encoding: chunked
Transfer-encoding: nothing
Content-Length: 4

12
GPOST / HTTP/1.1

0

```
