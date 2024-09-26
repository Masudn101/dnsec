# dnsec

To configure DNSSEC for the domain `abc.com` in a script on RHEL 8, you can create a Bash script that automates the process outlined earlier. Here's an example script:

### DNSSEC Configuration Script: `configure_dnssec.sh`

```bash
#!/bin/bash

# Variables
DOMAIN="abc.com"
ZONE_FILE="/var/named/${DOMAIN}.zone"
SIGNED_ZONE_FILE="/var/named/${DOMAIN}.zone.signed"
KEY_DIR="/etc/named"
KEY_SIZE=2048
ALGORITHM="RSASHA256"

# Install required packages
echo "Installing BIND and utilities..."
sudo dnf install -y bind bind-utils

# Check if the named.conf file exists
if [ ! -f /etc/named.conf ]; then
  echo "Error: /etc/named.conf not found. Make sure BIND is installed."
  exit 1
fi

# Enable DNSSEC in named.conf
echo "Configuring DNSSEC in /etc/named.conf..."
sudo sed -i '/options {/a \
    dnssec-enable yes; \
    dnssec-validation yes; \
    dnssec-lookaside auto;' /etc/named.conf

# Generate DNSSEC keys
echo "Generating DNSSEC keys for $DOMAIN..."
cd $KEY_DIR
sudo dnssec-keygen -a $ALGORITHM -b $KEY_SIZE -n ZONE $DOMAIN

# Retrieve key filenames
KEYFILE=$(ls K$DOMAIN.*.key)
PRIVATE_KEYFILE=$(ls K$DOMAIN.*.private)

# Sign the zone file
echo "Signing the zone file $ZONE_FILE..."
sudo dnssec-signzone -o $DOMAIN $ZONE_FILE

# Update named.conf to use the signed zone
echo "Updating /etc/named.conf with the signed zone..."
sudo sed -i "/zone \"$DOMAIN\"/a \
    type master; \
    file \"$SIGNED_ZONE_FILE\"; \
    key-directory \"$KEY_DIR\"; \
    auto-dnssec maintain; \
    inline-signing yes;" /etc/named.conf

# Restart BIND service
echo "Restarting the BIND service..."
sudo systemctl restart named

# Enable named service on boot
echo "Enabling named service on boot..."
sudo systemctl enable named

# Test DNSSEC
echo "Testing DNSSEC for $DOMAIN..."
dig +dnssec $DOMAIN

echo "DNSSEC configuration for $DOMAIN completed successfully."
```

### Steps to Use the Script:

1. **Save the script:**
   Save the script as `configure_dnssec.sh` in your home directory or wherever you prefer.

2. **Make the script executable:**
   Give the script execution permission:

   ```bash
   chmod +x configure_dnssec.sh
   ```

3. **Run the script:**
   Execute the script with sudo privileges:

   ```bash
   sudo ./configure_dnssec.sh
   ```

4. **Ensure the zone file exists:**
   Make sure that the zone file for `abc.com` exists at `/var/named/abc.com.zone`. If it doesn't exist, you'll need to create it before running the script.

### Zone File Example:
Here is an example of what the `/var/named/abc.com.zone` file might look like:

```
$TTL 86400
@   IN  SOA ns1.abc.com. admin.abc.com. (
        2024092601  ; Serial
        3600        ; Refresh
        1800        ; Retry
        1209600     ; Expire
        86400 )     ; Minimum TTL

    IN  NS  ns1.abc.com.
    IN  A   192.168.1.10

ns1 IN  A   192.168.1.10
```

This script should automate the process of enabling DNSSEC for the domain `abc.com`. Let me know if you need further customizations!
