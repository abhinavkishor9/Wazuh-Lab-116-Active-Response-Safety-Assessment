# Troubleshooting Notes — Lab 116: Wazuh Active Response Safety Assessment

## 1. Incomplete `wazuh-logtest` Output

### Symptom

Minimal JSON events reached Phase 1 and Phase 2, but Phase 3 did not appear before the process was interrupted.

One tested event was:

```json
{"win":{"system":{"eventID":"4625"}}}
```

### Action Taken

The test process was interrupted with `Ctrl+C` and relaunched. A synthetic SSH authentication event was submitted afterward.

### Result

The SSH event completed all three phases and matched rule `5710` at level `5`.

### Assessment

The successful SSH test does not establish the cause of the earlier JSON test behavior. The root cause remains unknown.

## 2. Manager Health Check

Commands used:

```bash
sudo systemctl is-active wazuh-manager
sudo /var/ossec/bin/wazuh-control status
sudo /var/ossec/bin/wazuh-analysisd -t
```

### Result

- The manager service was active.
- The analysis and execution daemons were running.
- The analysis configuration test returned without displaying an error.

These results do not establish that Active Response was enabled or that a response executed.

## 3. IndexerConnector Initialization Warnings

### Observation

The manager log contained IndexerConnector initialization warnings for multiple `wazuh-states-*` indices. Later messages reported successful initialization for several indices.

### Assessment

The warnings indicate initialization problems at the time of those messages. The later messages show successful initialization for the indices identified in those messages.

The available evidence does not establish that these warnings caused the earlier `wazuh-logtest` behavior. Treat them as separate observations unless additional evidence establishes a relationship.

## 4. Commented Active Response Example

### Observation

The inspected configuration contained an example Active Response block enclosed in an XML comment.

### Assessment

A commented-out block is not active configuration. However, the observation does not prove that Active Response is absent from all applicable configuration sources.

### Safe Handling

- Do not uncomment an example block simply to generate a response.
- Identify the effective response configuration and its trigger conditions.
- Inspect the referenced command before considering execution.
- Document the action's scope and rollback procedure.
- Avoid enabling disruptive actions without a controlled test plan.

## 5. Configuration Backup Verification

The backup path was:

`/var/ossec/etc/ossec.conf.bak-lab116`

The following command was used:

```bash
sudo ls -l /var/ossec/etc/ossec.conf*
```

The backup was present. The captured output showed the original configuration and backup at 9,236 bytes.

Restoration was not required because the Active Response configuration was not modified.

## 6. Safe Troubleshooting Principles

- Do not restart the manager without a specific reason.
- Do not modify detection rules merely to force a response.
- Do not assume that a command definition proves execution.
- Do not equate a rule match with successful containment.
- Preserve the original configuration before making changes.
- Verify rollback behavior before testing disruptive responses.
- Record unresolved issues as inconclusive rather than inventing a root cause.

## 7. Current Status

Manager health checks and the synthetic SSH rule-matching test completed successfully. The earlier JSON test behavior remains unexplained.

Effective Active Response trigger mapping, actual execution, and rollback behavior remain unverified.
