These are the improvements that was notated in my report to try improve the HIDS. The end goal is for the HIDS to becom a secruity system able to detect and respond to threats using Wazuh capabilties of active response

**Active-Response**

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
