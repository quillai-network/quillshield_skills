# ClawdTalk Message Templates

Pre-defined voice and SMS templates for security alerting.

## Voice Call Templates

### Critical Severity

**Template: Vulnerability Alert**
```
This is a Wach AI security alert.

A critical vulnerability has been detected in {CONTRACT_NAME}.

Issue: {VULNERABILITY_TYPE}
Impact: {IMPACT_DESCRIPTION}

Immediate action is required.

Press 1 to acknowledge this alert.
```

**Template: Active Exploit**
```
This is a Wach AI security alert.

An active exploit has been detected on {PROTOCOL_NAME}.

Attack vector: {ATTACK_VECTOR}
Estimated impact: {IMPACT_AMOUNT}

Immediate response is required.

Press 1 to acknowledge this alert.
```

**Template: Access Control Breach**
```
This is a Wach AI security alert.

A critical access control vulnerability has been confirmed.

Contract: {CONTRACT_NAME}
Issue: {UNAUTHORIZED_ACTION}

Privilege escalation is possible.

Press 1 to acknowledge this alert.
```

### High Severity

**Template: High Priority Finding**
```
Wach AI alert.

High severity vulnerability found in {CONTRACT_NAME}.

{VULNERABILITY_TYPE} detected.

Review recommended within the hour.

Press 1 to acknowledge.
```

## SMS Templates

### Critical
```
[WACH AI - CRITICAL]
Contract: {CONTRACT_NAME}
Issue: {VULNERABILITY_TYPE}
Impact: {IMPACT}
Check console for details. Voice call incoming.
```

### High
```
[WACH AI - HIGH]
Contract: {CONTRACT_NAME}
Issue: {VULNERABILITY_TYPE}
Location: {FUNCTION_NAME}
Impact: {IMPACT}
Review recommended.
```

### Medium Digest
```
[WACH AI - MEDIUM]
{COUNT} findings require attention.

Summary:
{FINDING_1}
{FINDING_2}
...

See full report for details.
```

## Template Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `{CONTRACT_NAME}` | Name of affected contract | "LendingPool" |
| `{VULNERABILITY_TYPE}` | Type of vulnerability | "reentrancy" |
| `{IMPACT}` | Quantified impact | "Up to $2M at risk" |
| `{PROTOCOL_NAME}` | Protocol identifier | "Compound fork" |
| `{ATTACK_VECTOR}` | How attack is executed | "flashloan price manipulation" |
| `{IMPACT_AMOUNT}` | Estimated loss | "$500K" |
| `{UNAUTHORIZED_ACTION}` | What's allowed incorrectly | "anyone can call mint()" |
| `{FUNCTION_NAME}` | Affected function | "deposit()" |
| `{COUNT}` | Number of findings | "3" |
| `{FINDING_N}` | Brief finding summary | "Oracle stale on L45" |

## Message Length Guidelines

- **Voice:** 20-30 seconds maximum
- **SMS:** 160 characters (single segment) for critical, 320 for others

## Acknowledgment Handling

When recipient presses 1:
1. Log acknowledgment with timestamp
2. Update alert status to "acknowledged"
3. Stop repeat call attempts
4. Record in session context

## Rate Limiting

To prevent alert fatigue:
- Max 3 voice calls per hour
- Max 10 SMS per hour
- Batch Medium findings into digest
