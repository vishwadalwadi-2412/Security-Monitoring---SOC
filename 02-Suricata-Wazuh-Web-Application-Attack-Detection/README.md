# Practical 02 — Suricata IDS and Wazuh Web Application Attack Detection
> Security Monitoring & SOC | Practical 02


## Aim
To configure and integrate Suricata IDS with Wazuh SIEM for detecting and monitoring custom web application attacks using custom detection rules.

## Requirements
- Wazuh open-source security platform
- Ubuntu Server
- Google Chrome browser
- Suricata
- Windows OS with XAMPP and DVWA
- Kali Linux OS

## Procedure / Steps
1. Download and install Suricata from the official Suricata download page.
2. In `C:\Program Files\Suricata\rules`, create `local.rules` and add the custom rule from the practical to detect an IP packet containing the text `DVWA` in its payload (SID `888888`).
3. Configure the Windows Wazuh agent's `ossec.conf` to monitor Suricata's `eve.json` log in JSON format.
4. Edit the Suricata YAML configuration as required in the practical.
5. Start Suricata against the selected interface:
   `suricata.exe -c suricata.yaml -i <ip_address> -k none`
6. Generate test traffic:
   `curl -v http://example.com/?attack=DVWA`
   Check Suricata's `fast.log` for **DVWA RAW PAYLOAD DETECTED!**
7. In Wazuh **Threat hunting**, search for `data.alert.signature: "CRITICAL: DVWA RAW PAYLOAD DETECTED!"` and verify the alert.
8. Modify `local.rules` to add the SQL Injection detection rule described in the practical. It searches TCP traffic for the `sqli` or `OR` patterns and uses SID `1000017`.
9. Restart Suricata on the Kali interface:
   `suricata -c suricata.yaml -i <IP_address_of_Kali>`
10. Start Apache and MySQL in XAMPP. From Kali, open the DVWA application hosted on Windows, sign in, navigate to the SQL Injection section, and submit the test input `' OR '1'='1`.
11. Check Suricata's `fast.log` for the SQL Injection alert.
12. In Wazuh **Threat hunting**, search for `data.alert.signature: "CUSTOM Web Application SQL Injection (OR Logic Bypass)"` and verify that the event is displayed.

## Expected Result
Suricata detects the custom DVWA payload and SQL Injection traffic, records the matching signatures in its logs, and the Wazuh agent forwards the events so they can be searched in the Threat Hunting dashboard.

## Conclusion
In this practical, Suricata IDS was successfully integrated with Wazuh to enable centralized monitoring and detection of network-based web application attacks. Custom Suricata rules were configured to detect both a specific DVWA payload and SQL Injection attack patterns, while the Wazuh agent was configured to monitor the generated eve.json logs. The Suricata engine inspected live network traffic and produced alerts whenever the defined signatures were matched. These alerts were successfully forwarded to Wazuh and visualized through the Threat Hunting dashboard.
