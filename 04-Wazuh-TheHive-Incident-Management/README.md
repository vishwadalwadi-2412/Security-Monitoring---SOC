# Practical 04 — Wazuh and TheHive Incident Management Integration

## Aim
To integrate Wazuh with a free ticketing system for automated generation and centralized management of security alerts as incident tickets.

## Requirements
- Docker
- TheHive ticketing platform
- Wazuh Server
- DVWA test application

## Procedure / Steps
1. Create the directories for TheHive data and logs, and prepare the `docker-compose.yml` configuration with the container settings, ports, environment variables, and persistent volumes.
2. Verify the compose file and start TheHive in detached mode:
   `docker-compose up -d`
   Check the container using `docker ps`.
3. Open `http://localhost:9000` in a browser and sign in to TheHive using the initial credentials configured for the lab. Change or secure default credentials before exposing the service.
4. In **Account details**, create the Wazuh integration user `soc@thehive.local` with the org-admin profile. Generate an API key for the integration.
5. On the Wazuh server, create the custom integration script at `/var/ossec/integrations/custom-thehive`. The script uses TheHive's API to submit Wazuh alert data.
6. Create a mock alert JSON file at `/tmp/test-alert.json` to test the integration independently of a live event.
7. Compile and execute the integration:
   `sudo python3 -m py_compile /var/ossec/integrations/custom-thehive`
   `sudo /var/ossec/integrations/custom-thehive /tmp/test-alert.json`
   The practical records a successful API response of **STATUS: 201** when the test alert is created.
8. Start Apache and MySQL in XAMPP, open DVWA, and submit three failed login attempts to trigger Wazuh rule `100061`.
9. Open TheHive's Alerts dashboard at `http://localhost:9000/alerts` and verify that the corresponding alert was automatically created from the Wazuh event.

## Expected Result
The custom integration accepts a test alert with an HTTP 201 response, and a DVWA brute-force event matching Wazuh rule `100061` is automatically forwarded to TheHive and appears in its Alerts dashboard.

## Conclusion
TheHive was successfully deployed using Docker with persistent data and log storage. A dedicated soc@thehive.local user and API key were configured for Wazuh integration. A custom Python script was created on the Wazuh server to send Wazuh alerts to TheHive through its API. The integration was first tested using a mock JSON alert and successfully returned STATUS: 201, confirming alert creation. The DVWA login activity was then used to trigger Wazuh rule 100061 for repeated failed login attempts. Finally, the generated Wazuh alert was automatically received and displayed as an alert/ticket in TheHive.
