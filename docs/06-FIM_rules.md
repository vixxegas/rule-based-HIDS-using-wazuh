## FIM rules

**Overview**

The document describes the design and implementation of custom Wazuh file integrity monitoring (FIM) rules for detecting suspicious changes to critical files on the agent VM. To achieve this goal, the steps were taken through auditing critical files, adding custom log sources, reviewing wazuh's built-in detection logic and creating custom rules to meet the project requirements, testing and troubleshooting.

The mains paths that will be monitored and alerted if changes occur:

1. /etc/passwd
2. /etc/shadow
3. /etc/group
4. /etc/sudoers
5. /etc/ssh/sshd_config
6. /Documents/fim_test.txt

These paths chosen becuase of the sensitive authentication and configuration data, any changes to them may have major secuirty impact.

**Logs**

Within the agent VMs ossec.conf file, I added these lines to allow for the pre-built FIM to scan for the files listed above (this is written under the File Integrity MOnitoring section in the ossec.conf ):
```xml
    <directories realtime="yes" report_changes="yes">/etc</directories>
    <directories realtime="yes" report_changes="yes">/etc/ssh</directories>
    <directories realtime="yes" report_changes="yes">/home/wazuhagent/Documents</directories>
```
**Built-in rules**

The main pre-built in rules used will alert me if the files are created, deleted or modified.

The rule discovered:
- 550

**Custom Rules**

The following rules were added to alert me if there any changes to the files listed above, which is done in the wazuh-manager VM:
```xml
<group name="FIM,">
    <rule id="100005" level="12">
        <if_sid>550</if_sid>
        <field name="file">/etc/passwd</field>
        <description>modification towards /etc/passwd</description>
    </rule>

    <rule id="100006" level="12">
        <if_sid>550</if_sid>
        <field name="file">/etc/shadow</field>
        <description>modification towards /etc/shadow</description>
    </rule>

    <rule id="100007" level="12">
        <if_sid>550</if_sid>
        <field name="file">/etc/group</field>
        <description>modification towards /etc/group</description>
    </rule>

    <rule id="100008" level="12">
        <if_sid>550</if_sid>
        <field name="file">/etc/sudoers</field>
        <description>modification towards /etc/sudoers</description>
    </rule>

    <rule id="100009" level="12">
        <if_sid>550</if_sid>
        <field name="file">/etc/ssh/sshd_config</field>
        <description>modification towards /etc/ssh/sshd_config</description>
    </rule>

    <rule id="100010" level="12">
        <if_sid>550</if_sid>
        <field name="file">/home/wazuhagent/Documents/fim_test.txt</field>
        <description>for testing and demonstration</description>
    </rule>
</group>
```
Changes to the wazuh agent or manager, must be restarted:
```bash
systemctl restart wazuh-agent
```

```bash
systemctl restart wazuh-manager
```

**Testing**
Testing and was done by modifying the fim_test.txt file located in Documents and seeing if it would appear in the dashboard or the threat hunting section in the wazuh-manager VM.
```bash
echo "trigger" >> /home/wazuhagent/Documents/fim_test.txt
```

**Trouble shooting**

Initially it did not fire due to multiple reasons, the reasons being are the default settings and misconfiguration within the agents ossec.conf file:

1. The default timing scan was set to 12hrs, changes this to 60 seconds:
```xml
    <frequency>60<frequency>
```
2. I thought the ossec.conf would monitor individual files, however it does not. It monitors changes to directories and the rules configured in the manager VM will alert changes to individual files.

**Conclusion**
 These custom FIM rule detection, expanded on wazuh's built-in fim monitoring, which allowed for a more focused way to detect changes to files, but reducing the amount of noise produced.

    

