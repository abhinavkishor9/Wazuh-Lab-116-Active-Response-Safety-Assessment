# Timeline — Lab 116: Wazuh Active Response Safety Assessment

## Phase 1 — Manager Health Check

- Checked the Wazuh Manager service.
- Confirmed that the manager service was active.
- Reviewed the status of individual Wazuh daemons.
- Ran `wazuh-analysisd -t`; no error was displayed.

## Phase 2 — Configuration Backup

- Created `/var/ossec/etc/ossec.conf.bak-lab116`.
- Verified that the backup existed.
- The captured output showed the original configuration and backup at 9,236 bytes.
- Left the Active Response configuration unchanged.

## Phase 3 — Configuration Review

- Inspected `ossec.conf` for Active Response blocks and command definitions.
- Identified command definitions including `disable-account`, `restart-wazuh`, and `netsh.exe`.
- Observed that the displayed example Active Response block was commented out.
- Confirmed that `/var/ossec/logs/active-responses.log` was configured as a log collection source.
- Did not establish that a specific response action was enabled or executed.

## Phase 4 — Rule Testing Issue

- Submitted minimal JSON events to `wazuh-logtest`.
- Observed Phase 1 and Phase 2 output without Phase 3 appearing before interruption.
- Interrupted the test and retried with a different event format.
- Did not establish a root cause for the earlier behavior.

## Phase 5 — Synthetic SSH Test

- Submitted a synthetic SSH failed-login event using documentation address `192.0.2.50`.
- Wazuh decoded the event using the `sshd` decoder.
- Rule `5710` matched at level `5`.
- The tool reported `Alert to be generated`.
- No Active Response action was deliberately triggered.

## Phase 6 — Log Review

- Reviewed entries in `/var/ossec/logs/ossec.log`.
- Observed IndexerConnector initialization warnings and later successful initialization messages for several indices.
- Did not establish a causal relationship between these messages and the earlier rule-testing behavior.

## Phase 7 — Assessment

- Manager health and detection-rule matching were verified.
- The configuration backup was confirmed.
- Effective Active Response trigger mapping remains incomplete.
- Actual response execution, impact, and rollback behavior remain unverified.
- No disruptive automated response was deliberately enabled or triggered.

## Conclusion

The lab established a safe baseline for evaluating Wazuh Active Response. The evidence supports successful detection-rule matching, but it does not establish successful automated response execution.

Further assessment should begin by identifying any effective response configuration and validating its trigger conditions, command scope, and rollback method.
