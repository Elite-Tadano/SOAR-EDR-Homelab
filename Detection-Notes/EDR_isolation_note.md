Story - Tines
Create detection in LimaCharlie > Detect Hacktools > Tines > Slack & Email

Slack & Email will contain 
- Time
- Computer Name
- Source IP
- Process
- Command Line
- File Path
- Sensor ID
- Link to the detection (If applicable)

Tines > Promt user to isolate the machine (Yes / No)
- is yes: LimaCharlie should automatically isolate the machine and a message should be sent to Slack
- Message: Isolation status with a note "The computer <computer> has been isolated."
- if no: LimaCharlie will not isolate
- Message: Isolation status with a note "The computer <computer> has not isolated. Please investigate."
