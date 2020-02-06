# webhook-for-alertmanager
Basic Python Flask Webhook for Alertmanager

## Usage

`prometheus.yml`

```
rule_files:
  - 'alert.rules'

alerting:
  alertmanagers:
  - dns_sd_configs:
    - names:
      - 'tasks.alertmanager'
      type: 'A'
      port: 9093
```

`alert.rules`

```
groups:
- name: alert.rules
  rules:
  - alert: high_load
    expr: node_load1 * on(instance) group_left(nodename) (node_uname_info) > 4
    for: 2m
    labels:
      alert_channel: "webhook"
      team: "devops"
      severity: "critical"
    annotations:
      title: "High Load Average on {{ $labels.nodename }}"
      description: "{{ $labels.nodename }} of job {{ $labels.job }} is under high load."
      summary: "- Node: {{ $labels.nodename }} \n- Metric: node_load1 \n- Value: {{ $value }}"

```

`alertmanager.yml`

```
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/x/x/x'
  
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'alert-to-slack'
  routes:
  - match:
      alert_channel: webhook
    receiver: alert-to-webhook

receivers:
- name: 'alert-to-slack'
  slack_configs:
    - send_resolved: true
      title_link: 'http://alertmanager.domain.com/#/alerts'
      title: "{{ range .Alerts }}{{ .Annotations.title }}{{ end }}"
      channel: '#system_events'
      text: "Alert Channel:  {{ range .Alerts }}{{ .Labels.alert_channel }}{{ end }}\nRouted to Team: {{ range .Alerts }}{{ .Labels.team }}{{ end }} \nDescription of the alarm is: {{ range .Alerts }}{{ .Annotations.description }}{{ end }}\nSummary of the alarm is: {{ range .Alerts }}{{ .Annotations.summary }}{{ end }}"
      username: 'AlertManager'
      icon_emoji: ':prometheus:'

- name: 'alert-to-webhook'
  webhook_configs:
    - send_resolved: true
      url: 'http://192.168.0.106:5000'
  slack_configs:
    - send_resolved: true
      channel: '#system_events'
      username: 'Alertmanager Alerts'
      title: "{{ range .Alerts }}{{ .Annotations.title }}{{ end }}"
      text: "Alert Channel:  {{ range .Alerts }}{{ .Labels.alert_channel }}{{ end }}\nRouted to Team: {{ range .Alerts }}{{ .Labels.team }}{{ end }} \nDescription of the alarm is: {{ range .Alerts }}{{ .Annotations.description }}{{ end }}\nSummary of the alarm is: {{ range .Alerts }}{{ .Annotations.summary }}{{ end }}"

```
