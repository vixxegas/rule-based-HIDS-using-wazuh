## Brute-Force rule creation

**Overview:**

This document describes the design and implementation of custom Wazuh rules for detecting brute-force attack behaviour on the agent VM. To achieve this goal, the steps were taken through these steps:
1. reviewing Wazuh's built-in detection rules
2. adding custom rules to meet the projects project detection design
3. Testing
4. Trouble shooting

**1.Built-in Rules reviewed:**

Identifying the pre-built wazuh rules that are used to detect brute-force related activity, I reviewed the following rule files:
1. sshd_rules.xml
2. pam_rules.xml
3. ossec_rules.xml
I discovered 2 pre-built rules that provides a baselines for authentication monitoring, but still needed more specific logic to be viable as a detection design:
1. 5710
2. 5503
3. 5501

**2.1 Custom SSH brute-force detection:**

A custom rule to detect repeated failed SSH login attempts from the same source IP, the intended threshold was 5 failed attempts within a minute. This will allow for a small number of legitimate mistakes while still detecting suspicious repeated failures.

ssh custom rule:

```xml
<group name="bruteforce,">
    <rule id="100002" level="10" frequency="5" timeframe=""60>
    <if_matched_sid>5710</if_matched_sid>
    <same_source_srcip/>
    <description>SSH bruteforce attempt, 5 login attempts from the same IP within a minute</description>
    </rule>
    </same_source_srcip>
    </rule>   
```

**2.2 local login detection:**

The detection design was one line for failed login and a successful line that were used to test the local login correlation logic.

local login custom rule:

```xml
    <rule id="100003" level="7" timeframe="120">
        <if_matched_sid>5503</if_matched_sid>
        <same_user/>
        <description>3 failed login attempts by the same user</description>
    </rule>
    <rule id="100004" level="10">
        <if_matched_sid>100003</if_matched_sid>
        <if_sid>5501</if_sid>
        <same_user/>
        <description>successful login after 3 failed attempts</description>
    </rule>
</group>
```

restarting the wazuh manager was necessary for changes to be applied:
systemctl restart wazuh-manager

**issues encountered:**

There were several errors that occured during the rule creation process:

1. custom rule could not be loaded because of XML formatting errors:

    <group name = "bruteforce,"> --> <group name="bruteforce,">

3. Rule ID conflict within one of the custom rules within local_rules.xml file
4. rule 100004 did not fire, while 100003 does - trouble shooting still needs to occur

**testing:**

Testing was done through wazuh dashboard rule testing capabilities, by inputing a valid log that matches either ssh/local-login brute-force, then identifying if the rules were fired or not.

**conclusion:**

The brute-force rule design extended wazuh's pre-built rules for authentication, which allows for a more logical way of monitoring authenticaion, but still being practical and reducing noise within the dashboard.
