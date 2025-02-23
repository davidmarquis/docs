---
title: "Receiving Updates & Callbacks"
excerpt: ""
---
Message status updates, events and messages from the RCS REST API and user agents, are delivered to the API client using webhook callbacks.

All types of callbacks are sent to the same HTTP(S) endpoint registered in the RCS REST API. See [Registering Webhook URI](#section-registering-webhook-uri) for details on how to register a webhook URI.

Callbacks are categorized as:

 * [Status Reports](#section-status-reports), sent every time the status of a sent message changes
 * [User Agent Events](#section-user-agent-events), sent for events generated by the user agent, e.g., composing notifications
 * [User Agent Messages](#section-user-agent-messages), sent whenever the user agent manually sends a message (text/image/location/etc.) or interacts with sent suggestion chips.

A callback is a HTTP POST request with a notification made by the RCS REST API to a URI of your choosing. The RCS REST API expects the receiving server to respond with a response code within the 2xx Success range. If no successful response is received then the RCS REST API will either schedule a retry if the error is expected to be temporary or discard the callback if the error seems to be permanent.

### Registering Webhook URI

Please contact Sinch to update the webhook URL.

### Callback Request

Callbacks are sent as a JSON encoded HTTP POST requests to the agent-specific webhook URI (See [Registering Webhook URI](#section-registering-webhook-uri)):

`POST {agent_callback_uri}`

### Callback Payload

The callback payload format is as such:

**JSON Representation**
```json
{
  "type": enum(
    "status_report_rcs",
    "user_agent_event_rcs",
    "user_agent_message_rcs"
  ),
  ... // type specific fields
}
```


#### Fields

Depending on the type of callback the payload will be one of (as indicated by `type` field):

 * [StatusReport](#section-status-reports)
 * [UserAgentEvent](#section-user-agent-events)
 * [UserAgentMessage](#section-user-agent-messages)

#### Examples

##### Successfully delivery RCS message
```json
{
  "type": "status_report_rcs",
  "message_id": "bc6776ee-7bde-4d6e-9c1e-102e87f92520",
  "at": "2017-10-31T13:06:30Z",
  "status_report": {
    "type": "delivered"
  }
}
```

##### User agent taps a suggested reply chip
```json
{
  "type": "user_agent_message_rcs",
  "message_id": "9lkj32asd712jkasdjkasdkhsadasd",
  "from": "4655123456",
  "message": {
    "type": "suggestion_response",
    "postback_data": "ef425f90-a27b-4956-9ba0-37cdd2d6e160_YES_CHIP",
    "text": "Yes"
  }
}
```


### Status reports

#### Status descriptions

| Status                       | Description                                                                                                                                                         |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| queued                       | Message has entered the RCS REST API system. This is the initial state for a message.                                                                               |
| aborted                      | Message expired or was revoked, and no fallback SMS was requested.                                                                                                  |
| failed                       | Message failed to be dispatched as RCS, and fallback SMS also failed.                                                                                               |
| delivered                    | Message was successfully delivered to the handset.                                                                                                                  |
| displayed                    | Message was displayed on the handset.                                                                                                                               |
| sms_failed                   | Message was not sent as RCS message, a fallback SMS has been dispatched and subsequently failed delivery. See [SMS Fallback](doc:rcs-rest-sms-fallback).            |
| sms_delivered                | Message was not sent as RCS message, a fallback SMS has been dispatched and was subsequently delivered successfully. See [SMS Fallback](doc:rcs-rest-sms-fallback). |


#### JSON Models

Detailed description of all available status report payloads

##### StatusReport

JSON Representation
```json
{
  "type": "status_report_rcs",
  "message_id": string,
  "at": timestamp,
  "status_report": {
    "type": enum(
      "queued",
      "aborted",
      "failed",
      "delivered",
      "displayed"
    ),
    ... // type specific fields
  }
}
```

<div class="magic-block-html">
    <div class="marked-table">
        <table>
            <thead>
            <tr class="header">
                <th>Field</th>
                <th>Type</th>
                <th>Description</th>
                <th>Default</th>
                <th>Constraints</th>
                <th>Required</th>
            </tr>
            </thead>
            <tbody>
            <tr class="odd">
                <td>type</td>
                <td>string</td>
                <td>Static string 'status_report_rcs'</td>
                <td>N/A</td>
                <td>N/A</td>
                <td>Yes</td>
            </tr>
            <tr class="even">
                <td>message_id</td>
                <td>string</td>
                <td>Message id for which this status report is relevant</td>
                <td>No</td>
                <td>^[0-9a-f]{8}-[0-9a-f]{4}-[1-5][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$</td>
                <td>Yes</td>
            </tr>
            <tr class="odd">
                <td>at</td>
                <td>string</td>
                <td>Timestamp of then the status report was created in the Sinch service</td>
                <td>No</td>
                <td>A timestamp in RFC3339 UTC "Zulu" format</td>
                <td>Yes</td>
            </tr>
            <tr class="even">
                <td>status_report</td>
                <td><dl>
                    <dt>oneOf:</dt>
                    <dd><ul>
                        <li>object(<code class="interpreted-text" data-role="ref">StatusReportQueued</code>)</li>
                        <li>object(<code class="interpreted-text" data-role="ref">StatusReportAborted</code>)</li>
                        <li>object(<code class="interpreted-text" data-role="ref">StatusReportFailed</code>)</li>
                        <li>object(<code class="interpreted-text" data-role="ref">StatusReportDelivered</code>)</li>
                        <li>object(<code class="interpreted-text" data-role="ref">StatusReportDisplayed</code>)</li>
                    </ul>
                    </dd>
                </dl></td>
                <td>Object describing the status report. The type of report is identified by the type property.</td>
                <td>No</td>
                <td>N/A</td>
                <td>Yes</td>
            </tr>
            </tbody>
        </table>
    </div>
</div>

##### StatusReportQueued
```json
{
  "type": "queued"
}
```


###### Fields

| Field | Type   | Description            | Default | Constraints | Required |
| ----- | ------ | ---------------------- | ------- | ----------- | -------- |
| type  | string | Static string 'queued' | N/A     | N/A         | Yes      |


##### StatusReportAborted
```json
{
  "type": "aborted",
  "revoked": boolean,
  "expired": boolean
}
```


###### Fields

| Field   | Type    | Description                       | Default | Constraints | Required |
| ------- | ------- | --------------------------------- | ------- | ----------- | -------- |
| type    | string  | Static string 'aborted'           | N/A     | N/A         | Yes      |
| revoked | boolean | Has the RCS message been revoked? | No      | N/A         | Yes      |
| expired | boolean | Has the RCS message expired?      | No      | N/A         | Yes      |

##### StatusReportFailed
```json
{
  "type": "failed",
  "revoked": boolean,
  "expired": boolean,
  "code": integer,
  "reason": string
}
```

###### Fields

| Field   | Type    | Description                                   | Default | Constraints | Required |
| ------- | ------- | --------------------------------------------- | ------- | ----------- | -------- |
| type    | string  | Static string 'failed'                        | N/A     | N/A         | Yes      |
| revoked | boolean | Has the RCS message been revoked?             | No      | N/A         | Yes      |
| expired | boolean | Has the RCS message been expired?             | No      | N/A         | Yes      |
| code    | integer | Error code of agent error causing the failure | No      | N/A         | Yes      |
| reason  | string  | Descriptive message of why the message failed | No      | N/A         | Yes      |

###### Error code values

| Code | Description                                                                                 |
| :--: | ------------------------------------------------------------------------------------------- |
|   1  | An internal error occurred in the gateway                                                   |
|   2  | The mobile subscriber device was reported to have no RCS capabilities                       |
|   3  | The mobile subscriber device does not support the capabilities required to send the message |
|   4  | The operator has barred this chatbot from this mobile subscriber                            |
|   5  | The operator has determined that this mobile subscriber number cannot be found              |
|   6  | Uploading of media to supplier failed                                                       |
|   7  | The operator reported a delivery failure asynchronously                                     |
|   8  | The API request was not formatted correctly                                                 |
|   9  | No provisioned supplier could be found to service request                                   |
|  10  | The requested supplier has not been provisioned                                             |
|  11  | The supplier reported a systems error                                                       |
|  12  | The throttle limit has been reached                                                         |




##### StatusReportDelivered
```json
{
  "type": "delivered"
}
```


###### Fields

| Field | Type   | Description               | Default | Constraints | Required |
| ----- | ------ | ------------------------- | ------- | ----------- | -------- |
| type  | string | Static string 'delivered' | N/A     | N/A         | Yes      |

##### StatusReportDisplayed
```json
{
  "type": "displayed"
}
```


###### Fields

| Field | Type   | Description               | Default | Constraints | Required |
| ----- | ------ | ------------------------- | ------- | ----------- | -------- |
| type  | string | Static string 'displayed' | N/A     | N/A         | Yes      |

### User Agent Events

Detailed descriptions of all available user agent event payloads.

#### JSON Models

##### UserAgentEvent
```json
{
  "type": "user_agent_event_rcs",
  "from": MSIDSN,
  "event": {
    "type": enum("composing"),
    ... // type specific fields
  }
}
```


###### Fields

<div class="magic-block-html">
    <div class="marked-table">
        <table>
            <thead>
            <tr class="header">
                <th>Field</th>
                <th>Type</th>
                <th>Description</th>
                <th>Default</th>
                <th>Constraints</th>
                <th>Required</th>
            </tr>
            </thead>
            <tbody>
            <tr class="odd">
                <td>type</td>
                <td>string</td>
                <td>Static string 'user_agent_event_rcs'</td>
                <td>N/A</td>
                <td>N/A</td>
                <td>Yes</td>
            </tr>
            <tr class="even">
                <td>from</td>
                <td>string</td>
                <td>MSISDN of the device that sent the event</td>
                <td>No</td>
                <td>No</td>
                <td>Yes</td>
            </tr>
            <tr class="odd">
                <td>event</td>
                <td><dl>
                    <dt>oneOf:</dt>
                    <dd><ul>
                        <li>object(<code class="interpreted-text" data-role="ref">UserAgentEventComposing</code>)</li>
                    </ul>
                    </dd>
                </dl></td>
                <td>Object describing the event</td>
                <td>No</td>
                <td>N/A</td>
                <td>Yes</td>
            </tr>
            </tbody>
        </table>
    </div>
</div>

##### UserAgentEventComposing
```json
{
  "type": "composing"
}
```


###### Fields

| Field | Type   | Description                                                                     | Default | Constraints | Required |
| ----- | ------ | ------------------------------------------------------------------------------- | ------- | ----------- | -------- |
| type  | string | Static string 'composing'. This type notifies the agent that the user is typing | N/A     | N/A         | Yes      |

### User Agent Messages

Detailed descriptions of all available user agent message payloads

#### JSON Models

##### UserAgentMessage
```json
{
  "type": "user_agent_message_rcs",
  "message_id": string,
  "from": MSISDN,
  "message": {
    "type": enum(
      "text",
      "file",
      "suggestion_response",
      "location"
    ),
    ... // type specific fields
  }
}
```


###### Fields

<div class="magic-block-html">
    <div class="marked-table">
        <table>
            <thead>
            <tr class="header">
                <th>Field</th>
                <th>Type</th>
                <th>Description</th>
                <th>Default</th>
                <th>Constraints</th>
                <th>Required</th>
            </tr>
            </thead>
            <tbody>
            <tr class="odd">
                <td>type</td>
                <td>string</td>
                <td>Static string 'user_agent_message_rcs'</td>
                <td>N/A</td>
                <td>N/A</td>
                <td>Yes</td>
            </tr>
            <tr class="even">
                <td>message_id</td>
                <td>string</td>
                <td>Unique message id for this user message</td>
                <td>No</td>
                <td>No</td>
                <td>Yes</td>
            </tr>
            <tr class="odd">
                <td>from</td>
                <td>string</td>
                <td>MSISDN of the device that sent the message</td>
                <td>No</td>
                <td>No</td>
                <td>Yes</td>
            </tr>
            <tr class="even">
                <td>message</td>
                <td><dl>
                    <dt>oneOf:</dt>
                    <dd><ul>
                        <li>object(<code class="interpreted-text" data-role="ref">TextMessage</code>)</li>
                        <li>object(<code class="interpreted-text" data-role="ref">FileMessage</code>)</li>
                        <li>object(<code class="interpreted-text" data-role="ref">SuggestionResponse</code>)</li>
                        <li>object(<code class="interpreted-text" data-role="ref">LocationMessage</code>)</li>
                    </ul>
                    </dd>
                </dl></td>
                <td>The message content</td>
                <td>No</td>
                <td>N/A</td>
                <td>Yes</td>
            </tr>
            </tbody>
        </table>
    </div>
</div>

##### SuggestionResponse
```json
{
  "type": "suggestion_response",
  "postback_data": string,
  "text": string
}
```


###### Fields

| Field          | Type   | Description                                                                                      | Default | Constraints | Required |
| -------------- | ------ | ------------------------------------------------------------------------------------------------ | ------- | ----------- | -------- |
| type           | string | Static string 'suggestion\_response'                                                             | N/A     | N/A         | Yes      |
| postback\_data | string | The postback data that was supplied in the object(`SuggestedReply`) or object(`SuggestedAction`) | No      | No          | No       |
| text           | string | The text that was supplied in the object(`SuggestedReply`) or object(`SuggestedAction`)          | No      | No          | No       |

At least one of `postback_data` and `text` will be set.

##### LocationMessage
```json
{
  "type": "location",
  "latitude": number,
  "longitude": number
}
```


###### Fields

| Field     | Type   | Description                | Default | Constraints                  | Required |
| --------- | ------ | -------------------------- | ------- | ---------------------------- | -------- |
| type      | string | Static string 'location'   | N/A     | N/A                          | Yes      |
| latitude  | number | Latitude of user location  | No      | MinValue: -90 MaxValue: 90   | Yes      |
| longitude | number | Longitude of user location | No      | MinValue: -180 MaxValue: 180 | Yes      |
| latitude  | number | Latitude of user location  | No      | MinValue: -90 MaxValue: 90   | Yes      |
| longitude | number | Longitude of user location | No      | MinValue: -180 MaxValue: 180 | Yes      |


<a class="gitbutton pill" target="_blank" href="https://github.com/sinch/docs/blob/master/docs/rcs/rcs-http-rest/rcs-rest-receiving-updates-callbacks.md"><span class="fab fa-github"></span>Edit on GitHub</a>