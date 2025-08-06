# EDR Homelab: Automated Threat Detection & Response

This repository documents the setup and implementation of a fully functional EDR and SOAR homelab. The project leverages LimaCharlie for endpoint security and Tines for automation to create a powerful, automated incident response pipeline.

## 🎯 Project Goals

* **Automated Detection:** Proactively identify malicious or suspicious activities on a monitored endpoint using custom detection rules.
* **Real-time Alerting:** Instantly notify security analysts via Slack and email with detailed information about the detection.
* **Interactive Response:** Provide analysts with the ability to take immediate action, such as isolating a compromised host, directly from the notification.
* **End-to-End Automation:** Create a seamless workflow from initial detection to final response, minimizing manual effort and response time.
* **Hands-On Learning:** Gain practical experience with industry-standard EDR and SOAR tools and concepts.

## ⚙️ Technologies Used

* **Virtualization:** Any hypervisor (e.g., VirtualBox, VMware, Proxmox)
* **Endpoint:** A virtual machine to act as the "victim" or monitored system (e.g., Windows 11).
* **Attacker Machine:** A virtual machine for simulating attacks (e.g., Kali Linux, Ubuntu Server).
* **EDR:** [LimaCharlie](https://limacharlie.io/)
* **SOAR:** [Tines](https://www.tines.com/)
* **Notifications:** Slack, Email

## 🗺️ Architectural Diagram

A high-level overview of the data flow and component interaction:

1.  **Detection:** A malicious action (e.g., execution of a hacking tool) occurs on the Windows endpoint.
2.  **Telemetry Collection:** The LimaCharlie sensor on the endpoint collects telemetry and sends it to the LimaCharlie cloud.
3.  **Rule Match:** A custom Detection & Response (D&R) rule in LimaCharlie matches the malicious activity and generates a detection.
4.  **Webhook to SOAR:** LimaCharlie forwards the detection data to a Tines webhook.
5.  **Playbook Execution:** The Tines playbook (Story) is triggered and begins to process the alert.
6.  **Notification:** Tines sends a detailed message to a designated Slack channel and an email to the security team. The message includes key details like hostname, process, and a link back to the LimaCharlie detection.
7.  **Analyst Decision:** The Slack message contains interactive buttons ("Isolate Host" or "Ignore").
8.  **Automated Response:**
    * If "Isolate Host" is clicked, Tines sends a command back to the LimaCharlie API to isolate the endpoint from the network. A confirmation message is sent to Slack.
    * If "Ignore" is clicked, the event is logged, and a message is sent to Slack indicating that no action was taken.

## 🛠️ Implementation Steps

### Part 1: Environment & EDR Setup

1.  **Create Virtual Machines:** Set up two VMs: a Windows machine (victim) and a Linux machine (attacker). Ensure they can communicate with each other and the internet.
2.  **Create a LimaCharlie Account:** Sign up for a free LimaCharlie account.
3.  **Create an Installation Key:** In your LimaCharlie organization, create an installation key to register your sensor.
4.  **Install LimaCharlie Sensor:** On the Windows VM, download and run the LimaCharlie installer using the installation key you created.
5.  **Verify Connectivity:** In the LimaCharlie web interface, verify that the new sensor is online and sending telemetry.

### Part 2: Detection Rule Creation

1.  **Identify Malicious Behavior:** Determine what you want to detect. For this project, a common example is detecting the use of a known hacking tool like `LaZagne.exe`.
2.  **Write a D&R Rule:** In LimaCharlie, navigate to `D&R Rules` and create a new rule. The rule will look for new processes where the file path ends with the name of the tool.

    ```yaml
    # Rule to Detect LaZagne.exe
    events:
     - NEW_PROCESS
     - EXISTING_PROCESS
    op: and
    rules:
     - op: is windows
     - op: or
       rules:
         - case sensitive: false
           op: ends with
           path: event/FILE_PATH
           value: lazagne.exe
         - case sensitive: false
           op: ends with
           path: event/COMMAND_LINE
           value: all
         - case sensitive: false
           op: contains
           path: event/COMMAND_LINE
           value: lazagne
         - case sensitive: false
           op: is
           path: event/HASH
           value: dc06d62ee95062e714f2566c95b8edaabfd387023b1bf98a09078b84007d5268
    ```
3.  **Define the Response:** Configure the rule to generate a `report` action. This creates a formal detection in LimaCharlie.

    ```yaml
    # Response Action
    - action: report
      metadata:
        author: Jugal
        description: Detects Lazagne  (SOAR-EDR-Tools) from view
        falsepositives:
          - Unlikely
        level: medium
        tags:
          - attack_credentials_access
      name: EDR-Hacktools-Lazagne
    ```

### Part 3: Tines & Communication Channel Setup

1.  **Create a Tines Account:** Sign up for the free Tines Community Edition.
2.  **Set up a Slack Workspace:** Create a free Slack workspace and a dedicated channel for security alerts (e.g., `#security-alerts`).
3.  **Create a Tines Webhook:** In a new Tines Story, drag a `Webhook` action onto the storyboard. This will generate a unique URL.
4.  **Configure LimaCharlie Output:** In LimaCharlie, go to `Outputs`, add a new output, and select `Tines` as the destination. Paste the webhook URL from Tines. This will now send all detections to your Tines Story.

### Part 4: Building the Automation Playbook in Tines

1.  **Receive and Parse Data:** The `Webhook` action will receive the detection data from LimaCharlie.
2.  **Send Slack Notification:**
    * Add an `Send to Slack` action.
    * Connect it to the `Webhook` action.
    * Configure the message to include dynamic data from the LimaCharlie event (e.g., `{{.webhook.routing.hostname}}`, `{{.webhook.detect.event.FILE_PATH}}`).
    * Use Slack's Block Kit to add interactive buttons for "Isolate Host" and "Ignore".
3.  **Send Email Notification:**
    * Add a `Send Email` action.
    * Configure it to send a detailed alert to a specified email address.
4.  **Handle Interactive Response:**
    * Add an `HTTP Request` action for the "Isolate Host" path.
    * Configure it to make a POST request to the LimaCharlie API endpoint for isolating a sensor.
    * You will need to create a LimaCharlie API key with the necessary permissions and store it as a Credential in Tines.
    * The Sensor ID (`sid`) will be available from the initial webhook data.
5.  **Provide Feedback:**
    * Add final `Send to Slack` actions to confirm the outcome (e.g., "Host has been isolated" or "Incident ignored").
<img width="1023" height="762" alt="Screenshot 2025-07-08 192702" src="https://github.com/user-attachments/assets/c7cdd0f7-5620-4267-9cee-e5b36939db4a" />

## 🚀 Running the Lab

1.  On the attacker machine, host a malicious file (e.g., using a simple Python web server).
2.  On the Windows victim machine, download and execute the malicious file.
3.  Observe the detection appearing in LimaCharlie.
4.  Check your Slack channel and email for the automated alert from Tines.
<img width="1337" height="289" alt="Screenshot 2025-07-07 202233" src="https://github.com/user-attachments/assets/a1c373a1-fa6a-4f1c-a8d2-695a312d5c88" />

5.  Click the "Isolate Host" button in Slack.
6.  Verify in LimaCharlie that the endpoint is now isolated and in your Slack channel that the confirmation message has been received.

This project provides a solid foundation for building more complex security automations and is an excellent showcase of practical defensive cybersecurity skills.
