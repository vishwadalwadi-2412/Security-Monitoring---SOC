# Practical 01 — Wazuh DVWA Brute-Force Detection
> Security Monitoring & SOC | Practical 01


## Aim
To install Wazuh and configure it to detect and visualize brute-force login attempts on DVWA by creating custom detection rules and security dashboards.

## Requirements
- Wazuh open-source security platform
- Ubuntu Server
- Google Chrome browser
- Windows system with XAMPP
- DVWA web application and Wazuh agent

## Procedure / Steps
1. Install the Wazuh all-in-one deployment on Ubuntu:
   `curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a`
2. Find the Ubuntu server IP using `hostname -I`. Open `https://<IP_Address>` in Chrome and sign in to the Wazuh dashboard.
3. In **Server management → Endpoint summary**, choose **Add agent**, select Windows, enter the Wazuh server address and an agent name, then run the generated installation and service-start commands in PowerShell as Administrator.
4. Confirm that the Windows agent appears as active. Open **Threat intelligence → Threat hunting** to review collected Windows events.
5. Install DVWA under the XAMPP `htdocs` directory. Rename `config.inc.php.dist` to `config.inc.php`, configure the database user as `root` with a blank password as in the practical, start Apache and MySQL, and open `http://localhost/DVWA-master/login.php`.
6. Configure the Windows Wazuh agent's `ossec.conf` to monitor the XAMPP Apache `access.log`. Restart the Wazuh service.
7. In the Wazuh dashboard, open **Server Management → Manage rules files**, edit `local_rules.xml`, and add the custom rules described in the practical: a rule for DVWA `POST` login requests and a frequency rule that detects three attempts from the same source IP within 60 seconds (rule ID `100061`).
8. Use **Ruleset Test** with a POST entry from Apache `access.log` to check pre-decoding, decoding and rule matching.
9. Submit three failed DVWA logins within one minute. Check **Threat hunting** and confirm rule `100061` is generated.
10. Create a Metric visualization using `wazuh-alerts-*`, filter `rule.id = 100061`, and save it as **DVWA 3-Login Brute Force Alerts**.
11. Create an Area chart filtered on rule `100061`, with `@timestamp` as a date histogram, and save as **DVWA Attack Timeline**. Create a Pie chart using `data.srcip` terms and save as **DVWA Attacker IPs**.
12. Create the **DVWA Security Monitoring** dashboard, add the three visualizations, arrange them, and save.

## Expected Result
Wazuh collects the DVWA Apache access logs, identifies repeated failed login attempts from the same source IP within 60 seconds, generates rule `100061`, and displays the events in the Threat Hunting view and the **DVWA Security Monitoring** dashboard.

## Conclusion
In this practical, Wazuh was configured to monitor web application activity and detect brute-force login attempts using custom correlation rules. By integrating DVWA with Wazuh, collecting Apache access logs, and creating visual dashboards, the security events were successfully detected, analyzed, and presented in an organized manner.
