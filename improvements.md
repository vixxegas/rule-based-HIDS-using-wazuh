These are the improvements that was noted in my report to try improve the HIDS. The end goal is for the HIDS to becom a secruity system able to detect and respond to threats using Wazuh capabilties of active response

***Active-Response - brute-force***

*Wazuh manager:*

The ossec.conf file was checked and modified to ensure that active response is active and specifies what rules need to be triggered to timeout an IP address.

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
```

*Wazuh agent and other VM:*

Another VM was used to perform a shh brute-force attempt (hydra) five times against the agent VM. Wazuh applied a temporary DROP rule on the agent VM firewall for the attacking IP address, confirming that rule 100002 can now trigger both detection and containment - improving the system and moving towards to a secruity system that can respond to threats.

***Active-Response - Disable user accounts***

Within the Wazuh-agent VM there is a testuser, the current system has rules to detect against against privilege escalation and critical files being changed. To improve the security system is to disable the user when these rules are triggered and only be activated again by a root user (wazuhagent), to mitigate the damage that can be caused.
