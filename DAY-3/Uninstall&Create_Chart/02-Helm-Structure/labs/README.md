# Day 3 — Chapter 2: Lab — Scaffold a Chart with `helm create`

**Difficulty:** Beginner  
**Estimated time:** 15–25 minutes  
**Goal:** Create a chart named `webapp`, remove its generated test directory, and save a list of template files.

## What you will create

```text
/root/work/webapp/
├── Chart.yaml
├── values.yaml
├── templates/
└── ...
```

And this answer file:

```text
/root/answers/webapp-templates.txt
```

## Step 1 — Check Helm is installed

Run:

```bash
helm version
```

If this command fails, fix your Helm installation before continuing.

## Step 2 — Check the working directory

Run:

```bash
mkdir -p /root/work
cd /root/work
pwd
```

The last command should print:

```text
/root/work
```

## Step 3 — Create the chart

Run:

```bash
helm create webapp
```

This creates the chart scaffold at `/root/work/webapp`.

Check that the directory exists:

```bash
ls -la /root/work/webapp
```

You should see `Chart.yaml`, `values.yaml`, `templates/`, and other generated files.

> If `/root/work/webapp` already exists, inspect it before proceeding. Do not overwrite work you need to keep.

## Step 4 — Inspect the chart metadata and values

Run:

```bash
cat /root/work/webapp/Chart.yaml
cat /root/work/webapp/values.yaml
```

Look for the chart name and version in `Chart.yaml`. In `values.yaml`, look for settings such as the image, service, and replica configuration.

## Step 5 — Remove the generated tests directory

The lab specifically asks you to remove `templates/tests` completely.

First check that it exists:

```bash
ls -la /root/work/webapp/templates/tests
```

Then remove only that directory:

```bash
rm -rf /root/work/webapp/templates/tests
```

Verify that it is gone:

```bash
if [ -d /root/work/webapp/templates/tests ]; then
  echo "ERROR: templates/tests still exists"
else
  echo "OK: templates/tests has been removed"
fi
```

Be careful with `rm -rf`: verify the exact path before running it.

## Step 6 — Save the list of template files

Create the answer directory:

```bash
mkdir -p /root/answers
```

Save the file list:

```bash
find /root/work/webapp/templates -maxdepth 2 -type f | sort \
  > /root/answers/webapp-templates.txt
```

Read the saved file:

```bash
cat /root/answers/webapp-templates.txt
```

The file should list the remaining files under `templates/`, one path per line. Since the command runs `find` on the absolute path, the entries will use absolute paths.

## Step 7 — Validate the chart

Run:

```bash
helm lint /root/work/webapp
```

This checks the chart for common problems. If lint reports an issue, read the message and note it; do not delete extra files unless the task asks you to.

Optionally render the chart:

```bash
helm template webapp /root/work/webapp
```

This renders YAML locally and does not install anything in Kubernetes.

## Step 8 — Final verification

Run:

```bash
test -f /root/work/webapp/Chart.yaml && echo "Chart.yaml exists"
test -f /root/work/webapp/values.yaml && echo "values.yaml exists"
test ! -d /root/work/webapp/templates/tests && echo "tests directory removed"
test -s /root/answers/webapp-templates.txt && echo "template list saved"
```

### Acceptance criteria

- [ ] Chart exists at `/root/work/webapp`.
- [ ] `Chart.yaml` and `values.yaml` exist and have been reviewed.
- [ ] `/root/work/webapp/templates/tests` has been removed.
- [ ] `/root/answers/webapp-templates.txt` contains the remaining template file list.
- [ ] You understand the purpose of the main chart files.

## Troubleshooting

- **`helm create` says the directory exists:** Inspect the existing chart first. Use a clean lab environment or back up the directory before changing it.
- **`templates/tests` does not exist:** Check the exact path with `find /root/work/webapp/templates -maxdepth 2 -type d`. Some Helm versions or scaffolds may differ; only remove the directory if it exists.
- **The answer file is empty:** Confirm `templates/` contains files and rerun the `find` command.
- **`helm lint` reports warnings:** Read the details. A warning is not necessarily the same as a fatal chart error.

## Reflection questions

1. What makes a directory a Helm chart?
2. What is the purpose of `Chart.yaml`?
3. How is `values.yaml` different from `templates/`?
4. What does `helm create` generate?
5. Why is `helm template` useful before installation?
