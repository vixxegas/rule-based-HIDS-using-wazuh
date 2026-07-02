These are the improvements that was notated in my report to try improve the HIDS. The end goal is for the HIDS to becom a secruity system able to detect and respond to threats using Wazuh capabilties of active response

***Active-Response***

**Wazuh manager**

The ossec.conf file was checked and modified to ensure that active response is active and specifies what rules need to be triggered to timeout an IP address:
```xml
 <command>
  <name>firewall-drop</name>
  <executable>firewall-drop</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>
...
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>100002,100003</rules_id>
  <timeout>300</timeout>
</active-response>
```xml
