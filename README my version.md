# Wazuh-Lab-116-Active-Response-Safety-Assessment
## Overview
### What Is Wazuh Active Response?

Wazuh normally detects suspicious activity by collecting events and matching them against detection rules. Active Response adds the ability to execute a configured command when specified conditions are met.

1. **Endpoint generates an event**

   Example: repeated failed logins.

2. **Wazuh evaluates detection rules**

   An event matches a rule and generates an alert.

3. **Active Response checks its configuration**

   The alert must meet the configured response conditions.

4. **A configured response may execute**

   Example: temporarily block a source IP address.

An alert does not automatically mean Active Response will execute. The response must be configured, and the relevant conditions must be satisfied.

### Example: Failed-Login Detection

Suppose Wazuh detects repeated failed login attempts from `192.0.2.50`.

- **Detection:** Wazuh generates an alert.
- **Response configuration:** The alert meets the conditions defined for a configured response.
- **Action:** A response command may block the source address.
- **Verification:** The analyst checks execution evidence and confirms whether the block took effect.

The address `192.0.2.50` is an example documentation address, not an IP observed in your environment.

### Why Can Active Response Be Dangerous?

| Risk | Example |
|---|---|
| False positive | Legitimate activity triggers a block. |
| Overly broad trigger | A low-confidence alert launches a disruptive action. |
| Excessive scope | A block affects an entire network instead of one source. |
| Repeated execution | A response runs repeatedly and creates operational problems. |
| Failed rollback | A temporary block remains in place longer than intended. |

The key SOC principle is: **automating a response can reduce reaction time, but it can also automate mistakes.**

This lab evaluates Wazuh Active Response configuration and operational safety from a SOC analyst's perspective. The investigation focuses on response command definitions, trigger conditions, detection-rule validation, execution evidence, and rollback considerations.

The assessment was performed without deliberately enabling or triggering disruptive automated actions.

## Lab Objectives

- Understand how Wazuh Active Response extends detection by executing configured commands when specified conditions are met.
- Examine the relationship between detection alerts, response configuration, and command execution.
- Review Active Response configuration to identify response commands, trigger conditions, and potential operational risks.
- Use `wazuh-logtest` to evaluate a synthetic SSH failed-login event and observe how Wazuh processes the event against its detection rules.
- Distinguish a detection-rule match from an Active Response trigger and a successfully executed response.
- Identify risks associated with false positives, overly broad triggers, excessive response scope, repeated execution, and failed rollback.
- Determine what evidence is needed to verify response execution, confirm whether an action took effect, and assess rollback behavior.
- Document findings while distinguishing verified observations from assumptions and unconfirmed behavior.
- Assess Active Response safety without deliberately triggering disruptive actions.

## Environment

| Component | Details |
|---|---|
| Platform | Wazuh |
| Manager version | 4.13.1 |
| Main configuration | `/var/ossec/etc/ossec.conf` |
| Local rules | `/var/ossec/etc/rules/local_rules.xml` |
| Manager log | `/var/ossec/logs/ossec.log` |
| Active Response log | `/var/ossec/logs/active-responses.log` |
| Testing utility | `wazuh-logtest` |

## Lab Scenario

Wazuh Active Response enables security teams to automate actions in response to detected threats. While this capability can reduce response time, an incorrectly configured response may block legitimate traffic, disrupt normal operations, or leave systems affected if rollback does not work as intended.

In this lab, the analyst evaluates the safety and behavior of Active Response in a Wazuh environment. The investigation begins by checking the manager's health and reviewing the available response configuration, including command definitions, trigger conditions, and any configured execution timeouts or rollback mechanisms.

A synthetic SSH failed-login event is then examined using `wazuh-logtest` to understand how Wazuh processes the event and matches it against detection rules. This provides a controlled way to study detection behavior without deliberately triggering a disruptive response.

### Investigation Focus

- Determine whether Active Response commands and response conditions are configured.
- Examine how detection alerts relate to configured response actions.
- Identify potential risks from false positives, broad trigger conditions, repeated execution, and inadequate rollback.
- Establish what evidence would be required to confirm command execution and verify that a response took effect.
- Document configuration gaps and distinguish observed behavior from assumptions.

The investigation does not assume that an alert automatically triggers a response or that a configured command has successfully executed. Findings must be supported by available configuration and execution evidence. No disruptive response is deliberately triggered during the assessment.

## Investigation

1. Checked manager service status and daemon health.
2. Backed up `ossec.conf` to `ossec.conf.bak-lab116`.
3. Inspected Active Response blocks and command definitions.
4. Examined the configured Active Response log collection path.
5. Tested a synthetic SSH authentication event using `wazuh-logtest`.
6. Reviewed manager logs for IndexerConnector initialization messages.
7. Documented confirmed findings and unresolved response-safety questions.

## Key Findings

### Manager Health

The Wazuh Manager service reported `active`. The analysis and execution daemons were reported as running, and `wazuh-analysisd -t` returned without displaying an error.

### Configuration Review

The inspected configuration contained command definitions including `disable-account`, `restart-wazuh`, and `netsh.exe`. The example Active Response block observed during inspection was inside an XML comment.

These findings do not establish whether another Active Response block is enabled elsewhere in the effective configuration.

The configuration included `/var/ossec/logs/active-responses.log` as a log collection source. This does not establish that a response was executed.

### Synthetic SSH Detection

A synthetic SSH failed-login event was decoded by `sshd` and matched rule `5710`, level `5`, with the description:

`sshd: Attempt to login using a non-existent user`

The event used the documentation IP address `192.0.2.50`. It was a test event, not evidence of an actual attack.

The successful rule match demonstrates detection behavior only. It does not prove that an Active Response action was configured, triggered, or executed.

### Log Observations

The manager log contained IndexerConnector initialization warnings for multiple `wazuh-states-*` indices. Later messages reported successful initialization for several indices.

The available evidence does not establish that these warnings caused the earlier `wazuh-logtest` behavior.

## Limitations

- Effective Active Response configuration has not been fully established.
- The association between rule `5710` and any enabled response remains unverified.
- No Active Response execution was deliberately initiated.
- No account, firewall, or service change was verified as a response outcome.
- Response timeout and rollback behavior were not tested.
- The cause of the earlier incomplete JSON test output remains unknown.

