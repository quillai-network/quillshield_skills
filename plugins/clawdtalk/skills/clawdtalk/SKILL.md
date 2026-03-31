---
name: clawdtalk
description: Voice and SMS alert integration for AI security agents. Use when critical security events need immediate human notification via phone call or SMS. Powers real-time alerting for Wach AI security findings.
---

# ClawdTalk Voice Alerts

Send voice calls and SMS notifications for security events detected by AI agents.

## When to Use

- Critical vulnerability detection (always call)
- High-severity smart contract audit findings
- Security breach alerts requiring immediate attention
- On-call notification for Web3 security events

## When NOT to Use

- Low/Info severity findings (log only)
- Routine audit progress updates
- Non-urgent development notifications

## Alert Decision Matrix

| Severity | Action | Priority |
|----------|--------|----------|
| Critical | Voice call + SMS | Immediate |
| High | SMS | Within 5 min |
| Medium | SMS digest | Within 15 min |
| Low | Skip | N/A |

## Prerequisites

Before triggering alerts, verify:

1. `CLAWDTALK_WS_URL` is configured
2. `CLAWDTALK_FROM_NUMBER` is set (your ClawdTalk number)
3. `CLAWDTALK_TO_NUMBER` is set (recipient)

## Voice Call Protocol

### Step 1: Assess Severity

From audit findings or security events:
- Critical = funds at risk, active exploit, access control bypass
- High = significant vulnerability, needs prompt attention
- Medium = notable finding, not time-sensitive

### Step 2: Format Alert Message

For voice calls, keep messages concise:

```
SECURITY ALERT - [Severity Level]

Issue: [1-line summary]
Contract: [name/address]
Impact: [quantified risk]

Acknowledge by pressing 1.
```

### Step 3: Send Alert

```
Connect to WebSocket: CLAWDTALK_WS_URL
Send JSON:
{
  "action": "call",
  "to": CLAWDTALK_TO_NUMBER,
  "from": CLAWDTALK_FROM_NUMBER,
  "message": "[formatted alert]",
  "urgency": "critical|high|medium"
}
```

### Step 4: Log and Track

Record alert sent in session context:
- Timestamp
- Alert type
- Recipient
- Finding reference

## SMS Protocol

For High/Medium severity:

```json
{
  "action": "sms",
  "to": CLAWDTALK_TO_NUMBER,
  "from": CLAWDTALK_FROM_NUMBER,
  "message": "[Wach AI] [Severity]: [Summary]\nContract: [name]\nDetails: [brief details]"
}
```

## Voice Message Templates

### Critical Vulnerability

```
This is a Wach AI security alert.

A critical vulnerability has been detected.

[Contract name] has [vulnerability type].

Immediate action required. Impact: [impact description].

Press 1 to acknowledge this alert.
```

### Exploit in Progress

```
This is a Wach AI security alert.

Active exploit detected on [contract/protocol].

Attack vector: [vector type]
Estimated impact: [amount/risk]

Immediate response required. Press 1 to acknowledge.
```

### Access Control Breach

```
This is a Wach AI security alert.

Access control vulnerability confirmed.

[Contract name] allows [unauthorized action].

Privilege escalation possible. Press 1 to acknowledge.
```

## Integration Pattern

ClawdTalk integrates as an output channel:

```
[Other Plugin] → Finds vulnerability → SKILL.md execution
                                           ↓
                                    Severity check
                                           ↓
                              Critical/High? → ClawdTalk alert
                                           ↓
                                    Log to session
```

## Best Practices

1. **Respect urgency** — Only use voice for Critical
2. **Be concise** — Voice messages under 30 seconds
3. **Include context** — Contract name, severity, brief impact
4. **Track acknowledgments** — Note when alerts are confirmed
5. **Avoid alert fatigue** — Batch Medium findings into digest

## Error Handling

If WebSocket connection fails:
1. Log the error
2. Retry once after 5 seconds
3. If still failing, escalate to alternative notification
4. Record failure in session context
