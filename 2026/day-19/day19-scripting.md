# Day 19 – Shell Scripting: Log Rotation, Backups & Scheduled Maintenance
## Overview
### Goal: Automate routine Linux maintenance tasks using Bash scripts and cron.
#### Today I practiced:
- Log rotation and compression
- Creating timestamped backups
- Scheduling tasks with cron
- Combining scripts into a maintenance workflow
- Logging output with timestamps
- Testing and verifying scheduled maintenance
  
# Task 1: Log Rotation Script
## Objective
Create log_rotate.sh that accepts a log directory, compresses .log files older than 7 days, deletes .gz files older than 30 days, reports the number of files processed, and exits with an error if the directory does not exist.
### Implementation Notes
- Log directory used: <!-- Add your log directory -->
- Compression age: 7 days
- Compressed-log retention: 30 days
- Handles a missing directory with an error
#### Script
File: log_rotate.sh
<!-- Paste your final log_rotate.sh script here -->
Test Commands and Output
<!-- Paste the command(s) you used to test the script -->
<!-- Paste the terminal output here -->
Screenshot
<!-- Attach your screenshot below. Save the image in this folder, for example: screenshots/day19-task1-log-rotation.png -->

 
#### What I learned
1. I learned how to check whether a log directory exists before processing its files.
2. I learned how log files can be compressed and old compressed files removed to manage disk space.
3. I learned to report the number of files processed and handle missing directories safely.

Task 2: Backup Script
Objective
Create backup.sh that accepts a source and destination directory, creates a timestamped .tar.gz archive, verifies the archive, reports its name and size, and removes backups older than 14 days.
Implementation Notes
- Source directory: /home/ubuntu/backup-practice/source
- Destination directory: /home/ubuntu/backup-practice/destination
- Backup archive format: .tar.gz
- Retention period: 14 days
Script
File: backup.sh
<!-- Paste your final backup.sh script here -->
Test Commands and Output
<!-- Paste the command(s) you used to test the script -->
<!-- Paste the terminal output here -->
Screenshot
<!-- Attach your screenshot below. Save the image in this folder, for example: screenshots/day19-task2-backup.png -->

 
What I learned
1. I learned how to pass source and destination directories as script arguments.
2. I learned to create timestamped .tar.gz archives and verify their contents.
3. I learned how backup retention policies can remove old archives automatically.
Task 3: Scheduling with Cron
Objective
Understand cron syntax and schedule scripts to run automatically.
Key Concepts
Field	Meaning	Allowed values
Minute	Minute of the hour	0–59
Hour	Hour of the day	0–23
Day of month	Calendar day	1–31
Month	Month	1–12
Day of week	Day of week	0–7 (Sunday is 0 or 7)


Useful Commands
crontab -e   # Edit your cron jobs
crontab -l   # List your cron jobs
Cron Entry Used
<!-- Paste the cron entry you configured for this task -->
Example weekly backup schedule (use only if configured):
0 3 * * 0 /home/ubuntu/pract-script/scripts/backup.sh /home/ubuntu/backup-practice/source /home/ubuntu/backup-practice/destination >> /home/ubuntu/backup-cron.log 2>&1
Screenshot
<!-- Attach your screenshot below. Save the image in this folder, for example: screenshots/day19-task3-cron.png -->

 
What I learned
1. I learned the five fields used in a cron schedule: minute, hour, day of month, month, and day of week.
2. I practiced editing and checking scheduled jobs with crontab -e and crontab -l.
3. I learned that cron runs according to the server's time zone and needs valid script paths and permissions.
Task 4: Combined Scheduled Maintenance Script
Objective
Create maintenance.sh to:
1. Run the log rotation script.
2. Run the backup script.
3. Write output and errors to /var/log/maintenance.log with timestamps.
4. Schedule it to run daily at 1:00 AM.
Implementation Notes
- Maintenance script: /home/ubuntu/pract-script/scripts/maintenance.sh
- Maintenance log: /var/log/maintenance.log
- Log rotation path configured in the script: <!-- Confirm your actual log directory -->
- Cron schedule: daily at 1:00 AM, based on server time
Script
File: maintenance.sh
<!-- Paste your final maintenance.sh script here -->
Test Commands and Output
./maintenance.sh
tail -30 /var/log/maintenance.log
<!-- Paste the actual terminal output here -->
Cron Entry
0 1 * * * /home/ubuntu/pract-script/scripts/maintenance.sh
Screenshot
<!-- Attach your screenshot below. Save the image in this folder, for example: screenshots/day19-task4-maintenance.png -->

 
What I learned
1. I learned how to combine separate log-rotation and backup scripts into one maintenance workflow.
2. I learned to redirect standard output and errors to a log file and add timestamps for troubleshooting.
3. I learned to test the maintenance script manually and verify its logs before relying on the daily cron schedule.
Troubleshooting Notes
Record any issues you encountered and how you fixed them.
Issue	Cause	Fix
date: unbound variable	<!-- Add the cause you found -->	<!-- Add the fix you applied -->
Cron entry not visible in crontab -l	<!-- Add what happened -->	<!-- Add the fix -->


Key Takeaways
1. Bash scripts can automate repeatable maintenance work.
2. Timestamped backups and retention policies support recovery and disk-space management.
3. Cron schedules recurring jobs using five time fields.
4. Centralized timestamped logs help troubleshoot automation.
5. Always test scripts manually before relying on cron.

