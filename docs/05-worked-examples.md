# 05 — Worked examples

Three end-to-end cases showing the same pattern: **define properties, shape oracles, synthesize a witness, verify it.** Each demonstrates why old tests missed the bug and what changed.

---

## Case 1 — Login

### Attack surface
Endpoints: `/login`, session middleware, password reset, MFA, remember-me, SSO callback.

### Properties → oracles

| Property | Oracle | Why old tests missed it |
|---|---|---|
| Session rotates after login | state diff (cookie pre/post) | tests checked status code only |
| Wrong credentials never reveal account existence | statistical timing/message oracle | tests checked "login fails," not leakage |
| Failed logins are rate-limited | bounded-count oracle | never tested repetition |
| User A cannot see user B's data | differential-access oracle | no multi-user setup |
| Password comparison is constant-time | timing distribution oracle | no timing measurement |
| Session cannot be fixed by an attacker | session-reuse oracle | no pre-login session injection |

### Example: session fixation oracle

```python
def test_session_rotates_on_login(session):
    pre = session.cookies.get("SID")
    session.post("/login", data={"u": "alice", "p": "correct"})
    post = session.cookies.get("SID")
    assert post and post != pre, "session not rotated; fixation risk"
```

### Example: username-enumeration oracle

```python
import statistics

def test_no_username_enumeration(session, measure):
    t_exist = [measure(session, "alice", "bad") for _ in range(50)]
    t_miss  = [measure(session, "nouser", "bad") for _ in range(50)]
    # reject on a practically significant, calibrated difference
    assert abs(statistics.mean(t_exist) - statistics.mean(t_miss)) < THRESHOLD
```

### Why a "new" bug appears later
Combination effects: MFA + password reset + SSO can create a bypass path that exists only in composition. The login function was fine; the *system* was not.

---

## Case 2 — Calculator

### Buggy code

```python
def calc(expr):
    return eval(expr)                      # arbitrary code execution
```

```python
@app.get("/calc")
def calc_route():
    e = request.args["e"]
    return render_template_string(f"<p>{eval(e)}</p>")   # SSTI / XSS / RCE
```

### Bug classes, triggers, oracles

| Bug | Trigger | Oracle | Why missed |
|---|---|---|---|
| RCE via `eval` | `__import__('os').system('id')` | side-effect / OOB | tests only checked arithmetic |
| SSTI / XSS | `{{7*7}}`, `<script>` | response diff / execution | no output-encoding oracle |
| DoS via huge exponent | `9**9**9` | bounded-time invariant | result is "correct"; needs time oracle |
| Stack overflow | 100k nested parens | crash | crash oracle existed, generator was weak |
| ReDoS | crafted string | timeout oracle | same |
| Division by zero | `1/0` | status/exception oracle | edge cases untested |
| History IDOR | request another user's history | differential-access | no multi-user test |
| Shared accumulator race | concurrent requests | differential / TSan | tests were sequential |

### The three "new bug" stories

1. **RCE that was always there.** The oracle changed from "wrong number" to "did a side effect occur?"
2. **DoS that is "correct."** Only a *bounded-time* oracle classifies it as a bug.
3. **Data leak with no wrong answer.** Every calculation is correct; only a *differential-access* oracle catches it.

---

## Case 3 — Deserialization → gadget chain → RCE

### The bug shape

```php
// sink: unserialize($_COOKIE['data'])
O:9:"Frontend":1:{s:6:"helper";O:7:"Backend":1:{s:3:"cmd";s:2:"id";}}
```

```php
class Frontend { function __destruct() { $this->helper->run(); } }   // harmless alone
class Backend  { function run() { system($this->cmd); } }            // evil only if reached
```

### Why the crash fuzzer failed

| Requirement | Fuzzer | LLM |
|---|---|---|
| Valid serialization / length prefixes | almost never produces it | constructs it exactly |
| Existing class names | unknown | reads the class list |
| Correct nested property graph | no model | reasons over the class graph |
| Multi-class magic-method chain | no | walks backward from sink |
| Detection signal | crash never fires (HTTP 200) | defines side-effect/OOB oracle |
| Reachability | needs auth/state | reads route and sink |

### LLM procedure

1. Find the `unserialize()` sink and prove input is attacker-controlled.
2. Enumerate classes with `__destruct`, `__wakeup`, `__toString`, `__call`, `__invoke`.
3. Build the call graph: which method calls what on which property.
4. Walk backward from `system`/`eval`/`call_user_func` to a magic method entry point.
5. Assemble the object graph; serialize it with exact names and lengths.
6. Verify in a sandbox: does the command run? Is there an OOB callback?

### The oracle (the real difference)

```
Oracle: a side effect outside the response (file created, callback received),
        or command output — NOT a crash.
```

This is the entire lesson: the bug was **always reachable**, but no crash-oracle tool could *see* it. The LLM added a **semantic oracle** plus **constructive reasoning**, then **verified**.
