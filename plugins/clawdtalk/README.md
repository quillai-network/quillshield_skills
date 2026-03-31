# ClawdTalk Voice Alerts

Voice and SMS notification plugin for AI security agents. Enables real-time phone alerts for critical security events detected by Wach AI.

## What It Does

1. **Voice Call Alerts** — Initiate phone calls to notify security teams of critical threats
2. **SMS Notifications** — Send detailed text messages for Medium+ severity findings
3. **WebSocket-based** — No inbound server required; works with outbound WebSocket connection
4. **Telnyx Powered** — Enterprise-grade telephony with real phone numbers

## When to Use

- Critical vulnerability detection requiring immediate human attention
- Security breach alerts that need to wake on-call personnel
- High-severity findings from smart contract audits
- Web3 security events that demand real-time notification

## Integration with Wach AI

ClawdTalk acts as an alerting layer for security events detected by other plugins:

```
Security Event Detected → Severity Assessment → ClawdTalk Alert
                                                   ↓
                                          Voice Call (Critical)
                                          SMS (High/Medium)
```

## Setup Requirements

1. **ClawdTalk Account** — Get a phone number at https://clawdtalk.com
2. **API Credentials** — Telnyx API key and connection credentials
3. **WebSocket URL** — Your ClawdTalk WebSocket endpoint

## Environment Variables

```bash
CLAWDTALK_WS_URL=wss://your-endpoint.clawdtalk.com
CLAWDTALK_FROM_NUMBER=+1234567890
CLAWDTALK_TO_NUMBER=+0987654321
```

## Structure

```
clawdtalk/
├── skills/
│   └── clawdtalk/
│       ├── SKILL.md                    # Alert triggers and message templates
│       └── references/
│           └── message-templates.md    # Voice and SMS templates
├── .claude-plugin/
│   └── plugin.json
└── README.md
```

## Alert Severity Matrix

| Severity | Alert Type | Response Time |
|----------|-----------|---------------|
| Critical | Voice Call + SMS | Immediate |
| High | SMS | < 5 minutes |
| Medium | SMS (summary) | < 15 minutes |
| Low | Log only | N/A |

## Links

- **Website:** https://clawdtalk.com
- **GitHub:** https://github.com/team-telnyx/clawdtalk-client
- **Telnyx:** https://telnyx.com
