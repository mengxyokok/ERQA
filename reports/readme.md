# ERQA Web Report Service

This directory contains the static ERQA evaluation report pages and the assets used by those pages.

## Current Directory Layout

```text
/home/mxy/robot/eval/erqa/
  outputs/
    <model_or_run>/
      *.jsonl
      *.summary.json

  reports/
    index.html                         # dashboard / run list
    manifest.json                      # metadata for all registered runs
    erqa_report.html                   # current/latest report alias
    <run_id>.html                      # stable report page for one run
    report_assets/
      <run_id>/
        images/
          *_thumb.jpg
          *_full.jpg
```

Current registered run:

```text
run_id: eas_qwen36_27_sglang_20260608
report: reports/eas_qwen36_27_sglang_20260608.html
assets: reports/report_assets/eas_qwen36_27_sglang_20260608/images/
```

`reports/report_assets/images/` is a legacy compatibility directory from an earlier layout. New reports should use `reports/report_assets/<run_id>/...`.

## URLs

The web service should be served from the ERQA project root, not from `reports/` directly.

```text
Project root: /home/mxy/robot/eval/erqa
Service URL:  http://10.0.16.23:8899/
Dashboard:    http://10.0.16.23:8899/reports/index.html
Current run:  http://10.0.16.23:8899/reports/eas_qwen36_27_sglang_20260608.html
Latest alias: http://10.0.16.23:8899/reports/erqa_report.html
```

## Start the Service

Run this on the remote machine:

```bash
cd /home/mxy/robot/eval/erqa
nohup python3 -m http.server 8899 --bind 0.0.0.0 --directory /home/mxy/robot/eval/erqa > /tmp/erqa_report_http.log 2>&1 &
echo $! > /tmp/erqa_report_http.pid
```

The important part is `--directory /home/mxy/robot/eval/erqa`. This keeps URLs like `/reports/index.html` and `/reports/report_assets/...` valid.

## Check Status

```bash
cat /tmp/erqa_report_http.pid
ps -p $(cat /tmp/erqa_report_http.pid) -o pid,cmd
curl -I http://127.0.0.1:8899/reports/index.html
```

From another machine on the LAN:

```bash
curl -I http://10.0.16.23:8899/reports/index.html
```

## Stop the Service

```bash
kill $(cat /tmp/erqa_report_http.pid)
```

If the PID file is stale, find the process manually:

```bash
ps aux | grep 'python3 -m http.server 8899'
```

## Restart the Service

```bash
kill $(cat /tmp/erqa_report_http.pid)
cd /home/mxy/robot/eval/erqa
nohup python3 -m http.server 8899 --bind 0.0.0.0 --directory /home/mxy/robot/eval/erqa > /tmp/erqa_report_http.log 2>&1 &
echo $! > /tmp/erqa_report_http.pid
```

## Add a New Evaluation Run

For a new model or a new evaluation date:

1. Save raw outputs under `outputs/<model_or_run>/`.
2. Generate one stable report page as `reports/<run_id>.html`.
3. Put image assets under `reports/report_assets/<run_id>/`.
4. Append the run metadata to `reports/manifest.json`.
5. Regenerate or update `reports/index.html` so the dashboard lists the new run.

Recommended run id format:

```text
<model_name>_<yyyymmdd>
```

Example:

```text
reports/qwen_next_model_20260609.html
reports/report_assets/qwen_next_model_20260609/images/
```

## Current Service Notes

The current service was started with Python's built-in static HTTP server. It does not require Flask, Node, nginx, or any extra web framework. It only serves files from `/home/mxy/robot/eval/erqa`.

The service log is here:

```text
/tmp/erqa_report_http.log
```

The PID file is here:

```text
/tmp/erqa_report_http.pid
```
