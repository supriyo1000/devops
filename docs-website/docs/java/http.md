# How HTTP Actually Works

Imagine your Java application wants user data from a server.

```
Your Java App
      |
      | HTTP Request
      v
Server
      |
      | HTTP Response
      v
Your Java App
```

### Real Example

Suppose you open:
```
https://jsonplaceholder.typicode.com/users/1
```

Your browser sends:
```
GET /users/1 HTTP/1.1
Host: jsonplaceholder.typicode.com
```
The server responds:
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id":1,
  "name":"Leanne Graham"
}
```

Your browser then displays the data.


|Part	|Meaning|
|-------|-------|
|https	|Protocol|
|jsonplaceholder.typicode.com	|Server/Host|
|/users/1	|Resource Path|


### What Is a Request?

An HTTP request is sent from a client (like your web browser or a mobile app) to a server.

It includes :
+ __The Method (Verb) :__ Tells the server what type of action to perform.
+ __The URL/URI :__ The exact address of the resource or data you are targeting.
+ __Headers :__ Extra metadata, such as what language you prefer, what type of device you are using, and security tokens.
+ __Body :__ The actual data you are sending to the server (e.g., the username and password you typed into a sign-up form)

### What Is a Response?

An HTTP response is the response that a server sends back to a client after receiving an HTTP request.

It includes :
+ __Status Line :__ The very first line of the message. It includes the HTTP version (e.g., HTTP/1.1) and a three-digit status code that indicates the outcome.
+ __Headers :__ A block of metadata organized as key-value pairs. These instruct the client how to handle the data by providing details like Content-Type (e.g., HTML text or a JSON object) and Cache-Control.
+ __Body :__ The actual payload or content of the message. It contains the visual layout or raw data, though it can be omitted if the server is only returning an error or metadata.

#### Example:
```
HTTP/1.1 200 OK

{
  "id":1,
  "name":"Leanne Graham"
}
```

## Most Important HTTP Methods

### GET
```
Fetch data.

GET /users/1
```

### POST /users

for insert data.

### PUT

Update everything.
```
PUT /users/1
```

Meaning :

Replace user 1 completely

### PATCH

Update only some fields.

PATCH /users/1

Example:

{
  "name":"New Name"
}

Only name changes.

### DELETE

Remove data.

DELETE /users/1

Meaning:

Delete user 1


## HTTP Status Codes

When the server responds, it gives a status.
```
200 OK
HTTP/1.1 200 OK
```
Success.

```
201 Created
HTTP/1.1 201 Created
```
New data created.

```
400 Bad Request
HTTP/1.1 400 Bad Request
```
Your request is wrong.

```
401 Unauthorized
HTTP/1.1 401 Unauthorized
```
Login/token missing.

```
403 Forbidden
HTTP/1.1 403 Forbidden
```
You are not allowed.

```
404 Not Found
HTTP/1.1 404 Not Found
```
Resource doesn't exist.

Example:

/users/999999

```
500 Internal Server Error
HTTP/1.1 500 Internal Server Error
```
Server crashed or has a bug.

### Headers

Headers contain extra information.

Example:
```
GET /users/1 HTTP/1.1

Accept: application/json
Authorization: Bearer xyz123
```

+ __Accept__ : Tell server what format you want

Example:
```
Accept: application/json
```

+ __Authorization__ : Prove who you are

Example:
```
Authorization: Bearer token123
```
Very common in APIs.

+ __Request Body__ : Only some methods use a body.

Example POST :
```
POST /users

{
  "name":"Supriyo",
  "age":30
}
```

The JSON part is called the body.

## GET Request

#### Step 1: Your First GET Request

```ruby
package org.example;
import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

public class App {
    public static void main(String[] args) throws IOException, InterruptedException {
        HttpClient client = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder().uri(URI.create("https://jsonplaceholder.typicode.com/users/1")).GET().build();
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        System.out.println(response.body());
    }
}
```

### Create Client
```
HttpClient client =
        HttpClient.newHttpClient();
```
Creates the HTTP engine.

Usually create once and reuse.

### Create Request
```
HttpRequest request =
        HttpRequest.newBuilder()
```
Starts building the request.

### Set URL
```
.uri(
     URI.create(
         "https://jsonplaceholder.typicode.com/users/1"))
```
Target API.

### Set Method
```
.GET()
```

### Build Request
```
.build();
```
Request becomes immutable.

### Send Request
```
HttpResponse<String> response =
        client.send(...)
```

Actually contacts the server.

Flow:
```
Java
 ↓
Internet
 ↓
Server
 ↓
Response
 ↓
Java
```

#### Step 2: Print Status Code
```
System.out.println(response.statusCode());
```

Output :
```
200
```


#### Step 3: Print Headers
```
System.out.println(response.headers());
```
Example:
```
Content-Type=[application/json]
Date=[...]
```

#### Step 4: Read Individual Header
```
String contentType =
        response.headers()
                .firstValue("Content-Type")
                .orElse("Unknown");
```

System.out.println(contentType);

Output:
```
application/json
```

#### Step 5: Add Request Headers

Many APIs require headers.
```
HttpRequest request =
        HttpRequest.newBuilder()
        .uri(URI.create(url))
        .header("Accept", "application/json")
        .GET()
        .build();
```

Generated request:
```
GET /users/1

Accept: application/json
```

#### Step 6: Multiple Headers
```
HttpRequest request =
        HttpRequest.newBuilder()
        .uri(URI.create(url))
        .header("Accept", "application/json")
        .header("Authorization",
                "Bearer abc123")
        .GET()
        .build();
```

## POST

#### Request Body

Unlike GET, POST usually contains data.

Example :
```
{
  "name":"Supriyo",
  "city":"Kolkata"
}
```
This JSON is called the request body.

#### Example :

```ruby
package org.example;

import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

public class App {
    public static void main(String[] args) throws IOException, InterruptedException {
        String json = """
                        {
                            "userId" : 1,
                            "title" : "Java learning",
                            "body" : "learning json"
                        }
                        """;
        HttpClient client = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder().uri(URI.create("https://jsonplaceholder.typicode.com/posts"))
        .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(json)).build();
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

        System.out.println(response.statusCode());
        System.out.println(response.body());
    }
}
```

### Content-Type Header
```
.header(
    "Content-Type",
    "application/json")
```

Tells server:
```
I am sending JSON
```

Without this, many APIs reject requests.

### POST Method
```
.POST(
    HttpRequest.BodyPublishers.ofString(json))
```

Equivalent HTTP request:
```
POST /posts

Content-Type: application/json

{
  "title":"Java Learning",
  "body":"Learning HttpClient",
  "userId":1
}
```

Expected Response

Status:

201

Meaning:

Created Successfully

Response:

{
  "title":"Java Learning",
  "body":"Learning HttpClient",
  "userId":1,
  "id":101
}
#### Common Status Codes for POST

|Code	|Meaning|
|-------|-------|
|200	|Success|
|201	|Created|
|400	|Invalid Request|
|401	|Unauthorized|
|403	|Forbidden|
|500	|Server Error|

### Sending Authorization Header

Many APIs require a token.

```
HttpRequest request =
        HttpRequest.newBuilder()
        .uri(URI.create(url))
        .header("Content-Type",
                "application/json")
        .header("Authorization",
                "Bearer myToken")
        .POST(
            HttpRequest.BodyPublishers.ofString(json))
        .build();
```

Equivalent :
```
Authorization: Bearer myToken
```

#### Difference Between GET and POST?
|GET	|POST|
|-------|----|
|Fetch data	|Send/Create data|
|Usually no body	|Usually contains body|
|Can be cached	|Usually not cached|
|Safe operation	|Changes data|

#### Why Content-Type Header?

Tells the server what format you're sending.

Example:
```
Content-Type: application/json
```

#### What is BodyPublisher?
````
HttpRequest.BodyPublishers.ofString(json)
```

Converts your String into an HTTP request body.



