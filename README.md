# twilio-sms Viam modular resource

This module implements the [RDK generic API](https://github.com/rdk/generic-api) in a mcvella:messaging:twilio-sms service model.
With this model, you can:

- Send SMS and MMS (media) text messages via a [Twilio](https://www.twilio.com/) phone number.
- Retrieve received messages from a Twilio phone number, optionally filtered by date range and sender.

## Requirements

You must have a registered Twilio account with an account SID and auth token.
In order to send media (MMS), you must also set up a [Twilio service](https://console.twilio.com/us1/develop/functions/services) and provide the service SID.

## Build and run

To use this module, follow the instructions to [add a module from the Viam Registry](https://docs.viam.com/registry/configure/#add-a-modular-resource-from-the-viam-registry) and select the `rdk:generic:mcvella:messaging:twilio-sms` model from the [`mcvella:messaging:twilio-sms` module](https://app.viam.com/module/rdk/mcvella:messaging:twilio-sms).

## Configure your Twilio SMS service

> [!NOTE]  
> Before configuring your Twilio service, you must [create a machine](https://docs.viam.com/manage/fleet/machines/#add-a-new-machine).

Navigate to the **Config** tab of your machine's page in [the Viam app](https://app.viam.com/).
Click on the plus icon and choose **Service**.
Select the `generic` type, then select the `mcvella:messaging:twilio-sms` model.
Click **Add module**, then enter a name for your Twilio SMS service and click **Create**.

On the new component panel, copy and paste the following attribute template into your generic's **Attributes** box:

```json
{
  "account_sid": "<your Twilio account SID>",
  "auth_token": "<your Twilio auth token>",
  "media_sid": "<your Twilio service SID if sending MMS>",
  "environment_sid": "<your Twilio environment SID if reusing an existing environment for media uploads, optional>",
  "default_from": "<default from phone number, optional>",
  "store_log_in_data_management": false,
  "app_api_key": "daifdkaddkdfhshfeasw",
  "app_api_key_id": "weygwegqeyygadydagfd",
  "organization_id": "abc123asssa",
  "part_id": "9dosadsaewad"
}
```

> [!NOTE]  
> For more information, see [Configure a Machine](https://docs.viam.com/manage/configuration/).

### Attributes

The following attributes are available for `rdk:generic:mcvella:messaging:twilio-sms`:

| Name | Type | Inclusion | Description |
| ---- | ---- | --------- | ----------- |
| `account_sid` | string | **Required** |  Your Twilio account SID. |
| `auth_token` | string | **Required** |  Your Twilio auth token. |
| `media_sid` | string | Optional |  Your Twilio service SID, if you plan on sending local media. |
| `environment_sid` | string | Optional |  Your Twilio environment SID, if you want to reuse an existing environment for media uploads. If not provided, a new environment will be created and deleted for each media upload. |
| `default_from` | string | Optional |  Default Twilio phone number to send from, optional as it can be passed on each send request. |
| `preset_messages` | object | Optional|  A set of key (preset name) and value (preset message body) pairs that can be used to send pre-configured text. Template strings can be embedded within double angle brackets, for example: <<to_replace>> |
| `enforce_preset` | boolean | Optional, default false |  If set to true, preset_messages must be configured and a preset message must be selected when sending. |
| `store_log_in_data_management` | boolean | Optional, default false |  If set to true, will run a background loop storing Twilio log data in data management for use by the [get](#get) command, which in some cases help avoid Twilio rate limiting. |
| `app_api_key` | string | Optional, required if store_log_in_data_management is true | Viam app api key. |
| `app_api_key_id` | string | Optional, required if store_log_in_data_management is true | Viam app api key ID. |
| `organization_id` | string | Optional, required if store_log_in_data_management is true | Viam organization ID for which to store log data. |
| `part_id` | string | Optional, required if store_log_in_data_management is true | Part ID for which to store log data. |

### Example configuration

```json
{
  "account_sid": "abc123adskjsd32lf23op",
  "auth_token": "821ssdaodsd2aspods9k2",
  "media_sid": "ms923odofdsopkfdsokd",
  "environment_sid": "ev123odofdsopkfdsokd",
  "default_from": "18775550123",
  "preset_messages": {
    "alert": "This is an alert message about <<about>>"
  }
}
```

## API

The Twilio SMS service provides the [DoCommand](https://docs.viam.com/services/generic/#docommand) method from Viam's built-in [rdk:service:generic API](https://docs.viam.com/services/generic/)

### do_command(*dictionary*)

In the dictionary passed as a parameter to do_command(), you must specify a *command* by passing a the key *command* with one of the following values.

#### send

When *send* is passed as the command, an SMS will be sent via the configured Twilio account.
The following may also be passed:

| Key | Type | Inclusion | Description |
| ---- | ---- | --------- | ----------- |
| `to` | string | **Required** |  The phone number to send the message to. |
| `body` | string | Optional |  The message text. |
| `from` | string | Optional |  The twilio phone number from which to send the message. If not specified, will use *default_from*, if configured. |
| `media_path` | string | Optional |  A path on the Viam machine of a media file to send with the message.  If this is specified, *media_sid* must be configured. |
| `media_url` | string | Optional |  A publicly reachable URL for media to send with the message. |
| `media_base64` | string | Optional |  A base64 encoded ascii string for media to send with the message. If passed, media_mime_type must also be set. |
| `media_mime_type` | string | Optional |  The MIME type of the base64 encoded ascii string for media to send with the message. |
| `preset` | string | Optional |  The name of a configured preset message, configured with preset_messages.  If the service is configured with enforce_preset=true, this becomes required. |
| `template_vars` | object | Optional | A key/value pair of template parameter names and values to insert into preset messages. |

Example:

```python
sms.do_command({"command": "send", "body": "Hello there", "to": "5551234567"})
```

#### get

When *get* is passed as the command, messages received by the configured Twilio account will be retrieved, in LIFO order.
The following may also be passed:

| Key | Type | Inclusion | Description |
| ---- | ---- | --------- | ----------- |
| `number` | int | Optional |  The number of messages to retrieve, default 5, max 1000. |
| `from` | string | Optional|  Filter messages by the phone number sent from. |
| `to` | string | Optional |  Filter messages by the Twilio phone number received to. |
| `time_start` | string | Optional |  Filter messages received on or after this time in "%d/%m/%Y %H:%M:%S" format. |
| `time_end` | string | Optional |  Filter messages received on or before this time in "%d/%m/%Y %H:%M:%S" format. |

Example:
```python
sms.do_command({"command": "get", "from": "5551234567"})
```

Note that Twilio rate-limits message retrieval, so if you are building a solution that requires realtime notifications of messages, consider using Twilio webhooks instead of [get](#get), or leverage the *store_log_in_data_management* feature.
