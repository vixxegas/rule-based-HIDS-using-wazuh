## Privilege Escalation Rules

**Overview:**

This document describes the design and implementation of custom Wazuh rules for detecting privilege-escalation on the agent wazuh. To achieve this goal, these were the steps taken:

1. Installing audit logs
2. log source
3. Wazuh's built-in detection rules
4. Custom rules to meet project requirements and design
5. Testing
6. Troubleshooting

**Audit logs:**

This is needed because it'll allow wazuh to view raw system data that will trigger the rules, therefore alerting for privilege escalation.
```bash
systemctl start auditd
systemctl enable auditd
```
**log source**

Adding these rules to the 'audit.rules' file.
```text
-a always,exit -F arch=b32 -S execve -F auid=1000 -F egid!=994 -F auid!=-1 -F key=audit-wazuh-c
-a always,exit -F arch=b64 -S execve -F auid=1000 -F egid!=994 -F auid!=-1 -F key=audit-wazuh-c
-w /home -p w -k audit-wazuh-w
-w /home -p a -k audit-wazuh-a
-w /home -p r -k audit-wazuh-r
-w /home -p x -k audit-wazuh-x

-a always,exit -F arch=32 -S execve -F path=usr/bin/sudo -k priv_esc
-a always,exit -F arch=64 -S execve -F path=usr/bin/sudo -k priv_esc

-a always,exit -F arch=32 -S execve -F path=/bin/su -k priv_esc
-a always,exit -F arch=64 -S execve -F path=/bin/su -k priv_esc

-a always,exit -F arch=32 -S execve -F path=usr/sbin/usermod -k priv_esc
-a always,exit -F arch=64 -S execve -F path=usr/sbin/usermod -k priv_esc

-a always,exit -F arch=32 -S execve -F path=usr/sbin/groupmod -k priv_esc
-a always,exit -F arch=64 -S execve -F path=usr/sbin/groupmod -k priv_esc

-a always,exit -F arch=32 -S execve -F path=usr/bin/visudo -k priv_esc
-a always,exit -F arch=64 -S execve -F path=usr/bin/visudo -k priv_esc

-a always,exit -F arch=32 -S execve -F path=usr/bin/sudoedit -k priv_esc
-a always,exit -F arch=64 -S execve -F path=usr/bin/sudoedit -k priv_esc
```
```bash
auditctl -R
``` 

After adding the audit log to the 'ossec.conf' file, which allows for wazuh to read the log files.
```xml
<localfile>
    <log_format>syslog</log_format>
    <location>/var/log/audit/audit.log</location>
</localfile>

<localfile>
    <log_format>syslog</log_format>
    <location>/var/log/auth.log</location>
</localfile>
```
**Built-in rules review:**

reviewing the wazuh rules that are used to detect activities related to privilege escalation. These rules provided a baseline for audit and authentication monitoring, but still needed more specific logic to fit the detection design.

The rules discovered:

1. 80700
2. 5404
3. 5402
4. 5403_

**Log detection:**
The custom rule design focused on audit and authentication logs. I configured auditd to monitor the relevant activity and added rules that would trigger when the 'priv_esc' audit key was detected, so any changes to the directories listed in the 'auditd.rules' files will trigger an alert.
```xml
<group name="privilege_escalation,">
    <rule id="1000011" level="12">
        <if_sid>80700</if_sid>
        <field name="audit_key">priv_esc</field>
        <description>changed towards a critical directory</description>
    </rule>
    <rule id="100012" level="5">
        <if_sid>80700</if_sid>
        <field name="audit.key">audit-wazuh-c</field>
        <description>changes to home directory</description>
    </rule>
```
**Sustom sudo detection:**
I also added custom rules to monitor 'auth.log' for sudo related activity. The goal was to detect suspicious sudo usage and failed password attempts.
```xml
    <rule id="100013" level="12">
        <if_sid>5404</if_sid>
        <match>incorrect password|authentication failure</match>
        <description>failed sudo authentication attempt</description>
    </rule>
    <rule id="100015" level="12">
        <if_sid>5402, 5403</if_sid><regex>passwd|chmod|chown|curl|wget|useradd|usermod|groupmod|visudo|sudoedit</regex>
        <description>suspciosus command via sudo</description>
</group>
```
```bash
systemctl restart wazuh-manager
```
To apply the changes

**Testing:**
Testing was carried out with wazuh dashboard log testing and 'wazuh-logtest'. I used valid log lines that matched the audit and sudo detection patterns to check whether the custom rules fired correctly.

Another way of testing that was done, was seeing if the rules would fire if I performed privilege-escalation related scenrios in the agent VM.

**Troubleshooting:**
when a rule did not fire, I would check the parent rule relationship, log format and XML syntax. I also compared the event against the built-in wazuh rule chain to make sure custom rules was attached to the correct parent event.

This trouble shooting was important due to some events matched different parent sudo rules, so detection design had to account for both cases.

Further trouble shooting has to be done for audit and hoem directory rules as they did not fire.

**conclusion**
The privilege-escalation rule detection design extended wazuh's built-in authentication and audit monitoring, which created a more focused way to detect suspicious escalation activity while reducing noise in the dashboard. The final worked in testing and demonstrated that custom rules can improve the visibility into privilege-escalated bahviour.
