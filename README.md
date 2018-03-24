# Elastabot

Slack bot that listens for commands from Slack users to:

* Acknowledge an alert created by elastalert
* Triage alerts or arbitrary issues

## Acknowledge Elastalerts

When told to ack an alert generated by Elastalert, Elastabot will look for the alert and silence it by creating a silence document in the appropriate Elasticsearch index. Additionally, if the ack command includes a question mark, ?, then the alert will be sent through the triage process.

## Triage

Elastabot understands the vague notion of a triage command. Currently, this is simply the generation of an SMTP email. This is useful for pushing issues into a ticketing system, such as Atlassian's JIRA tool, etc, and avoids complexities of direct integration to those tools such as how to handle downtime of the tool itself, or licensing costs of additional service users.

## Slack Commands

Users will interact with Elastabot in the Slack interface. A Slack community admin will need to register a bot for Elastabot and provide bot token needed for Elastabot to connect to Slack. Once Elastabot connects to Slack, users can invite the bot into one or more channels, or send direct messages to interaction with Elastabot.

### Commands

To see a list of available commands, users can type:

```slack
!help
```

Or, to get detailed help on a specific command, user can type:

```slack
!ack help
```

## Configuration

Elastabot expects two groups of configuration inputs.

1. JSON configuration file with non-sensitive values
2. Environment variables with sensitive values

### JSON Configuration

An example configuration file is shown below, followed by descriptions of each setting.

```json
{
  "elasticsearch": {
    "host": "elasticsearch",
    "port": 9200,
    "sslEnabled": false,
    "sslStrictEnabled": false,
    "timeoutSeconds": 10,
    "urlPrefix":""
  },
  "elastalert": {
    "index": "elastalert_status",
    "silenceMinutes": 240,
    "recentMinutes": 4320
  },
  "smtp": {
    "host": "email-smtp.us-east-1.amazonaws.com",
    "port": 587,
    "secure": false,
    "starttls": true,
    "timeoutSeconds": 4,
    "to": "jira@mycompany.atlassian.net",
    "from": "engineering_team@mycompany.invalid",
    "subjectPrefix": "[mini] ",
    "debug": false
  },
  "commandPrefix": "!",
  "triageTarget": "smtp"
}
```

| setting                        | description |
|--------------------------------|-------------|
| elasticsearch.host             | Hostname for the Elasticsearch server
| elasticsearch.port             | Port for the Elasticsearch server
| elasticsearch.sslEnabled       | If true, uses SSL/TLS to connect to Elasticsearch |
| elasticsearch.sslStrictEnabled | If true, the SSL/TLS certificates will be validated against known certificate authorities 
| elasticsearch.timeoutSeconds   | Number of seconds to wait for an Elasticsearch response
| elasticsearch.urlPrefix        | URL prefix for Elasticsearch, typically an empty string
| elastalert.index               | The index prefix used by Elastalert within Elasticsearch, typically elastalert or elastalert_status
| elastalert.silenceMinutes      | Number of minutes to silence an acknowledge alert if a silence duration is not explicitly given with the ack command.
| elastalert.recentMinutes       | Number of minutes to look back in history for a fired alert in the Elasticsearch index
| smtp.host                      | Hostname for the SMTP server
| smtp.port                      | Port for the SMTP server
| smtp.secure                    | If true, will connect to the SMTP host over SSL/TLS
| smtp.starttls                  | If true, will send the starttls command (typically not used with smtp.secure=true
| smtp.timeoutSeconds            | Number of seconds to wait for the SMTP server to respond
| smtp.to                        | Email address that will receive the triage email
| smtp.from                      | Sender email address
| smtp.subjectPrefix             | If non-empty string, will be prepended to each email subject
| smtp.debug                     | If true, the SMTP connectivity details will be logged to stdout
| commandPrefix                  | Special character or phrase to trigger the bot, typically an exclamation point, !. Ex: !ack
| triageTarget                   | How to initiate the triage process, currently only smtp is supported.

### Environment Variables

The following environment variables are used as inputs for sensitive information.

| variable               | required | description 
|------------------------|----------|------------
| SLACK_BOT_TOKEN        | true  | The Slack-generated bot token, provided by slack.com
| ELASTICSEARCH_USERNAME | false | Optional Elasticsearch username, provided by your ES admin
| ELASTICSEARCH_PASSWORD | false | Optional Elasticsearch password, provided by your ES admin
| SMTP_USERNAME          | false | Optioanl SMTP username, provided by your SMTP admin
| SMTP_PASSWORD          | false | Optioanl SMTP password, provided by your SMTP admin

## Docker

A Dockerfile is provided for Elastabot, and a Docker image will auto-build at hub.docker.com/jertel/elastabot.

The image will expect a configuration file to exist in the /opt/elastabot/elastabot.json location, so the recommended way to configure Elastabot is to use a file-based volume mount override from the host to this location.

Ex:

```bash
docker run --rm -v /host/path/elastabot.json:/opt/elastabot/elastabot.json jertel/elastabot
```

## Kubernetes

Elastabot was originally written for installation into a Kubernetes cluster via Helm. A sample chart is provided but no official prebuilt chart package is available at this time.