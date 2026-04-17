## Privilege Escalation Rules

**Overview**

This document describes the design and implementation of custom Wazuh rules for detecting privilege-escalation on the agent wazuh. To achieve this goal, these were the steps taken:

1. Installing audit logs
2. log source
3. Wazuh's built-in detection rules
4. Custom rules to meet project requirements and design
5. Testing
6. Troubleshooting

**Audit logs**

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

**Built-in rules review**

reviewing the wazuh rules that are used to detect activities related to privilege escalation


