.. _`monitoring`:

.. highlight:: yaml

Monitoring
==========

OpenBikes components, especially the ttn tracker adapters, provide monitoring endpoints at ``/metrics``. There you find values in the `text format`_ understood by monitoring tools like Prometheus_.


Setup
-----

For installation of Prometheus_ see the official `installation documentation <https://prometheus.io/docs/prometheus/latest/installation/>`_. They provide pre-compiled binaries working on many distributions, sometimes your distributions package repository also contains relatively current versions.

Example prometheus configuration::

    global:
      scrape_interval:     15s # By default, scrape targets every 15 seconds.
      evaluation_interval: 15s # By default, evaluate rules every 15 seconds.
    
    scrape_configs:
      - job_name: cykel-ttn
        static_configs:
          - targets: ['localhost:9000']
      - job_name: cykel-ttn-wifi
        static_configs:
          - targets: ['localhost:9003']

For each different tracker adapter, you need to add another scrape job. In the example above we're using the easy ``static_configs`` for the list of hostname and port of the adapters, but you can also use more advanced stuff like the ``file_sd_configs`` or runtime based service discovery for docker swarm or kubernetes. Read the `Prometheus Configuration Documentation`_ if you want to know more about these.

Displaying
^^^^^^^^^^

For displaying the monitored values (e.g. battery levels) we're using Grafana_ with the `Prometheus data source`_. Grafana is also available as package for your distribution, standalone linux binary or in your distributions repository. See the `Grafana download page`_ for more details where to get and how to install.

After installing and running through the first-use assistant, you can create your own dashboards with panels.

One panel for displaying battery levels has these settings::

    Type: Line Chart
    Metric: tracker_battery_volts
    Legend: {{device_id}}
    Left Y Axis Unit: Volt (V)

You can bring even more details in your charts, feel free to experiment and share them in this documentation.

Alerting
^^^^^^^^

For monitoring and graphing or displaying a dashboard, Prometheus and Grafana are all you need. If you want to be notified, you also need the Alertmanager_.

Example additions to your prometheus configuration if you want to be notified::

    rule_files:
      - "cykel.rules.yml"

    alerting:
      alertmanagers:
        - path_prefix: '_alert'
          static_configs:
          - targets: ['localhost:9093']

The rules on which alerts are triggered are placed in their own ``rule_files``. For example, we're using rules based on these to get notified if trackers have low battery levels or if they didn't send their location for a longer time.

::

    groups:
    - name: cykel
      rules:
      - alert: TrackerLowBattery
        expr: tracker_battery_volts < 3.3
        labels:
          severity: warning
          area: battery
        annotations:
          summary: "Low Battery on Tracker {{ $labels.device_id }}"
          description: "Tracker {{ $labels.device_id }} has low battery voltage: {{ $value }} V"
      - alert: TrackerCriticalBattery
        expr: tracker_battery_volts < 3.05
        labels:
          severity: critical
          area: battery
        annotations:
          summary: "Critical Battery on Tracker {{ $labels.device_id }}"
          description: "Tracker {{ $labels.device_id }} has critical battery voltage: {{ $value }} V"
      - alert: TrackerLastUpdate45mAgo
        expr: time() - tracker_last_data_update > 60 * 45
        labels:
          severity: info
          area: uplink
        annotations:
          summary:  "Last Update of Tracker {{ $labels.device_id }} more than 45m ago"
          description: "Tracker {{ $labels.device_id }} last reported {{ humanizeDuration $value }} ago"
      - alert: TrackerLastUpdate2hAgo
        expr: time() - tracker_last_data_update > 60 * 60 * 2
        labels:
          severity: warning
          area: uplink
        annotations:
          summary: "Last Update of Tracker {{ $labels.device_id }} more than 2h ago"
          description: "Tracker {{ $labels.device_id }} last reported {{ humanizeDuration $value }} ago"

Where these alerts are delivered is configured in the `Alertmanager`_ configuration. Have a look into their documentation for the different alert receivers (like Slack, Pagerduty) and how to configure them. Even more receivers are listed in the `Integrations Documentation`_.

.. _text format: https://prometheus.io/docs/instrumenting/exposition_formats/
.. _Prometheus: https://prometheus.io
.. _Prometheus Configuration Documentation: https://prometheus.io/docs/prometheus/latest/configuration/configuration/
.. _Alertmanager: https://prometheus.io/docs/alerting/latest/overview/
.. _Integrations Documentation: https://prometheus.io/docs/operating/integrations/#alertmanager-webhook-receiver
.. _Grafana: https://grafana.com
.. _Prometheus data source: https://grafana.com/docs/grafana/latest/features/datasources/prometheus/
.. _Grafana download page: https://grafana.com/grafana/download