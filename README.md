# zabbix-hpe-msa-gen6-api

Native health and availability monitoring for HPE MSA Gen6 storage systems using the web-management API.

The template communicates directly with the MSA from Zabbix. It requires no agent, external script, cron job, Python installation or SNMP configuration.

## Features

- Automatic API login and session handling
- Low-level discovery of:
  - Drives
  - Controllers
  - Enclosures
  - Power supplies
  - Fans
  - Host ports
  - Management ports
  - SAS expanders
- Separate HDD and SSD metrics
- Drive health, status, temperature, SMART state and operating hours
- SSD life remaining only for SSDs
- Rotational speed class only for HDDs
- Controller health, redundancy, failover, firmware, cache and memory
- Fan speed and health
- PSU status, health, model, serial number and firmware
- Enclosure health and power consumption
- Port status, health and link speed
- Active MSA alerts grouped by severity
- Latest active problem, reason and HPE recommended action
- API availability monitoring using `nodata()`
- Trigger dependencies to reduce duplicate alerts

The template uses a small number of API master items and distributes their data through dependent items. Hundreds of monitored values therefore do not produce hundreds of API requests.

## Tested configuration

- Zabbix 7.4
- HPE MSA 1060 SAS
- Controller firmware `IN210P02-01`

Other HPE MSA Gen6 models may use the same API structure, but have not yet been verified. Reports from MSA 2060 and MSA 2062 systems are welcome.

## Installation

1. Download `template_hpe_msa_api_7.4_v1.0.0.yaml`.
2. In Zabbix, open **Data collection → Templates → Import**.
3. Import the YAML file.
4. Link the template **HPE MSA API** to the desired host.
5. Configure the host macros listed below.
6. Wait for the master items and discovery rules to run, or execute them manually.

When updating an existing installation, enable **Update existing**. Use **Delete missing** for discovery rules, item prototypes and trigger prototypes when upgrading from an earlier development version, so obsolete prototypes are removed.

## Required macros

Set these macros on the monitored host:

| Macro | Example | Description |
|---|---|---|
| `{$HPE.MSA.API.HOST}` | `msa.example.local` | Management hostname or IP address |
| `{$HPE.MSA.API.USER}` | `Administrator` | MSA API username |
| `{$HPE.MSA.API.PASSWORD}` | — | MSA password; store as a secret macro |
| `{$HPE.MSA.API.PORT}` | `443` | Management HTTPS port |
| `{$HPE.MSA.API.SCHEME}` | `https` | URL scheme |

Optional thresholds:

| Macro | Default | Description |
|---|---:|---|
| `{$HPE.MSA.DRIVE.TEMP.WARN}` | `55` | Drive temperature warning threshold in °C |
| `{$HPE.MSA.DRIVE.TEMP.CRIT}` | `60` | Drive temperature critical threshold in °C |
| `{$HPE.MSA.API.NODATA}` | `10m` | Time without core API data before an availability problem is raised |
| `{$HPE.MSA.ALERT.NODATA}` | `15m` | Time without alert API data before an alert-endpoint problem is raised |

## Permissions and security

Use a dedicated monitoring account with the least privileges that still permit the required read-only API calls. Do not place credentials directly in item scripts. Store the password in the secret host macro `{$HPE.MSA.API.PASSWORD}`.

The template uses the same API consumed by the MSA web interface. This interface is not presented as a stable public REST API by HPE, so firmware changes may require parser adjustments.

## How it works

The template performs the following flow inside Zabbix:

```text
MSA login
   ↓
Session key
   ↓
API master items
   ↓
Parsed JSON
   ↓
Dependent items and low-level discovery
   ↓
Items, triggers and alert context
```

Authentication is generated at runtime from the configured username and password. Session keys are not stored permanently.

## Troubleshooting

### `Username or password is missing`

The secret password macro is not automatically inserted into Zabbix's manual item test dialog. Enter it manually for the test, or let the saved item execute normally on the host.

### `Login failed`

Check the username, password, management address, scheme and port. Confirm that the account can sign in to the MSA web interface.

### API items are unsupported

Open the affected master item and review its error message. Common causes are:

- Wrong credentials
- DNS or routing failure from the Zabbix server or proxy
- Firewall restrictions
- Changed API response after an MSA firmware update

### Duplicate item keys while linking the template

Remove older locally created MSA items and discovery rules from the host before linking the template. Host-level macros can remain in place.

## Known limitations

- Tested on one HPE MSA 1060 SAS system so far
- Performance analytics and capacity planning are intentionally outside the v1.0.0 scope
- Historical MSA alert conditions are not reproduced as thousands of permanent Zabbix items
- Informational active alerts are collected but do not automatically create high-severity problems
- The HPE web-management API may change between firmware versions

## Project scope

Version 1.0.0 focuses on operational health:

> Is the MSA reachable, are its critical components healthy, and will Zabbix clearly report a failure with useful context?

Performance analytics may be considered separately in a future release.

## Contributing

Feedback is especially useful from administrators running:

- HPE MSA 1060 FC or iSCSI
- HPE MSA 2060
- HPE MSA 2062
- Different Gen6 firmware releases

When reporting an issue, include the Zabbix version, MSA model, controller firmware and the exact unsupported-item error. Remove hostnames, addresses, serial numbers and credentials before sharing API output.

## License

Choose and add a license before publishing the repository. MIT is a practical option for a community Zabbix template.
