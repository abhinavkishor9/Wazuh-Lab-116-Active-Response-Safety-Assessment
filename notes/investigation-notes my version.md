# Investigation Notes

## 1. Manager Health

Commands executed:

```bash
sudo systemctl is-active wazuh-manager
sudo /var/ossec/bin/wazuh-control status
sudo /var/ossec/bin/wazuh-analysisd -t
```

### Observations

- The manager service returned `active`.
- The analysis and execution daemons were reported as running.
- `wazuh-analysisd -t` returned without displaying an error.
- Several other daemons were reported as not running.

The status of individual daemons must be interpreted according to their role and the environment. A daemon not running is not, by itself, proof of a fault.

## 2. Configuration Backup

The configuration backup was created at:

`/var/ossec/etc/ossec.conf.bak-lab116`

The backup was verified using `ls -l`. The captured output showed the original configuration and backup at 9,236 bytes.

The Active Response configuration was not modified during this assessment, so restoration was not required.

## 3. Active Response Configuration Review

The primary configuration inspected was:

`/var/ossec/etc/ossec.conf`

### Command Definitions

The inspected configuration included:

- `disable-account`
- `restart-wazuh`
- `netsh.exe`

These are observed command definitions. Their presence does not prove that a corresponding Active Response action is enabled.

### Active Response Block

The example Active Response block observed during inspection was enclosed in an XML comment. That particular block was not active configuration.

This observation does not establish that all other applicable configuration sources are free of Active Response blocks.

A search was performed against the local rules file:

```bash
sudo grep -n -i -A 8 -B 4 'active-response' /var/ossec/etc/rules/local_rules.xml
```

The command returned no output. This means the literal search found no matching text in that file; it does not establish the absence of response configuration elsewhere.

### Log Collection

The configuration included `/var/ossec/logs/active-responses.log` as a log collection source.

This confirms the configured collection path, not the presence of a successful response execution record.

## 4. Synthetic SSH Event

The following synthetic event was submitted to `wazuh-logtest`:

```text
Oct 24 07:00:00 localhost sshd[1234]: Failed password for invalid user labuser from 192.0.2.50 port 54321 ssh2
```

### Observed Results

| Field | Value |
|---|---|
| Decoder | `sshd` |
| Source IP | `192.0.2.50` |
| Source user | `labuser` |
| Rule ID | `5710` |
| Rule level | `5` |
| Description | `sshd: Attempt to login using a non-existent user` |
| Groups | `syslog`, `sshd`, `authentication_failed`, `invalid_login` |
| Result | Alert to be generated |

The event was synthetic, and `192.0.2.50` is a documentation-only IP address. Neither should be presented as evidence of a real attack.

### Interpretation

The test confirms successful event decoding and matching of rule `5710`.

It does not establish that:

- Rule `5710` is linked to an enabled Active Response action.
- A response command was invoked.
- A firewall rule or account state was changed.
- The source address was blocked.
- A response timeout or rollback was successful.

## 5. Troubleshooting Observations

Earlier minimal JSON events reached Phase 1 and Phase 2 in `wazuh-logtest`, but Phase 3 did not appear before the process was interrupted.

A subsequent synthetic SSH event completed all three phases and matched rule `5710`.

The cause of the earlier behavior remains unknown.

The manager log also contained IndexerConnector initialization warnings for multiple `wazuh-states-*` indices, followed by successful initialization messages for several indices. No causal relationship between these warnings and the earlier testing behavior was established.

## 6. Evidence Classification

### Confirmed

- The manager service was active during the checks.
- The analysis engine was running.
- The configuration test returned without displaying an error.
- A synthetic SSH event matched rule `5710` at level `5`.
- A configuration backup was created and verified.
- The inspected example Active Response block was commented out.
- The Active Response log path was configured for collection.
- IndexerConnector warnings and later successful initialization messages were observed.

### Not Established

- Whether another Active Response block is enabled in the effective configuration.
- Whether rule `5710` can trigger an enabled response.
- Whether a response command has executed.
- Whether any account, firewall, or service state was changed by Active Response.
- Whether response timeout and rollback behavior work as intended.

