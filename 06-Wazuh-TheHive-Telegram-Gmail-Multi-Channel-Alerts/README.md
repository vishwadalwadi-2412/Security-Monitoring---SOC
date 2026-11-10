# Practical 06 — Multi-Channel SOC Alert Notifications with TheHive, Telegram and Gmail
> Security Monitoring & SOC | Practical 06


## Aim
To integrate a ticketing system with Gmail and a messaging application for automated alert generation and multi-channel notification of security incidents.

## Requirements
- TheHive
- Telegram
- Gmail account
- Wazuh

## Procedure / Steps
1. Deploy TheHive using the Docker Compose configuration and configure the Wazuh-to-TheHive custom Python integration at `/var/ossec/integrations/custom-thehive`.
2. In Telegram, open BotFather, start the conversation, create a bot, and record its bot token securely. The practical uses the bot username `TheWazuhHiveAlert_bot`.
3. Obtain the Telegram chat ID by sending a test message to the bot and checking the Bot API `getUpdates` response. Do not publish the bot token or chat ID in this repository.
4. On the Wazuh server, create `/var/ossec/integrations/custom-telegram`. Configure it to use the bot token and chat ID and send the alert details to Telegram.
5. Set the script ownership and permissions, compile it, and test it with the sample alert JSON:
   `sudo python3 -m py_compile /var/ossec/integrations/custom-telegram`
   `sudo /var/ossec/integrations/custom-telegram /tmp/test-alert.json`
6. Add a `custom-telegram` integration block to `/var/ossec/etc/ossec.conf`, scoped to Wazuh rule ID `100061`.
7. Submit three failed DVWA login attempts and confirm that rule `100061` (**DVWA: 3 Login attempts detected**) appears in Wazuh Threat Hunting. Verify that the event is reflected in TheHive and Telegram.
8. In TheHive administrator settings, open **Platform Management → SMTP** and configure Gmail SMTP using `smtp.gmail.com` on port `587`. Use an app password or other supported secure authentication method; do not commit credentials.
9. Sign in as the TheHive notification user, create a notification named **Wazuh Alert Notification**, set its trigger to **AnyEvent**, and configure the `EmailerToAddr` action as described in the practical.
10. Trigger rule `100061` again by submitting three failed DVWA logins.
11. Check the Gmail inbox for **Wazuh Alert - New Security Event** and verify that the message contains the alert details, including severity and description.

## Expected Result
A DVWA brute-force alert matching rule `100061` is forwarded to TheHive and Telegram. TheHive's SMTP notification sends an email to Gmail containing the security event details.

## Conclusion
Wazuh was successfully integrated with TheHive, Telegram, and Gmail for automated security alert notification. A custom Telegram integration was configured using a BotFather-generated bot token and chat ID, and its communication was verified successfully. Wazuh rule 100061 was used to detect repeated failed DVWA login attempts and forward the resulting alert to TheHive and Telegram. Gmail SMTP was then configured in TheHive, followed by the creation of an email notification for security events. A second DVWA brute-force simulation successfully triggered the rule and generated notifications. The alert details were consequently reflected in TheHive, Telegram, and Gmail, demonstrating multi-channel security alerting and incident notification.
