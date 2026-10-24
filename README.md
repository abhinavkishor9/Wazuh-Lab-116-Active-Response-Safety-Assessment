# Lab 116 — Wazuh Active Response Safety Assessment

## Overview

This lab evaluates Wazuh Active Response configuration and operational safety from a SOC analyst's perspective. The investigation focuses on response command definitions, trigger conditions, detection-rule validation, execution evidence, and rollback considerations.

The assessment was performed without deliberately enabling or triggering disruptive automated actions.

## Objectives

- Review Wazuh Manager health and configuration.
- Identify Active Response command definitions and assess their potential impact.
- Examine available configuration evidence for response trigger conditions.
- Validate a synthetic SSH authentication event using `wazuh-logtest`.
- Review relevant manager logs and Active Response log collection.
- Identify gaps in execution verification and rollback validation.
- Document findings while distinguishing confirmed behavior from unresolved questions.

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

## Scenario

A SOC analyst is assessing whether automated responses could introduce operational risk. An incorrectly configured response might disable a legitimate account, block legitimate traffic, or interrupt a service.

The investigation examines the available configuration and command definitions, validates a representative SSH detection event, and identifies what additional evidence is needed before a response can be considered safe.

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

## Conclusion

The assessment verified manager health and successful matching of a synthetic SSH event. It also identified command definitions and configuration details relevant to Active Response safety.

The available evidence does not demonstrate that an automated response was enabled or executed. Further validation requires identifying the effective response configuration, examining the referenced command, mapping trigger conditions, and documenting a safe rollback procedure.

## Safety Note

Potentially disruptive response actions should be tested only in an isolated environment with a documented recovery procedure. Do not enable account disabling, firewall blocking, or service restarts solely to generate portfolio evidence.
