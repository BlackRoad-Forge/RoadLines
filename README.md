# RoadLines

**OS Model Checker for BlackRoad Infrastructure**

RoadLines is a formal verification tool for operating system models. It exhaustively explores all possible execution states — concurrency bugs, crash recovery, race conditions, TOCTTOU — and proves correctness or finds violations.

Zero dependencies. 500 lines of Python. Runs anywhere.

## Usage

```bash
# Run a model (random execution trace)
python3 roadlines.py --run examples/hello.py

# Model-check (exhaustive state space exploration)
python3 roadlines.py --check examples/hello.py

# Find all possible outputs
python3 roadlines.py --check examples/hello.py | grep stdout | sort | uniq

# Interactive state explorer
python3 roadlines.py --check examples/hello.py | python3 -m vis
```

## Writing Models

RoadLines models are Python programs with system calls. Entry point is `main()`:

```python
def main():
    pid = sys_fork()
    sys_sched()
    if pid == 0:
        sys_write('World\n')
    else:
        sys_write('Hello\n')
```

## System Calls

| Call | Behavior |
|------|----------|
| `sys_fork()` | Clone current thread with copied heap |
| `sys_spawn(f, xs)` | Spawn heap-sharing thread executing `f(xs)` |
| `sys_write(xs)` | Write string to shared console |
| `sys_bread(k)` | Read block `k` from storage |
| `sys_bwrite(k, v)` | Write `(k, v)` to storage buffer |
| `sys_sync()` | Persist all buffered writes |
| `sys_sched()` | Non-deterministic context switch |
| `sys_choose(xs)` | Non-deterministic choice from `xs` |
| `sys_crash()` | Non-deterministic crash (partial persist) |

## Examples

| File | What it tests |
|------|---------------|
| `hello.py` | Fork + schedule ordering |
| `parallel-inc.py` | Race condition on shared counter |
| `cond-var.py` | Condition variable correctness |
| `fork-buf.py` | Fork with buffered I/O |
| `tocttou.py` | Time-of-check/time-of-use bug |
| `fs-crash.py` | File system crash consistency |
| `xv6-log.py` | xv6 log-based crash recovery |

## How It Works

The OS model is a state machine driven by system calls. Each system call returns all possible non-deterministic choices. The checker does BFS over the full state space, hashing states to avoid revisiting.

Programs are rewritten at the AST level — `sys_xxx()` calls become `yield` expressions, turning functions into generators. The OS can then freeze, serialize, and replay any thread at any point.

## Requirements

Python 3.10+ (tested on 3.10, 3.11, 3.12, 3.14)

Visualization requires `jinja2` and `pygments`:
```bash
pip install jinja2 pygments
```

---

Copyright (c) 2024-2026 BlackRoad OS, Inc. All Rights Reserved.

Based on research from USENIX ATC'23 Paper #202.
