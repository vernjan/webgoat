# (A2) Password reset

## Lesson 6 - Password reset
The idea is to intercept the password reset request and tamper the `Host` header. This header is used for creating
the password reset link (hint). Change the header to `localhost:9090` (or were your _WebWolf_ runs) and once "Tom clicks
the reset link", you will see the request captured in _WebWolf_. Grab the token and use it for changing Tom's password
(you should ask for a password reset on your account first to get the password reset link).