---
title: Agentfence — An eBPF Based Agent Harness
layout: default
date: 2026-10-28
parent: Blog
---
# Agentfence — An eBPF Based Agent Harness

Everyone is shipping agents now. An LLM gets a shell, a working directory, and a vague goal, and we trust it. The usual containment story is a userland wrapper that logs what the agent did, or a seccomp profile somebody copied out of a Dockerfile. That is detection after the fact at best, and the wrapper itself is just another process the agent can race, ptrace, or walk away from.

My idea: if the agent runs on my Linux machine, the kernel should be the one saying no. I want to wrap an arbitrary process and deny its policy-violating actions at the kernel level, before they land. Detection tells you something bad happened. I want prevention, with a report attached.

At NTT I worked on syscall based anomaly detection for containers, so I have spent a lot of time watching what processes do from inside the kernel. The natural next step for me is turning that telemetry into verdicts: same vantage point, but instead of scoring the behaviour, just block it.

## What it is

A Rust CLI called `agentfence` — an eBPF based policy enforcement harness. Two crates in one workspace, the standard aya layout:

- `agentfence` — the userspace CLI
- `agentfence-ebpf` — the eBPF programs, built with `cargo xtask build-ebpf`

```
agentfence run --policy policy.yaml [--report report.json] -- <command> [args...]
```

Behavior: create a cgroup (v2) at `/sys/fs/cgroup/agentfence/<session-id>`, spawn the command inside it, load the eBPF programs, stream live verdicts to the terminal, and on session end write a JSON report of all events. With `exit_on_violation: true`, kill the entire agent process tree on the first violation and exit with code 42.

Target is a single Linux machine, x86_64, kernel >= 6.1.

## The part I care about: scoping

The hard requirement I set for myself: the programs must never inspect or affect processes outside the session cgroup. Every hook starts by matching the current task's cgroup id against the session cgroup id stored in a map. Not our session? Return 0 immediately, we never existed.

Cgroup scoping also solves the child problem for free. If the agent spawns children, they inherit cgroup membership, so enforcement applies to the whole tree. No pid list bookkeeping, no races when the agent forks faster than userspace can notice.

## Policy

A YAML file, parsed with serde_yaml into an exact schema:

```yaml
name: demo
scope:
  workspace: /tmp/agentfence-demo
rules:
  fs_read_deny: ["~/.ssh/id_rsa", "~/.env"]     # glob paths, ~ expands to $HOME
  fs_write_allow: ["/tmp/agentfence-demo/**"]   # writes anywhere else are denied
  exec_allow: ["/usr/bin/env", "/bin/bash"]     # exec of anything else denied
  net_connect_allow: ["localhost:*"]            # host:port, "*" allowed for port
  exit_on_violation: true
  report: report.json
```

The policy is compiled in userspace and shipped to the kernel via BPF maps, pinned by name, so it survives and is inspectable. Path matching uses a fixed-size 256-byte path buffer with glob matching (`*`, `**`, `?`) in-kernel using a compact matcher. Bounded buffers, no dynamic allocation in the programs — the usual eBPF discipline.

## Two modes

**1) ENFORCEMENT (default if available):** BPF LSM programs attached to `file_open`, `bprm_check_security`, and `socket_connect`.

- `file_open`: path matches `fs_read_deny`, or it is a write (check the open flags for O_WRONLY/O_RDWR) outside `fs_write_allow` → return `-EPERM`.
- `bprm_check_security`: executable path not in `exec_allow` → return `-EPERM`.
- `socket_connect`: read the sockaddr; if AF_INET/AF_INET6 and `ip:port` not in `net_connect_allow` → return `-EPERM`.

The important property: the denial happens inside the kernel, in the syscall path. The agent cannot outsmart, race, or uninstall it from user space.

**2) OBSERVE+KILL (fallback):** if `/sys/kernel/security/lsm` does not contain `bpf` (BPF LSM not enabled), print a warning and fall back to raw tracepoints and kprobes — `sys_enter_openat`, `sys_enter_execve`, `connect` — with the same cgroup scoping. Events go to userspace over a ring buffer, userspace re-checks the policy, and on violation SIGKILLs the offending PID (and with `exit_on_violation`, the whole process group) and logs it.

Fallback is strictly weaker: the syscall may have already happened by the time userspace reacts. That is the honest tradeoff — observe and kill after the fact vs. deny in the moment. The mode ends up in the report (`"mode": "lsm"` or `"tracepoint"`) so a demo never lies about which one ran.

## Live output and the report

All events, allowed-with-note and denied, go to a ring buffer consumed by the CLI and printed live, colorized: green ALLOW lines only for notable categories (exec, connect), red DENIED lines with the rule name, path or syscall, and PID. No spamming every file read, just the ones you would want to audit.

On session end (agent exit or Ctrl-C, with clean unload of programs and cgroup removal):

```json
{
  "session_id": "...",
  "mode": "lsm",
  "started": "...",
  "ended": "...",
  "events": [
    { "ts": "...", "kind": "file_open", "pid": 4242,
      "path_or_dst": "/home/me/.ssh/id_rsa",
      "verdict": "deny", "rule": "fs_read_deny" }
  ]
}
```

## Demo

`make demo` runs four scenarios against a mock agent and asserts the outcomes:

- `good` — touch a file in the workspace, run `/usr/bin/env python3 -c "print('ok')"`. Succeeds.
- `steal-keys` — `cat ~/.ssh/id_rsa`. Must be BLOCKED.
- `exfil` — open a socket to the outside world. Must be BLOCKED.
- `tamper` — `echo pwned > /etc/hostname-test`. Must be BLOCKED.

Three DENIED lines, exit code 42, PASS/FAIL summary. If the harness lets an agent read my SSH keys, the demo fails. That is the whole product statement in one make target.

## Limitations, honestly

- Needs root, and BPF LSM needs `lsm=bpf` on the kernel command line (check `/sys/kernel/security/lsm`). 
- Fallback mode can only kill after the fact.
- Glob matching in-kernel on fixed 256-byte buffers means paths longer than that are truncated, not matched.
- Single machine, x86_64, kernel >= 6.1. No namespaces, no containers yet.

Full details, kernel requirements, and build instructions will live in the README once the repo is up. Will write more when it actually blocks something.
