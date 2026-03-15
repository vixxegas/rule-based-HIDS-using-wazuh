Wazuh. (n.d.). Deploying Wazuh agents on Linux systems - Wazuh agent. Documentation.wazuh.com. https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html
The refrence above was used for the installation for both the wazuh manager and agent.

**Wazuh Server installation in the Manager VM:**
1. sudo apt-get install gnupg apt-transport-https
	- this command adds two packages for preparation to install Wazuh

2. sudo curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && sudo chmod 644 /usr/share/keyrings/wazuh.gpg
	- may not install if not given elevated permission: including sudo within each command
	- installs Wazuh's signing key into dedicated system keyring so apt can verify Wazuh packages and sets proper file permissions for that key

3. echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee -a /etc/apt/sources.list.d/wazuh.list
	- Commands adds the Wazuh repository to my APT source with GPG signature verification, which allows for secure installation of Wazuh packages through apt.

4. sudo apt-get update
	- This updates the packages, prepared for wazuh installation

5. curl -sO https://packages.wazuh.com/4.10/wazuh-install.sh
6. sudo bash ./wazuh-install.sh -a
	- this command will install: the wazuh server, indexer and dashboard
	- it will also give us the username and password to access the dashboard

**Wazuh agent installation in the Client VM:**
1. repaet steps 1-4 from the wazuh server
	- Adds the wazuh repository and allows for secure installation

2. sudo WAZUH_MANAGER="10.0.0.2" apt-get install wazuh-agent
	- Deploys the wazuh agent

3. systemctl daemon-reload
4. systemctl enable wazuh-agent
5. systemctl start wazuh-agent
	- These commands enable and start the Wazuh-agent

**Troubleshooting:**

I ran into some difficulties due to a mistake i made when installing the manager. The agent was unable to be added to the server, due to wazuh manager being an older version of the agent. To fix this prblem, the solution was to remove the old version of wazuh manager and installing the correct and most recent version.

1. sudo /var/ossec/in/wazuh-control info | grep -i version
	- on both the manager and client VM, you can discover if  theres a problem with the versions

2. curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
3. sudo bash ./wazuh-install.sh -a -o
	- removes the old version of the wazuh server and installs the same version as the client

4. repeat steps 3-5, this should add the agent to the server. Should be now monitored by the manager and now should appear on the dashboard as active

5. sudo systemctl enable wazuh-manager wazuh-indexer wazuh-dashboard
	- this will autorun the manager, indexer and dashboard when the VM is booted up
