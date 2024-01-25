# Cascade Alerting Overview

# Configuration

Three components are used for alerts with AlertManager

1. Routes which match particular alerts with receivers
2. Receivers describe delivery configurations
3. Alerting rules for defining custom alerts

## Routes

The alertmangerconfig-alerts.yaml describes how tenant alerts are routed.

For example, the following route matches any alerts originating from any namespace starting with tenant2 (using a regular-expression match) and directs it to a receiver named tenant2-alerts-msteams.

```
    - match_re:
        namespace: tenant2.*
      receiver: tenant2-alerts-msteams
```

Once an alert has been matched, it will be considered routed unless the `continue` option is used.

## Target receivers

Receivers configure the delivery of alerts. tenant2 receivers are also defined in alertmangerconfig-alerts.yaml.

For tenant2, we currently only use MS Teams-based receivers using the an msteams adapter.

The [prometheus-msteams](https://github.com/prometheus-msteams) adapter is used to direct alerts to teams channels using a webhook. We have one msteams adapter which is deployed from the cluster/monitoring configuration. This is not currently standard OneGateway config. See: /cluster/monitoring/prometheus-msteams.yaml

In this configuration, each receiver webookConfig (e.g. in the alertmangerconfig-alerts.yaml) references a connector endpoint defined in the adapter configuration which points at a teams-channel-specific webhook.

## Alerting rules

Custom alerts may be definted in a PrometheusRule. For tenant2, the release rules are defined in cascade-release-rules.yaml.
  alertmanagerconfig:
    enabled: true
    alerts:
      - name: example
        groupBy: ["alertname", "job"]
        groupWait: 55s
        groupInterval: 5m
        repeatInterval: 12h
        continue: true
        routes:
          matcher:
            - name: tenant
              matchType: =
              value: "tenantName"
            - name: namespace
              matchType: =~
              value: "tenant2"
        receiver: gssit-alert-manager
        receivers:
          - name: gssit-alert-manager
            webhookConfigs:
              - urlSecret:
                  key: webhook
                  name: teams-notifications-webhook
These alerts describe the query criteria and message for the custom alerts. The current examples alert on successful helm releases.

## Webhook URL format

Webhook url format

In case of any spaces in URL, replace it with %20.

Example of incorrect URL:


https://API_URL.azurewebsites.net/api/createTicket/?code=XXXX&bmcQueue=DS SaaS Operations L2


Correct URL:

https://API_URL.azurewebsites.net/api/createTicket/?code=XXXX&bmcQueue=DS%20SaaS%20Operations%20L2



## Custom Prometheus Alert

Alerting rules allow you to define alert conditions based on Prometheus expressions and to send notifications about firing alerts to an external service. Whenever the alert expression results in one or more vector elements at a given point in time, the alert counts as active for these elements label sets.

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: prometheus-alert-rules
  namespace: prometheus
  labels:
    app: kube-prometheus-stack
spec:
  groups:
    - name: argocd.rules
      rules:
        - alert: ArgoAppMissing
          expr: absent(argocd_app_info)
          for: 15m
          labels:
            severity: critical
            tenant: <NAME_OF_TENANT>
          annotations:
            summary: "[ArgoCD] No reported applications"
            description: >
              ArgoCD has not reported any applications data for the past 15 minutes which
              means that it must be down or not functioning properly.  This needs to be
              resolved for this cloud to continue to maintain state.
        - alert: ArgoAppNotSynced
          expr: argocd_app_info{sync_status!="Synced"} == 1
          for: 12h
          labels:
            severity: warning
            tenant: <NAME_OF_TENANT>
          annotations:
            summary: "[{{`{{$labels.name}}`}}] Application not synchronized"
            description: >
              The application [{{`{{$labels.name}}`}} has not been synchronized for over
              12 hours which means that the state of this cloud has drifted away from the
              state inside Git.
```
Under metadata, very important is app label, because Prometheus is using selector, which adds custom alerts with this label.

```
  labels:
    app: kube-prometheus-stack
```

Another important part of the configuration is tenant label in each alert definition, which identifies alerts for particular tenant.

```
  labels:
    tenant: <NAME_OF_TENANT>
```

Under groups, it is possible to manage multiple groups with multiple alert rules. The example above shows, which fields should be used for the Prometheus alert rule.

More details on defining Prometheus alerts can be found [here](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/).

## Grafana Dashboard

Grafana dashboard to display configured alerts. This dashboard is a part of tenant's Grafana instance.
