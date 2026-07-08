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

***Active-Response - Disable user accounts - first attempt***

Within the Wazuh-agent VM there is a testuser, the current system has rules to detect against against privilege escalation and critical files being changed. To improve the security system is to disable the user when these rules are triggered and only be activated again by a root user (wazuhagent), to mitigate the damage that can be caused.

*Wazuh Manager*

```xml
<command>
  <name>disable-account</name>
  <executable>disable-account</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>
...
<active-response>
    <disabled>no</disabled>
    <command>disable-account</command>
    <location>local</location>
    <rules_id>100016</rules_id>
    <timeout>300</timeout>
</active-response>
```

*Wazuh Agent:*

Troubleshooting: 

The response triggered successfully in the Wazuh dashboard, and rule 100016 was observed as expected. However, checking the account status on the agent showed that testuser remained unlocked. Investigation showed that the decoded alert field dstuser was resolving to root, so the built-in disable-account action was attempting to lock the root account instead of testuser.

Result: 

The first attempt did not work as intended because the response target was derived from the alert context rather than being explicitly set to testuser. A reliable way to redirect dstuser to testuser could not be found, so the built-in active response path was abandoned.

***Active-Response - Disable user accounts - second attempt***

A custom active response script was introduced to bypass the dstuser problem and lock testuser directly. The script was placed on the agent VM and configured to run when rule 100016 fired. This approach made the containment action explicit and ensured that the intended account was disabled regardless of how the alert decoded the sudo event.

*Wazuh Agent:*

```bash
sudo nano /var/ossec/active-response/bin/disable-testuser.sh
```
in that new file add:
```bash
#!/bin/bash

USER_TO_DISABLE="testuser"

if id "$USER_TO_DISABLE" >/dev/null 2>&1; then
  /usr/bin/passwd -l "$USER_TO_DISABLE"
  /usr/bin/logger "Wazuh active response: locked user $USER_TO_DISABLE"
fi

exit 0
```
After give Wazuh the permission to run the script:
```bash
sudo chmod 750 /var/ossec/active-response/bin/disable-testuser.sh
sudo chown root:wazuh /var/ossec/active-response/bin/disable-testuser.sh
```

*Wazuh Manager:*

Modify the ossec.conf file to include:
```xml
<command>
  <name>disable-testuser</name>
  <executable>disable-testuser.sh</executable>
  <timeout_allowed>no</timeout_allowed>
</command>
...
<active-response>
  <disabled>no</disabled>
  <command>disable-testuser</command>
  <location>local</location>
  <rules_id>100016</rules_id>
</active-response>
```

*Result:*

When rule 100016 was triggered, the custom active response script executed successfully on the agent and locked testuser. The account status changed from P to L, confirming that the user password was locked as intended.
