# Practical 07 — Telegram-Based Automated IP Blocking and Unblocking with Wazuh
> Security Monitoring & SOC | Practical 07


## Aim
To demonstrate an automated workflow in which a security alert is sent to a messaging application and an authorized action button triggers blocking of the specified IP address.

## Requirements
- Telegram
- Wazuh SIEM
- Windows 10 VM
- Ubuntu Server

## Procedure / Steps
1. On the Wazuh server, edit `/var/ossec/etc/ossec.conf` and add the `custom-thehive` and `custom-telegram` integration blocks scoped to rule ID `100061`.
2. Configure `/var/ossec/integrations/custom-telegram` to extract the rule ID, level, agent, source IP, and description from the Wazuh alert JSON. Format the Telegram message with inline **Block IP** and **Unblock IP** buttons. Keep bot credentials private.
3. Create `/opt/telegram-listener.py`. As described in the practical, the listener uses `requests`, `subprocess`, `time`, and `winrm`, and connects to the Windows host through WinRM with NTLM authentication. Store Windows credentials securely rather than in the repository.
4. Implement the listener's block and unblock functions to execute Windows Firewall `netsh` commands remotely. Poll Telegram for callback events, extract the source IP and selected action, run the appropriate command, and send a result message.
5. Create `/etc/systemd/system/telegram-listener.service` to run the listener as a background service, restart it on failure, and start it at boot. Reload systemd and enable/start the service as configured in the lab.
6. On the Windows host, check firewall profile status:
   `netsh advfirewall show allprofiles state`
   Enable the profiles for the test as required:
   `netsh advfirewall set allprofiles state on`
   The practical notes that firewall settings should be restored after testing.
7. From the Windows 10 VM, open the DVWA login page hosted on the test system and submit three failed logins. Confirm Wazuh rule `100061` (**DVWA: 3 Login attempts detected (Possible Brute Force)**) is triggered with the VM's source IP.
8. Confirm that Telegram receives the alert with rule ID, level, agent, source IP, description, and the two response buttons.
9. Run `sudo python3 /opt/telegram-listener.py` for the practical's manual listener test. Click **Block IP** and verify that the listener executes the Windows Firewall command, logs the action to syslog, and replies in Telegram.
10. Verify that Windows Firewall rules `BLOCK_IN` and `BLOCK_OUT` are created for the source IP and that DVWA is no longer reachable from the test VM.
11. Click **Unblock IP**. Verify that the firewall rules are removed, DVWA becomes reachable again, and Telegram confirms the unblock action.
12. Configure Wazuh to monitor `/var/log/syslog`. Add rules `100062` and `100063` in `local_rules.xml` to detect the `wazuh-block` program entries for blocked and unblocked events. Restart the Wazuh manager.
13. In Wazuh Threat Hunting, verify the response event, including rule `100062` (**SOC Analyst blocked an IP via Telegram**), and check the corresponding unblock event.

## Expected Result
A Wazuh brute-force alert is delivered to Telegram with analyst action buttons. Selecting **Block IP** creates Windows Firewall block rules and prevents access to DVWA; selecting **Unblock IP** removes the rules and restores access. The response events are written to syslog and detected in Wazuh.

## Conclusion
In this practical, Wazuh was integrated with a Telegram bot to provide automated alert notification and response capabilities. A DVWA brute-force event triggered rule 100061, and the alert was forwarded to Telegram with Block IP and Unblock IP options. The Telegram listener processed the selected action and remotely created or removed Windows Firewall rules for the source IP. The blocking action prevented access to DVWA, while the unblock action restored connectivity. Finally, block and unblock activities were logged through syslog and monitored in Wazuh using rules 100062 and 100063, completing the automated detection, response, and monitoring cycle.
