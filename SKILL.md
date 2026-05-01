---
name: skill-health
description: Audit installed Hermes Agent skills for dependency availability, API connectivity, version tracking, and overall health with actionable fix suggestions
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  tags:
    - devops
    - health-check
    - audit
    - dependencies
    - monitoring
    - cron
  related_skills:
    - hermes-agent
---

# Skill Health

Audit all installed Hermes Agent skills for dependency availability, API connectivity, version status, and overall health. Generates a report card with actionable fix suggestions and supports scheduled cron-based monitoring.

## Overview

Skill Health proactively monitors your Hermes Agent skill ecosystem. It checks that required dependencies (Python packages, Node tools, CLI binaries) are installed, APIs are responding, skill versions are current, and configurations are valid. Each skill receives a health status: ✅ healthy, ⚠️ degraded, or ❌ broken.

## Commands

### `/skill-health [skill-name]`
Run a health audit. If a skill name is provided, audit only that skill. Otherwise, audit all installed skills.

**Examples:**
```
/skill-health
/skill-health speed-reader
/skill-health social-poster meme-generator
```

### `/skill-health-full`
Run a comprehensive audit including API connectivity tests and version update checks. Takes longer but provides the most thorough report.

### `/skill-health-deps [skill-name]`
Check only dependencies for a specific skill or all skills.

**Example:**
```
/skill-health-deps speed-reader
```

### `/skill-health-apis [skill-name]`
Test API connectivity for skills that depend on external APIs.

**Example:**
```
/skill-health-apis xurl
```

### `/skill-health-fix [skill-name]`
Get auto-generated fix suggestions for a degraded or broken skill.

**Example:**
```
/skill-health-fix meme-generator
```

### `/skill-health-report`
Generate a full health report card and save it to Obsidian.

### `/skill-health-schedule <cron_expression>`
Set up a cron-based health check schedule.

**Examples:**
```
/skill-health-schedule "0 9 * * 1"     # Every Monday 9AM
/skill-health-schedule "0 */6 * * *"    # Every 6 hours
```

## Health Status Levels

| Status | Icon | Meaning |
|--------|------|---------|
| Healthy | ✅ | All dependencies met, APIs responding, config valid |
| Degraded | ⚠️ | Non-critical issues: optional deps missing, slow APIs, minor version drift |
| Broken | ❌ | Critical failures: required deps missing, APIs down, config invalid |

## Audit Checks

### 1. Dependency Check

Each skill declares its dependencies in the SKILL.md frontmatter or a `depends.json` file. The health checker verifies:

**Python Packages:**
```bash
python3 -c "import pillow; print(pillow.__version__)"
```

**Node Tools:**
```bash
npm list -g browser-sync 2>/dev/null
```

**CLI Binaries:**
```bash
which jq curl ffmpeg
```

**Skill Dependencies:**
```bash
# Check that required sister skills are installed
ls ~/.hermes/skills/productivity/speed-reader/SKILL.md
ls ~/.hermes/skills/browser/
```

### 2. API Connectivity Tests

For skills that interact with external APIs, perform health checks:

```bash
# X/Twitter API via xurl skill
curl -s -o /dev/null -w "%{http_code}" https://api.twitter.com/2/tweets

# Telegram Bot API
curl -s -o /dev/null -w "%{http_code}" https://api.telegram.org/bot${TELEGRAM_TOKEN}/getMe

# Generic health endpoint
curl -sf https://api.example.com/health || echo "API DOWN"
```

### 3. Version Tracking

Compare installed skill versions against a known registry:

```json
{
  "speed-reader": {"installed": "1.0.0", "latest": "1.2.1", "update_available": true},
  "social-poster": {"installed": "1.0.0", "latest": "1.0.0", "update_available": false}
}
```

### 4. Configuration Validation

Verify each skill's configuration:
- Required config keys are present
- File paths exist (e.g., Obsidian vault path)
- API tokens are set (without exposing values)
- Numeric values are within valid ranges

## Report Card Format

```
🏥 Hermes Agent Skill Health Report
═══════════════════════════════════════
Generated: 2025-01-15 09:00 UTC

┌─────────────────────┬────────┬───────────────────────────────┐
│ Skill               │ Status │ Issues                        │
├─────────────────────┼────────┼───────────────────────────────┤
│ speed-reader        │ ✅      │ None                          │
│ social-poster       │ ⚠️      │ Twitter API slow (2.3s)       │
│ meme-generator      │ ❌      │ Pillow not installed          │
│ skill-health        │ ✅      │ None                          │
│ xurl                │ ✅      │ None                          │
│ ocr-and-documents   │ ⚠️      │ Tesseract version outdated    │
│ browser             │ ✅      │ None                          │
│ obsidian-vault      │ ✅      │ None                          │
└─────────────────────┴────────┴───────────────────────────────┘

Summary: 5 ✅  2 ⚠️  1 ❌
```

### Detailed Skill Report

```
❌ meme-generator v1.0.0

DEPENDENCIES:
  ❌ Python package: pillow — NOT INSTALLED
  ✅ Python package: textwrap — available (stdlib)
  ⚠️ Font: Impact (/usr/share/fonts/impact.ttf) — NOT FOUND (fallback available)

APIS:
  N/A — No external APIs

CONFIG:
  ✅ vault_path: ~/obsidian-vault — EXISTS
  ✅ default_format: png — VALID

SUGGESTED FIXES:
  → Install Pillow: pip3 install Pillow
  → Install Impact font: sudo apt install fonts-freefont-ttf

Last checked: 2025-01-15 09:00 UTC
```

## Dependency Schema

Each skill can declare dependencies in its SKILL.md or a companion `depends.json`:

```json
{
  "dependencies": {
    "python": ["pillow>=9.0", "requests>=2.28"],
    "node": [],
    "cli": ["curl", "jq"],
    "fonts": ["impact"],
    "skills": ["obsidian-vault", "browser"],
    "apis": [
      {
        "name": "Twitter API",
        "url": "https://api.twitter.com/2/tweets",
        "method": "GET",
        "expected_status": 401,
        "env_var": "TWITTER_TOKEN"
      }
    ],
    "config": [
      {
        "key": "vault_path",
        "type": "path",
        "required": true,
        "must_exist": true
      },
      {
        "key": "default_format",
        "type": "enum",
        "values": ["png", "jpg", "webp"],
        "required": false
      }
    ]
  }
}
```

## Auto-Fix Suggestions

When a skill is degraded or broken, `/skill-health-fix` generates targeted fix commands:

```
🔧 Suggested fixes for meme-generator:

1. [CRITICAL] Install missing Python package:
   $ pip3 install Pillow

2. [RECOMMENDED] Install Impact font for classic meme style:
   $ sudo apt install fonts-freefont-ttf
   # OR download from: https://example.com/fonts

3. [OPTIONAL] Install ImageMagick for additional format support:
   $ sudo apt install imagemagick

Run all critical fixes:
  $ pip3 install Pillow && sudo apt install fonts-freefont-ttf
```

## Scheduled Health Checks

### Cron Configuration

Health checks can be scheduled via cron:

```bash
# Add to crontab
0 9 * * 1  hermes-agent skill-health-full > ~/.hermes/logs/health-$(date +\%Y\%m\%d).log
```

### Built-in Scheduling

```
/skill-health-schedule "0 9 * * 1"
→ ✅ Health check scheduled for every Monday at 9:00 AM
→ 📄 Schedule saved to: ~/.hermes/skills/devops/skill-health/schedule.json
```

Schedule is stored in `schedule.json`:

```json
{
  "schedules": [
    {
      "id": "schedule-001",
      "cron": "0 9 * * 1",
      "type": "full",
      "enabled": true,
      "last_run": "2025-01-13T09:00:00Z",
      "next_run": "2025-01-20T09:00:00Z",
      "notify_on_failure": true,
      "notify_channel": "telegram"
    }
  ]
}
```

### Alert Notifications

When a health check detects a broken skill, it can notify via Telegram:

```
🚨 Skill Health Alert

❌ meme-generator is BROKEN
  Missing dependency: Pillow

Run /skill-health-fix meme-generator for fix suggestions.
```

## Implementation

### Dependency Discovery

The health checker discovers skills and their dependencies by:

1. Scanning `~/.hermes/skills/` for all SKILL.md files
2. Parsing YAML frontmatter for skill metadata
3. Loading companion `depends.json` if present
4. Checking SKILL.md body for dependency declarations in code examples

```python
import os
import yaml
import json
import subprocess

SKILLS_DIR = os.path.expanduser("~/.hermes/skills")

def discover_skills():
    """Find all installed skills."""
    skills = {}
    for root, dirs, files in os.walk(SKILLS_DIR):
        if "SKILL.md" in files:
            skill_path = os.path.join(root, "SKILL.md")
            with open(skill_path) as f:
                # Parse YAML frontmatter
                content = f.read()
                if content.startswith("---"):
                    _, frontmatter, _ = content.split("---", 2)
                    meta = yaml.safe_load(frontmatter)
                    name = meta.get("name", os.path.basename(root))
                    version = meta.get("version", "0.0.0")
                    skills[name] = {
                        "path": root,
                        "version": version,
                        "meta": meta
                    }
    return skills

def check_python_package(package):
    """Check if a Python package is installed."""
    try:
        mod = package.replace("-", "_").split(">=")[0].split("==")[0]
        result = subprocess.run(
            ["python3", "-c", f"import {mod}; print({mod}.__version__)"],
            capture_output=True, text=True, timeout=10
        )
        if result.returncode == 0:
            return {"installed": True, "version": result.stdout.strip()}
        return {"installed": False}
    except Exception as e:
        return {"installed": False, "error": str(e)}

def check_cli_binary(name):
    """Check if a CLI binary is available."""
    result = subprocess.run(
        ["which", name], capture_output=True, text=True
    )
    if result.returncode == 0:
        return {"installed": True, "path": result.stdout.strip()}
    return {"installed": False}

def check_api_health(url, expected_status=200, timeout=5):
    """Check if an API endpoint responds."""
    try:
        result = subprocess.run(
            ["curl", "-s", "-o", "/dev/null", "-w", "%{http_code}",
             "--max-time", str(timeout), url],
            capture_output=True, text=True, timeout=timeout + 2
        )
        status = int(result.stdout.strip())
        return {
            "reachable": True,
            "status": status,
            "healthy": status == expected_status
        }
    except Exception as e:
        return {"reachable": False, "error": str(e)}
```

## Pitfalls

1. **Environment differences** — Dependencies available in one shell session may not be in another (e.g., virtualenv not activated). The health checker uses the system Python PATH.

2. **API authentication** — Many APIs return 401/403 without proper auth. The health checker expects this for protected endpoints and considers them "reachable" if they respond at all.

3. **Network-dependent checks** — API connectivity tests require internet access. Failures may indicate network issues rather than API problems. The report distinguishes between "unreachable" and "responding with errors."

4. **Version registry availability** — Checking for updates requires access to the skill registry. If the registry is unreachable, version checks are skipped with a warning.

5. **Cron environment** — Cron runs in a limited environment. PATH may not include all binary locations. Always use full paths in cron jobs (e.g., `/usr/bin/python3` instead of `python3`).

6. **Permission issues** — Some checks (installing packages, system fonts) require `sudo`. The health checker identifies these but doesn't auto-execute privileged commands.

7. **Circular skill dependencies** — If skill A depends on skill B which depends on skill A, the checker handles this gracefully by skipping already-visited skills.

8. **Stale schedule entries** — If a skill is removed but its cron schedule remains, the scheduler will produce warnings. Use `/skill-health-schedule` with `cleanup` to remove orphaned entries.

9. **Rate limiting** — API health checks that hit rate-limited endpoints should use HEAD requests or cache results for at least 5 minutes to avoid being blocked.

10. **False positives** — A skill marked ❌ broken due to an optional dependency is actually ⚠️ degraded. The checker uses the `required` flag in depends.json to distinguish critical from optional.

## Verification Steps

1. **Check skill is loaded:**
   ```
   /skill-health skill-health
   ```
   Should return a healthy status for the skill-health skill itself.

2. **Run full audit:**
   ```
   /skill-health
   ```
   Should scan all skills in `~/.hermes/skills/` and produce a report card.

3. **Test dependency checking:**
   ```
   /skill-health-deps meme-generator
   ```
   Should check for Pillow, font availability, and other declared dependencies.

4. **Test API connectivity:**
   ```
   /skill-health-apis
   ```
   Should test all external API endpoints declared by installed skills.

5. **Test fix suggestions:**
   Intentionally break a skill (e.g., uninstall a dependency), then run:
   ```
   /skill-health-fix <skill-name>
   ```
   Should provide actionable installation commands.

6. **Generate and save report:**
   ```
   /skill-health-report
   ```
   Should create a markdown report in `~/obsidian-vault/3-Resources/Health Reports/`.

7. **Test cron scheduling:**
   ```
   /skill-health-schedule "0 9 * * 1"
   ```
   Verify the schedule is saved and the next run time is displayed.

8. **Test notification delivery:**
   Configure Telegram notifications and verify that health alerts are delivered when a skill is broken.

9. **Test edge cases:**
   - Audit with no skills installed → should report empty but not error
   - Audit a skill with no dependencies → should report ✅ healthy
   - Audit a skill with missing SKILL.md → should report ⚠️ degraded