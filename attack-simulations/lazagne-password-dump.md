
# Attack Simulation: LaZagne Password Dump

## Objective
Simulate a credential extraction attack using LaZagne on a Windows 10 VM to test LimaCharlie’s detection and playbook response.

## Steps
1. Download LaZagne from the official repo.
2. Run the binary: `lazagne.exe all`
3. Observe process behavior and logs in LimaCharlie.
4. Tines triggers alert with full telemetry.
5. Follow isolation decision process (Slack/Email).
