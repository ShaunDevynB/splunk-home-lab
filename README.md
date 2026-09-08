# Splunk Home Lab — macOS Security Log Analysis

## Project Overview
This project is a self-built Splunk home lab, deployed locally in Docker on a MacBook Pro (Apple Silicon), used to ingest, index, and analyze real macOS system logs. The goal was to gain hands-on experience with Splunk's core workflow — data ingestion, search (SPL), field extraction, and dashboard creation — while working through real-world setup obstacles (platform architecture mismatches, licensing requirements, and log volume management).

The lab ingests macOS's built-in unified log data, filters it down to security-relevant events (authentication, login, and failure activity), and surfaces findings through a two-panel Splunk dashboard.

## Architecture Diagram

```
macOS Unified Log (log show)
        |
        v
Filtered log export (.txt file, auth/login/failed events)
        |
        v
Docker container: splunk/splunk:latest (linux/amd64, run under emulation on M1)
        |
        v
Splunk Enterprise (Web UI @ localhost:8000)
        |
        v
SPL Search & Field Extraction (rex, stats, timechart)
        |
        v
Dashboard: "Mac Security Home Lab"
   -> Panel 1: Top Failing Processes (bar chart)
   -> Panel 2: Failures Over Time (line chart)
```

## Tools Used
- **Splunk Enterprise** (via official Docker image) — log indexing, search, and dashboarding
- **Docker Desktop** — containerized deployment on macOS
- **macOS `log` command** — native system log extraction
- **SPL (Search Processing Language)** — `rex`, `stats`, `timechart`, keyword filtering
- **Terminal / Bash** — command execution and file management
- **MacBook Pro (Apple Silicon / M1)** — host environment

## What I Did — Step by Step

1. **Environment setup** — Confirmed Docker Desktop was running locally before attempting any container operations.
2. **Pulled the Splunk image** — Initial `docker pull` failed with a `no matching manifest for linux/arm64/v8` error, since Splunk's official image isn't built for Apple Silicon. Resolved by explicitly pulling the `linux/amd64` image and running it under Docker's built-in emulation.
3. **Launched the Splunk container** — First launch attempt failed due to a missing `SPLUNK_GENERAL_TERMS` acceptance flag (a newer Splunk licensing requirement). Removed the failed container and relaunched with both required license flags (`SPLUNK_START_ARGS`, `SPLUNK_GENERAL_TERMS`) accepted.
4. **Verified container health** — Used `docker logs -f` to confirm Splunk's internal Ansible provisioning completed successfully before attempting to log in.
5. **Logged into the Splunk Web UI** at `localhost:8000` and confirmed the internal `_internal` index was searchable, validating the instance was indexing correctly.
6. **Extracted real log data** — Used macOS's `log show` command to pull system logs. An initial 1-hour full export was too large (662MB) for a lab use case, so the extraction was narrowed to a 5-minute window filtered to authentication/login/failure events using `log show --predicate`.
7. **Uploaded the filtered log file into Splunk** — Verified timestamp parsing and event breaks in the "Add Data" preview screen before committing the upload. 277 events were successfully indexed.
8. **Built SPL searches to find security-relevant patterns** — Used keyword filtering (`"failed"`) combined with `rex` for field extraction, pulling out the process name responsible for each failure event from the raw log text.
9. **Aggregated and ranked findings** — Used `stats count by process_name | sort -count` to identify which macOS processes generated the most failure events.
10. **Built a two-panel dashboard** — "Top Failing Processes" (bar chart) and "Failures Over Time" (`timechart span=1m count`), saved as a persistent Splunk dashboard.
11. **Documented the project**, including troubleshooting steps, in this README.

## Key Findings
- Of 277 indexed log events, `coreaudiod` (macOS's core audio daemon) was the leading source of failure events, accounting for 85 occurrences — more than triple the next-highest source.
- `kernel` (45) and `Safari` (29) were the second and third most frequent sources of failure events, respectively.
- The "Failures Over Time" panel revealed that nearly all failure activity was concentrated in a single short burst rather than spread evenly across the capture window — consistent with the timing of the log export itself, and a useful reminder that log volume/timing context matters when interpreting security data.

## Screenshots
See `/screenshots` folder:
1. `01_docker_setup_troubleshooting.png` — Docker pull/run process, including the ARM64 and licensing errors encountered and resolved
2. `02_splunk_login_success.png` — Splunk Web UI running locally
3. `03_first_search_internal_logs.png` — First successful SPL search against Splunk's internal index
4. `04_data_upload_preview.png` — Timestamp/event parsing verification during data upload
5. `05_uploaded_data_confirmed.png` — Confirmed 277 real macOS log events indexed and searchable
6. `06_top_failing_processes_table.png` — SPL search results ranking failing processes
7. `07_final_dashboard.png` — Completed two-panel Splunk dashboard

## Skills Demonstrated
- Docker containerization and troubleshooting (platform architecture mismatches, licensing/config errors)
- Splunk Enterprise deployment and administration
- Log data ingestion and source type/timestamp validation
- SPL (Search Processing Language): field extraction (`rex`), aggregation (`stats`), time-series analysis (`timechart`)
- macOS system log analysis and filtering
- Security-relevant data analysis and pattern identification
- Dashboard design and data visualization
- Technical documentation

## Resume Bullet
Built a local Splunk Enterprise home lab in Docker to ingest and analyze macOS system logs, using SPL to identify and visualize top failure sources across 277+ indexed events, and resolved platform compatibility and licensing configuration issues during deployment.
