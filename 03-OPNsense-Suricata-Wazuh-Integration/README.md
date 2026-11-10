# Practical 03 — OPNsense, Suricata and Wazuh Integration

## Aim
To configure OPNsense with Suricata IDS and integrate it with Wazuh SIEM for detecting, monitoring, and centrally analyzing network traffic and custom intrusion detection alerts.

## Requirements
- OPNsense firewall
- Wazuh SIEM
- Windows host system
- Windows 7 VM
- VirtualBox

## Procedure / Steps
1. Download the OPNsense ISO, create a VM with WAN and LAN adapters, install the system, assign interface addresses, and access its web GUI through the LAN address.
2. In **Services → Intrusion Detection → Administration**, enable intrusion detection, select the LAN interface, configure capture and logging options, and save.
3. Open the OPNsense console shell and create the custom rules directory:
   `mkdir -p /usr/local/etc/suricata/rules`
4. Edit `/usr/local/etc/suricata/rules/youtube.rules` and add the ICMP test rule:
   `alert icmp any any -> any any (msg:"ICMP Ping Detected"; sid:9000002; rev:1;)`
   Ping the OPNsense address from Windows and verify the alert in **Services → Intrusion Detection → Log File**.
5. Add the TLS SNI rule from the practical to detect YouTube access (SID `9000001`). Generate a test request with:
   `curl --insecure --resolve youtube.com:443:192.168.1.1 https://youtube.com`
   Verify **YouTube Access Detected** in the Suricata log.
6. To test using a browser, temporarily add the OPNsense address as the mapping for `youtube.com` and `www.youtube.com` in the Windows hosts file, then access the site. Remove these entries after testing.
7. Connect Kali and OPNsense to the same test network and generate YouTube traffic from Kali. Verify the detection alert.
8. Replace the detection action with the drop rule from the practical for YouTube TLS SNI traffic (SID `9000001`). Test again and verify that access is blocked and an alert is generated.
9. In OPNsense **System → Settings → Logging → Destinations**, configure remote log forwarding to the Wazuh server. Configure Wazuh to receive the OPNsense syslog messages on UDP port 514 from the gateway address.
10. Configure the required firewall rules and install the `os-wazuh-agent` plugin through **Firmware → Plugins**.
11. In **Services → Wazuh Agent → Settings**, configure the agent connection to the Wazuh manager and save.
12. Open the Wazuh dashboard, navigate to **Threat Hunting**, and verify that Suricata alerts from the OPNsense agent (shown in the practical as `OPNsense.internal`) are ingested.

## Expected Result
OPNsense Suricata logs the ICMP test and YouTube TLS SNI events, applies the configured drop rule during the blocking test, and forwards the resulting security events to Wazuh for centralized review in Threat Hunting.

## Conclusion
In this practical, OPNsense was configured with the Suricata Intrusion Detection System to monitor network traffic using custom detection rules. A custom ICMP rule was initially used to verify traffic monitoring, followed by a rule to detect access attempts to YouTube based on the TLS Server Name Indication (SNI). The generated Suricata alerts were successfully observed in the OPNsense Intrusion Detection logs. OPNsense was then integrated with Wazuh by configuring remote syslog forwarding and installing the Wazuh agent plugin to enable centralized log collection. Finally, the generated Suricata alerts were successfully ingested into the Wazuh Threat Hunting dashboard.
