![Fastly WAF](/images/fastly-waf.png)
# Fastly WAF - BigQuery & Looker visualization example
## Fastly Edge Security Solutions team

## Intro

Visibility is the foundation of an effective defensive strategy. The ability to capture security events with minimal latency, enables rapid mitigation of attacks and other threats, and therefore helps minimise the potential damage that can result from such attacks.

Fastly’s [real-time streaming logs](https://docs.fastly.com/en/guides/about-fastlys-realtime-log-streaming-features) give you the ability to capture vital log data in near real time, allowing you to respond to anomalies as and when they occur.

Capturing log data is a great starting point. You’ll get a lot of information! But on its own, it is not enough to actually protect against the array of threats out there. Fastly’s [WAF](https://docs.fastly.com/en/guides/web-application-firewall) (Web Application Firewall) takes the OWASP [Core Rule Set](https://owasp.org/www-project-modsecurity-core-rule-set/) combined with commercial resources and our own research to offer more comprehensive protection.

Customizable dashboards allow you to focus only on what matters to you. You can rapidly respond to request patterns that are foreshadowing a breach and update your configuration in seconds. 

There are several choices for third-party log streaming tools available and Fastly supports many of them (full list [here](https://docs.fastly.com/en/guides/setting-up-remote-log-streaming)).

However, in this repo we're going to focus on:
 
 - configuring a Fastly service to stream HTTP request logs and WAF event logs to BigQuery
 - configuring Looker to create visualizations of the log data
 
You can find further details in the following blog post: https://www.fastly.com/blog

>High Level Architecture
![High Level Architecture](/images/high_level_architecture.png)

### BigQuery configuration

Choosing where to store your logs is a balance between concerns like cost, analytical features, retention options, schema flexibility, etc. The combination of a petabyte scale, serverless solution with powerful, SQL-based analytical capabilities makes BigQuery a compelling option. In this example we’ll be streaming logs directly into BigQuery.

There are two sources of real-time log data that you will want to send to BigQuery once the Fastly WAF is configured. You are already familiar with [Request logs](/fastly/request_logs_format.json). These logs have variables that provide data about the size, time, and geolocation of a request.

The second source of log data is the [WAF log](/fastly/waf_logs_format.json). This is where logging variables specific to the WAF (Rule IDs, severity of the rule matched, and action taken) become available. More information on the WAF log variables [here](https://docs.fastly.com/en/guides/fastly-waf-logging#using-waf-specific-variables).

It is not unusual for a single HTTP request to trigger multiple WAF log events. Using the following schema in BigQuery allows us to match any WAF event with the corresponding Request log.

> BigQuery Dataset Model
![BigQuery Architecture](/images/bigquery_architecture.png)

* The `bq` CLI app can be used to create the tables from the schema included in this repo. For the request logs:

`bq mk --table project_id:dataset.table bigquery/request_logs_schema.json`

* For the waf logs:

`bq mk --table project_id:dataset.table bigquery/waf_logs_schema.json`

* For additional `bq` parameters, see: 

https://cloud.google.com/bigquery/docs/bq-command-line-tool

### Fastly configuration

This [guide](https://docs.fastly.com/en/guides/log-streaming-google-bigquery) walks you through setting up a BigQuery logging endpoint on your Fastly service. You will need to create two logging endpoints, the first one for request logs and the second for WAF logs.

The second endpoint follows the same steps for table creation that are outlined, but with the WAF variables instead. 

Note: Adding the WAF endpoint requires the logging parameter to be moved to the `waf_debug_log` VCL subroutine. You can follow the instructions [here](https://docs.fastly.com/en/guides/fastly-waf-logging#using-the-web-interface) to set up the WAF log endpoint.

The log format for each log configuration can be found in the [`fastly`](/fastly/) folder

### Looker configuration

[Looker](https://looker.com/product/visualizations) is a powerful analytics and visualization tool that enables you to explore, analyze, and share real-time data seamlessly. It gives you the ability to query data against most of the databases, including Google’s BigQuery, and present findings in a dashboard to facilitate data analysis and security responses.

Combined with the Fastly's WAF, it empowers you to create customizable dashboards to illustrate key application-layer attacks, such as SQL injection attacks, cross site scripting, HTTP protocol violations and other OWASP Top 10 threats.

These dashboards can be leveraged to rapidly identify trends in traffic, breakdown of attacks over time, sudden surge in traffic from a given attacker and malicious patterns. These patterns can then be utilized to respond to threats by taking advantage of the Fastly's instant configuration change through ACL, custom VCL code or WAF rules.

>Looker Architecture
![Looker Architecture](/images/looker_architecture.png)

1. In Looker, link together the `request_logs` and the `waf_logs` from the dataset in BigQuery by navigating to Admin -> Connections -> New Connection. Follow documentation [here](https://docs.looker.com/setup-and-management/database-config/google-bigquery).
   - Tip: [Development Mode](https://docs.looker.com/data-modeling/getting-started/dev-mode-prod-mode) allows you to make changes to projects without interfering with other users.
  
2. Create the `waf_demo` LookML Project. See the [Creating a New LookML Project](https://docs.looker.com/data-modeling/getting-started/create-projects) documentation page for more information.

3. Generate the `waf_demo` model [automatically](https://docs.looker.com/data-modeling/getting-started/connect-to-db-and-generate-model) or by using the Fastly [model template](/looker/models/)

4. Create both `request_logs` and `waf_logs` view files by using the Fastly [view files templates](/looker/views/).

5. Generate the WAF dashboard by following [this](https://docs.looker.com/dashboards/creating-lookml-dashboards#creating_a_lookml_dashboard_file) Looker documentation and using the Fastly [LookML dashboard template](/looker/dashboards/).

### WAF Dashboard example

Click [here](/dashboard_examples/DASHBOARD-EXAMPLES.md) to see more examples of WAF dashboard.

![WAF Dashboard demo](/images/waf_dashboard_demo.gif)

### Contributing

We welcome pull requests for issues and new functionality. Please see [Contributing](documentation/CONTRIBUTING.md) for more details.
