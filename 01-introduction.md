#1. Introduction

##1.1 Project Overview
This project implements a rule-based Host Intrusion Detection System (HIDS) using Wazuh.  
The goal is to monitor a client machine for malicious activity, detect attacks, and generate alerts using both built-in and custom detection rules.  

The system consists of three virtual machines:
- **Wazuh Manager (Ubuntu Server)** – Runs the manager, dashboard, and stores alerts.  
- **Client Machine (Ubuntu)** – Runs the Wazuh agent and is the monitored host.  
- **Attacker Machine (Kali Linux)** – Simulates attacks on the client to test detection capabilities.

##1.2 Motivation
Host-based intrusion detection is crucial for identifying suspicious activity on servers and workstations.  
Rule-based HIDS solutions like Wazuh allow organizations to detect attacks in real-time and respond quickly to threats.

The dissertation focuses on:
- Deploying a functional HIDS environment in virtual machines.  
- Creating custom rules to detect attacks such as SSH brute-force, port scans, and file integrity violations.  
- Demonstrating the effectiveness of Wazuh in detecting simulated attacks from a controlled attacker environment.

##1.3 Objectives
The main objectives of the project are:
1. Deploy a virtualised environment with a Wazuh manager, a client agent, and an attacker machine.  
2. Install and configure Wazuh on both manager and client.  
3. Develop custom rules for detecting malicious activity.  
4. Simulate attacks from the Kali Linux machine.  
5. Evaluate the detection capability of the system through generated alerts.  

##1.4 Scope
- Host-level monitoring using Wazuh agents.  
- Rule-based detection of attacks (no network intrusion detection).  
- Simulated attack scenarios in a controlled virtual environment.  
