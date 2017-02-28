# HTTP notes

## Verbs

HTTP requests use verbs to communicate the action they desire the server to perform. Here's a full list of verbs:

Verb | Description
--- | ---
HEAD | Asks for a response like a GET but without the body
GET | Retrieves a representation of a resource
POST | Submits data to be processed in the body of the request
PUT | Uploads a representation of a resource in the body of the request
DELETE | Deletes a specific resource
TRACE | Echoes back the received request
OPTIONS | Returns the HTTP methods the server supports
CONNECT | Converts the request to a TCP/IP tunnel (generally for SSL)
PATCH | Apply a partial modification of a resource

<a name="status"></a>
## Status codes

Full list [here](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes).

code | type
--- | ---
100 | 1xx: Informational (request received and continuing process)
200 | 2xx: Success (request successfully received, understood, and accepted)
300 | 3xx: Redirection (further action must be taken to complete request)
400 | 4xx: Client Error (request contains bad syntax and can't be completed)
500 | 5xx: Server Error (server couldn't complete request)