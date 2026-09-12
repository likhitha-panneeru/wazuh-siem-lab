# Triage Note: Windows Brute Force Detection

## Alert Summary

| Field | Value |
|-------|-------|
| **Alert** | Multiple Windows logon failures detected - possible brute force attack |
| **Rule ID** | 100002 (Custom Rule) |
| **Severity** | Level 10 (High) |
| **Timestamp** | Sep 11, 2026 @ 18:06:00 UTC |
| **Host** | DESKTOP-WIN11 (Agent 001) |
| **MITRE ATT&CK** | T1110 - Brute Force / Credential Access |

## Evidence Observed

- Windows Event ID 4625 (Logon Failure) fired 5+ times within 120 seconds
- **Target username:** "hacker" — not a legitimate account on this system
- **Source IP:** 127.0.0.1 (lab environment; in production this would be the attacker's IP)
- **Logon Type:** 3 (Network logon — indicates remote authentication attempt)
- **Authentication Package:** NTLM
- **Failure Reason:** Unknown user name or bad password
- **Status Code:** 0xC000006D (logon failure)
- **Sub Status:** 0xC0000064 (user name does not exist)
- **Workstation:** DESKTOP-GDMPR2U

## Analysis

The alert was triggered by our custom rule 100002, which fires when 5 or more Windows logon failures (base rule 60122) occur within a 120-second window.

Key observations:
1. The targeted username "hacker" does not exist on the system, indicating the attacker is guessing usernames
2. Logon Type 3 means this is a network-based attack, not someone at the keyboard
3. NTLM authentication was used, which is common in brute force tools
4. The rapid frequency (multiple attempts per second) is inconsistent with human behavior
5. Sub Status 0xC0000064 confirms the account doesn't exist — the attacker hasn't found valid usernames yet

## Verdict

**True Positive** — The pattern of multiple rapid authentication failures against a non-existent account from the same source is consistent with an automated credential guessing / brute force attack.

## Recommended Actions

1. **Immediate:** Block the source IP at the firewall (in production)
2. **Investigate:** Check if any logon succeeded after the failures (Event ID 4624) from the same source
3. **Investigate:** Review other accounts targeted from the same source IP
4. **Harden:** Enable account lockout policy after X failed attempts if not already configured
5. **Monitor:** Watch for lateral movement attempts from the source IP
6. **Long-term:** Consider implementing MFA for all accounts

## MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|--------|-----------|-----|
| Credential Access | Brute Force | T1110 |
| Credential Access | Brute Force: Password Guessing | T1110.001 |

---

*Investigated by: zuzu | Date: Sep 11, 2026 | Wazuh SIEM Lab Project*
