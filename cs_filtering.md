# Client side filtering

# Lesson 3
Inspecting the network traffic reveals that the coupon code is checked like this:
```
http://localhost:8080/WebGoat/clientSideFiltering/challenge-store/coupons/my-coupon-123
```

What if we call this URL without any coupon? Then all coupons are retuned:
```
GET http://localhost:8080/WebGoat/clientSideFiltering/challenge-store/coupons/
```
```json
{
  "codes" : [ {
    "code" : "webgoat",
    "discount" : 25
  }, {
    "code" : "owasp",
    "discount" : 25
  }, {
    "code" : "owasp-webgoat",
    "discount" : 50
  }, {
    "code" : "get_it_for_free",
    "discount" : 100
  } ]
}
```