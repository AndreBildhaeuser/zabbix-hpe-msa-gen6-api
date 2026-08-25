# Zabbix HPE MSA Gen6 API Monitoring Template

Native **Zabbix monitoring template for HPE MSA Gen6 storage systems** using the built-in HPE MSA web-management API.

Designed for **HPE MSA 1060, MSA 2060 and MSA 2062** storage systems running Gen6 firmware. The template provides hardware health, availability and failure monitoring directly from Zabbix without requiring an agent, external scripts, cron jobs, Python or SNMP.

> **Compatibility:** HPE MSA Gen6 only. This template is **not compatible with HPE MSA Gen5 or earlier generations**.

## Why this template exists

Monitoring an HPE MSA Gen6 storage system with Zabbix should not require additional monitoring hosts, external scripts or manual configuration for every individual hardware component.

This template communicates directly with the **HPE MSA Gen6 WBI / web-management API** and automatically discovers the relevant storage hardware.

The focus is operational monitoring:

**Is the storage system healthy, redundant and available — and will Zabbix alert me when something fails?**

## Features

* Native HPE MSA Gen6 API monitoring from Zabbix
* Automatic API authentication and session handling
* No Zabbix agent required on the storage system
* No external scripts
* No Python installation
* No cron jobs
* No SNMP configuration
* Low-level discovery (LLD)
* Automatic hardware health monitoring
* Automatic trigger generation
* HDD and SSD aware monitoring
* Active MSA alert monitoring

### Automatically discovered components

The template uses Zabbix low-level discovery for:

* Drives
* Controllers
* Enclosures
* Power supplies
* Fans
* Host ports
* Management ports
* SAS expanders

## Drive monitoring

Each discovered drive can expose information including:

* Health
* Status
* Model
* Serial number
* Architecture / media type
* Temperature
* SMART state
* Power-on hours
* Storage pool
* Usage
* Number of I/Os
* Average response time
* Total data transferred

Drive-specific metrics are handled according to the detected media type:

### HDD

For rotating hard drives the template exposes the rotational speed class.

### SSD

For solid-state drives the template exposes SSD life remaining where supported by the MSA.

This prevents invalid values such as SSD lifetime information being interpreted for HDDs.

## Controller monitoring

Both storage controllers are automatically discovered and monitored for:

* Health
* Operational status
* Redundancy
* Failover state
* Firmware
* Cache state
* Cache memory
* System memory
* Write policy
* Management IP address

Controller failover and degraded redundancy conditions generate Zabbix problems automatically.

## Enclosure monitoring

The template monitors enclosure information including:

* Health
* Status
* Model
* Serial number
* Drive count
* Fan count
* PSU count
* Power consumption

## Power supply monitoring

Power supplies are automatically discovered and monitored for:

* Health
* Status
* Model
* Serial number
* Firmware revision

## Fan monitoring

Fans are automatically discovered and monitored for:

* Health
* Status
* Fan speed

## Host port monitoring

Host ports are automatically discovered per controller and monitored for:

* Health
* Status
* Actual link speed
* Target ID

## Management port monitoring

Management interfaces are automatically discovered and monitored for:

* Health
* IP address
* Link speed
* Duplex state

## SAS expander monitoring

SAS expanders are automatically discovered and monitored for:

* Health
* Status
* Firmware

## Active HPE MSA alerts

The template retrieves active alerts directly from the HPE MSA API and exposes information including:

* Total active alerts
* Informational alerts
* Warning alerts
* Critical alerts
* Active problem summary
* Problem component
* Problem description
* Problem reason
* Recommended action
* Severity
* Detection time

This allows Zabbix to report not only that a hardware component is unhealthy, but also MSA-generated diagnostic information where available.

## Failure detection

The template includes triggers for relevant hardware and redundancy conditions, including:

* Drive health failures
* Drive status failures
* High drive temperature
* Controller health failures
* Controller not operational
* Controller failover
* Controller redundancy degraded
* Enclosure health problems
* Power supply failures
* Fan failures
* Host port health problems
* Management port health problems
* SAS expander health problems
* Active MSA warning and critical alerts

## Real-world testing

The template has been developed and tested against an **HPE MSA 1060 Gen6 SAS storage system** running in a production Zabbix environment.

Controller failover and redundancy monitoring have also been validated during controller restart and HPE MSA firmware update operations.

During a controller transition, Zabbix successfully detected conditions including:

* Controller failover
* Degraded controller redundancy
* Controller health degradation
* Controller not operational
* Enclosure health degradation
* Management interface health degradation

The generated problems automatically recovered after the MSA returned to a healthy redundant state.

## Requirements

* HPE MSA Gen6 storage system
* HPE MSA 1060, MSA 2060 or MSA 2062 family
* HPE MSA web-management API / WBI access
* Zabbix 7.4 or newer recommended
* Network connectivity from the Zabbix server or proxy to the MSA management interface
* Dedicated MSA monitoring account recommended

## Installation

1. Download the latest YAML template from the GitHub Releases section.
2. Open Zabbix.
3. Navigate to **Data collection → Templates**.
4. Select **Import**.
5. Import the YAML template.
6. Link the template to the HPE MSA host.
7. Configure `{$HPE.MSA.API.HOST}`, `{$HPE.MSA.API.USER}` and `{$HPE.MSA.API.PASSWORD}` on the HPE MSA host (see [Configuration macros](#configuration-macros)).
8. Wait for the first discovery cycle.

Zabbix will then automatically discover the available MSA hardware and create the corresponding items and triggers.

## Configuration macros

After linking the template to the HPE MSA host, configure the following
host-specific macros:

**Data collection → Hosts → your HPE MSA host → Macros**

| Macro | Required | Description |
|---|:---:|---|
| `{$HPE.MSA.API.HOST}` | **Yes** | MSA management hostname or IP address. |
| `{$HPE.MSA.API.USER}` | **Yes** | MSA API username. A dedicated monitoring account with the minimum required permissions is recommended. |
| `{$HPE.MSA.API.PASSWORD}` | **Yes** | Password for the MSA API user. Store as **Secret text**. |

### Optional settings

The following macros have sensible defaults in the template and normally
do not need to be configured on the host:

| Macro | Default | Description |
|---|---:|---|
| `{$HPE.MSA.API.PORT}` | `443` | TCP port used for the MSA API connection. |
| `{$HPE.MSA.API.SCHEME}` | `https` | Protocol used for the MSA API connection. |
| `{$HPE.MSA.API.NODATA}` | `10m` | Time without successful API data before an availability problem is raised. |
| `{$HPE.MSA.ALERT.NODATA}` | `15m` | Time without alert API data before an alert-endpoint problem is raised. |
| `{$HPE.MSA.DRIVE.TEMP.WARN}` | `55` | Drive temperature warning threshold in °C. |
| `{$HPE.MSA.DRIVE.TEMP.CRIT}` | `60` | Drive temperature critical threshold in °C. |

Host-level macro values override the defaults inherited from the template.

## Design philosophy

This project intentionally focuses on **health and availability monitoring** rather than attempting to reproduce the complete HPE MSA performance interface inside Zabbix.

The primary goal is to answer operational questions such as:

* Is every drive healthy?
* Is a drive approaching failure?
* Are both controllers operational?
* Is controller redundancy intact?
* Has a controller failed over?
* Are both power supplies healthy?
* Are all fans operating?
* Are host and management interfaces healthy?
* Has the MSA generated a warning or critical alert?
* Has any hardware component degraded?

Detailed storage performance analysis can still be performed using the native HPE MSA management interface.

## Compatibility

| Platform                    | Status              |
| --------------------------- | ------------------- |
| HPE MSA 1060 Gen6           | Tested              |
| HPE MSA 2060 Gen6           | Expected compatible |
| HPE MSA 2062 Gen6           | Expected compatible |
| HPE MSA Gen5                | Not supported       |
| Earlier HPE MSA generations | Not supported       |
| Zabbix 7.4                  | Tested              |

Testing on additional HPE MSA Gen6 models is welcome.

If you successfully use the template with another MSA Gen6 model or firmware release, please open a GitHub issue or discussion with the model and firmware version.

## Project status

**Stable / production tested**

The template was originally developed because available HPE MSA Zabbix monitoring solutions did not provide the desired Gen6 API-based hardware health monitoring.

It is published as an open-source project so other administrators running **Zabbix with HPE MSA Gen6 storage** can use and improve it.

## Contributing

Bug reports, compatibility reports and improvements are welcome.

Especially useful are reports from administrators running:

* HPE MSA 2060 Gen6
* HPE MSA 2062 Gen6
* Different HPE MSA Gen6 firmware releases
* Different Zabbix 7.x releases

Please include the HPE MSA model, firmware version and Zabbix version when reporting compatibility issues.

## License

Released under the MIT License.

The software is provided **"as is"**, without warranty of any kind. Use in production environments is at your own risk.
