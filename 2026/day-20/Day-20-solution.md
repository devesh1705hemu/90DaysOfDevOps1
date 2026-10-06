# Day 20 – Bash Scripting Challenge: Log Analyzer and Report Generator

## Overview

**Challenge:** Build a Bash script that analyzes application logs, identifies errors and critical events, finds common error messages, and generates a summary report.

### Purpose of This Challenge

In real-world DevOps, applications and servers generate logs containing information, warnings, errors, and critical failures. Manually checking large log files is time-consuming.

The goal of this challenge is to automate log analysis using Bash and Linux commands. This helps engineers investigate incidents, identify recurring failures, and generate reports for monitoring and troubleshooting.

### Learning Objectives

- Validate command-line arguments and file paths.
- Count error-related log entries.
- Identify critical events with line numbers.
- Find the five most common error messages.
- Generate a dated summary report automatically.

---

## Task 1: Input and Validation

### Purpose

Input validation ensures that a script receives the correct arguments and that the specified file exists before processing begins. This prevents unnecessary failures and makes automation more reliable.

### Requirements

1. Accept a log file path as a command-line argument.
2. Display an error if no argument is provided.
3. Display an error if the file does not exist.

### Code

```bash
if [ "$#" -eq 0 ]; then
    echo "Error: No log file path provided."
    echo "Usage: $0 <log_file_path>"
    exit 1
fi

LOG_FILE="$1"

if [ ! -f "$LOG_FILE" ]; then
    echo "Error: File '$LOG_FILE' does not exist."
    exit 1
fi

echo "Input validated: $LOG_FILE"
```

### Test Commands

```bash
./log_report.sh
./log_report.sh /tmp/missing.log
./log_report.sh app.log
```

### Screenshot

**Attach screenshot here:** Input validation code and terminal output.

`![Task 1 Screenshot](screenshots/day20-task1.png)`

---

## Task 2: Error Count

### Purpose

Counting error-related lines gives engineers a quick overview of failures in an application. This can help identify unusual error activity and prepare monitoring summaries.

### Requirements

1. Count lines containing `ERROR` or `Failed`.
2. Print the total count.
3. Count each matching line once, even if both keywords occur on the same line.

### Code

```bash
ERROR_COUNT=$(grep -Ec 'ERROR|Failed' "$LOG_FILE" || true)
echo "Total error lines: $ERROR_COUNT"
```

### Screenshot

**Attach screenshot here:** Error count output.

`![Task 2 Screenshot](screenshots/day20-task2.png)`

---

## Task 3: Critical Events

### Purpose

Critical events can indicate serious problems such as database outages, low disk space, or unavailable services. Line numbers make it easier to locate the exact event in a large log file.

### Requirements

1. Search for lines containing `CRITICAL`.
2. Print each matching line with its line number.
3. Display a message if no critical events are found.

### Code

```bash
echo "--- Critical Events ---"

if grep -q 'CRITICAL' "$LOG_FILE"; then
    grep -n 'CRITICAL' "$LOG_FILE" |
        while IFS= read -r match; do
            line_number="${match%%:*}"
            content="${match#*:}"
            echo "Line ${line_number}:${content}"
        done
else
    echo "No critical events found."
fi
```

### Example Output

```text
--- Critical Events ---
Line 84: 2025-07-29 10:15:23 CRITICAL Disk space below threshold
Line 217: 2025-07-29 14:32:01 CRITICAL Database connection lost
```

### Screenshot

**Attach screenshot here:** Critical events with line numbers.

`![Task 3 Screenshot](screenshots/day20-task3.png)`

---

## Task 4: Top 5 Error Messages

### Purpose

Repeated error messages can reveal the underlying cause of an incident. Grouping and counting these messages helps engineers prioritize the failures that occur most frequently.

### Requirements

1. Extract lines containing `ERROR`.
2. Remove timestamps and the `ERROR` prefix.
3. Count repeated messages.
4. Sort by occurrence count in descending order.
5. Display up to five messages.

### Code

```bash
echo "--- Top 5 Error Messages ---"

if grep -q 'ERROR' "$LOG_FILE"; then
    grep 'ERROR' "$LOG_FILE" |
        sed -E 's/^.*ERROR[[:space:]]*//' |
        sort |
        uniq -c |
        sort -nr |
        head -n 5
else
    echo "No ERROR messages found."
fi
```

### Example Output

```text
--- Top 5 Error Messages ---
45 Connection timed out
32 File not found
28 Permission denied
15 Disk I/O error
9  Out of memory
```

### Screenshot

**Attach screenshot here:** Top five error messages and counts.

`![Task 4 Screenshot](screenshots/day20-task4.png)`

---

## Task 5: Summary Report

### Purpose

A summary report saves important findings in a dated text file. DevOps engineers can use these reports for incident investigations, daily maintenance, troubleshooting, and historical comparisons.

### Requirements

The report must include:

1. Date and time of analysis.
2. Log file name and path.
3. Total lines processed.
4. Total lines containing `ERROR` or `Failed`.
5. Top five error messages with occurrence counts.
6. Critical events with line numbers.

### Complete Script

Save the complete implementation as `log_report.sh`. Combine the validation, error count, critical-event search, top-five analysis, and report-generation sections from Tasks 1–4 into one script.

The report filename should follow this format:

```text
log_report_YYYY-MM-DD.txt
```

For example:

```text
log_report_2026-10-06.txt
```

### Run and Verify

```bash
chmod +x log_report.sh
bash -n log_report.sh
./log_report.sh app.log
cat "log_report_$(date +%Y-%m-%d).txt"
```

### Sample Console Output

```text
Report generated successfully: log_report_2026-10-06.txt
```

### Example Report

```text
========================================
          LOG SUMMARY REPORT
========================================
Date of Analysis: 2026-10-06 10:00:00
Log File Name: app.log
Log File Path: app.log
Total Lines Processed: 9
Total Error Count (ERROR or Failed): 6

--- Top 5 Error Messages ---
3 Connection timed out
1 Permission denied
1 File not found

--- Critical Events ---
Line 6:2026-10-06 09:09:00 CRITICAL Disk space below threshold
Line 9:2026-10-06 09:12:00 CRITICAL Database connection lost

=============== END OF REPORT ===============
```

*Note: The report timestamp is illustrative. Your script will use the actual system date and time. The overall error count includes both `ERROR` and `Failed`, while the top-five section groups lines containing `ERROR` only.*

### Screenshot

**Attach screenshot here:** Successful script execution and report contents.

`![Task 5 Screenshot](screenshots/day20-task5.png)`

---

## Commands and Tools Used

| Command/tool | Purpose |
|---|---|
| `bash` | Runs the script. |
| `$#`, `$1` | Reads command-line arguments. |
| `grep -E` | Matches multiple patterns. |
| `grep -n` | Shows matching lines with line numbers. |
| `sed -E` | Removes timestamps and error prefixes. |
| `sort` | Groups and sorts messages. |
| `uniq -c` | Counts repeated messages. |
| `sort -nr` | Sorts counts in descending order. |
| `head -n 5` | Shows the top five results. |
| `wc -l` | Counts newline-terminated lines. |
| `basename` | Extracts a filename from a path. |
| `date` | Creates a dated report filename. |
| `bash -n` | Checks Bash syntax without executing the script. |

---

## What I Learned: 3 Key Points

1. **Input validation and error handling:** I learned to validate command-line arguments and file paths before processing logs.
2. **Linux text processing:** I used `grep`, `sed`, `sort`, `uniq`, and `head` to extract, group, and analyze log entries.
3. **Automation and reporting:** I built a reusable Bash log analyzer that generates a dated report, providing a foundation for scheduled monitoring and incident investigation.

---

