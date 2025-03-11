# Wazuh Agent Installation Script(ubuntu)

This repository contains a bash script for automating the installation of Wazuh agents on Debian-based systems (Ubuntu, Debian, etc.).

## Overview

The script performs the following tasks:
- Installs necessary dependencies
- Adds the Wazuh GPG key and repository
- Installs the Wazuh agent
- Configures the agent to connect to your Wazuh manager
- Enables and starts the Wazuh agent service
- Secures the repository configuration to prevent accidental upgrades

## Prerequisites

- A Debian-based system (Ubuntu, Debian, etc.)
- Root privileges (sudo access)
- Internet connectivity to download packages
- A running Wazuh manager server

## Installation Script

```bash
#!/bin/bash

# Define Wazuh Manager IP
WAZUH_MANAGER="10.10.5.72"

# Check if the script is run as root
if [ "$EUID" -ne 0 ]; then
  echo "Please run this script as root or use sudo."
  exit 1
fi

# Install required dependencies
sudo apt-get update && sudo apt-get install -y gnupg apt-transport-https

# Add the GPG key
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && \
chmod 644 /usr/share/keyrings/wazuh.gpg

# Add the Wazuh repository
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | \
sudo tee /etc/apt/sources.list.d/wazuh.list

# Update package information
sudo apt-get update

# Install the Wazuh agent
sudo WAZUH_MANAGER="$WAZUH_MANAGER" apt-get install -y wazuh-agent

# Enable and start the Wazuh agent service
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent

# Disable Wazuh repository to prevent accidental upgrades
sudo chmod -x /etc/apt/sources.list.d/wazuh.list

# Print completion message
echo "Wazuh agent installation and setup completed successfully. The agent is now connected to the Wazuh manager at $WAZUH_MANAGER."
```

## Usage

1. **Download the script**:
   ```bash
   wget https://raw.githubusercontent.com/yourusername/wazuh-agent-installer/main/install-wazuh-agent.sh
   ```

2. **Make it executable**:
   ```bash
   chmod +x install-wazuh-agent.sh
   ```

3. **Edit the script** to change the `WAZUH_MANAGER` variable to your Wazuh manager's IP address:
   ```bash
   nano install-wazuh-agent.sh
   ```

4. **Run the script** with sudo privileges:
   ```bash
   sudo ./install-wazuh-agent.sh
   ```

## Verification

After installation, you can verify that the agent is running properly with:

```bash
sudo systemctl status wazuh-agent
```

The agent should also appear in the Wazuh dashboard within a few minutes after installation.

## Troubleshooting

If you encounter issues:

1. **Check agent status**:
   ```bash
   sudo systemctl status wazuh-agent
   ```

2. **View agent logs**:
   ```bash
   sudo tail -f /var/ossec/logs/ossec.log
   ```

3. **Verify connection to manager**:
   Edit the configuration file if needed:
   ```bash
   sudo nano /var/ossec/etc/ossec.conf
   ```

4. **Restart the agent** after configuration changes:
   ```bash
   sudo systemctl restart wazuh-agent
   ```

## Customization

The script can be modified to include additional configurations:

- **Agent name**: Add a `WAZUH_AGENT_NAME` variable and pass it during installation
- **Agent groups**: Configure agent groups in the ossec.conf file
- **Custom policies**: Add specific security policies for the agent

## For Other Operating Systems

This script is designed for Debian-based systems. For other operating systems:

- **CentOS/RHEL**: Use yum/dnf instead of apt
- **Windows**: Use PowerShell scripts or MSI installers
- **macOS**: Use brew or package installers

## Additional Resources

- [Official Wazuh Documentation](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html)
- [Wazuh Agent Configuration](https://documentation.wazuh.com/current/user-manual/agents/agent-configuration.html)
- [Wazuh GitHub Repository](https://github.com/wazuh/wazuh)
