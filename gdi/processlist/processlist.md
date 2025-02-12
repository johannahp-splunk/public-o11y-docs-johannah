(processlist)=

# Host process list
<meta name="description" content="Use this Splunk Observability Cloud integration for the processlist monitor. See benefits, install, configuration, and metrics">

## Description

The {ref}`Splunk Distribution of OpenTelemetry Collector <otel-intro>` provides this integration as the `processlist` monitor type for the Smart Agent Receiver.

This monitor reports the running processes for a host, similar to the output of the `top` or `ps` commands on *nix systems. The output format is a special base64-encoded event that appears under the Infrastructure view for a specific host. Historical process information is not retained on Splunk Observability Cloud.

This monitor is available on Linux and Windows.

### Benefits

```{include} /_includes/benefits.md
```

## Installation

```{include} /_includes/collector-installation.md
```

## Configuration

```{include} /_includes/configuration.md
```

```{note}
Provide a processlist monitor entry in your Collector or Smart Agent (deprecated) configuration. Use the appropriate form for your agent type.
```

### Splunk Distribution of OpenTelemetry Collector

To activate this monitor in the Splunk Distribution of OpenTelemetry Collector, add the following to the `receivers` section of your agent configuration:

```yaml
receivers:
  smartagent/processlist:
    type: processlist
    ...  # Additional config
```

To complete the monitor activation, you must also include the `smartagent/processlist` receiver item in a `logs` pipeline. To do this, add the receiver item to the `service` > `pipelines` > `logs` > `receivers` section of your configuration file. 

The following example shows how to configure the `logs` pipeline using the required `signalfx` exporter:

```yaml
service:
  pipelines:
    logs/signalfx:
      receivers: [signalfx, smartagent/processlist]
      exporters: [signalfx]
      processors: [memory_limiter, batch, resourcedetection]
```

### Smart Agent

To activate this monitor in the Smart Agent, add the following to your agent configuration:

```yaml
monitors:  # All monitor config goes under this key
  - type: processlist
    ...  # Additional config
```

See {ref}`smart-agent` for an autogenerated example of a YAML configuration file, with default values where applicable.

## Metrics

The Splunk Distribution of OpenTelemetry Collector does not do any built-in filtering of metrics for this monitor.

## Get help

```{include} /_includes/troubleshooting.md
```
