Event Notification
------------------

RackHD support event notification via both AMQP and web hook.

Web hook allow user's application to subscribe certain RackHD published events published by configured URL, when one of the subscribed events is triggered, RackHD will send a HTTP POST request with payload to configured URL.

AMQP allow users don't need to register webhook URL, all RackHD defined events will be published, but users need to filter events in user side.

Events Payloads
~~~~~~~~~~~~~~~~~~~~

.. _common_event_payload_format:

All published external events' payload formats are common, the event attributes are as below:

========= ====== =================================
Attribute Type   Description
========= ====== =================================
version   String Event payload format version.
type      String It could be one of the values: **heartbeat**, **node**, **polleralert**, **graph**.
action    String The event happened, which is performed by the `type` attribute.
severity  String Event severity, it could be one of the values: **critical**, **warning**, **information**.
typeId    String It could be graph's Id, poller's Id, or service name, or filename, etc. which is performed by the `type` attribute.
createdAt String The time event happened.
nodeId    String The node `Id`, it's `null` for 'heartbeat' event.
data      Object Detail information are included in this attribute.
========= ====== =================================

.. _table:

The table for `type`, `typeId`, `action` and `severity` for all external events

+--------------+------------------------+------------------------+----------------+-----------------------------------+
| *type*       | *typeId*               | *action*               | *severity*     | Description                       |
|              |                        |                        |                |                                   |
+==============+========================+========================+================+===================================+
| heartbeat    | <fqdn>.<service name>  | updated                | information    | Each running RackHD service will  |
|              |                        |                        |                | publish a periodic heartbeat      |
|              |                        |                        |                | event message to notify that      |
|              |                        |                        |                | service is running.               |
+--------------+------------------------+------------------------+----------------+-----------------------------------+
| polleralert  | the 'Id' of poller     | sel.updated            | related to sel | Triggered when sel information    |
|              |                        |                        | rules, it      | is updated.                       |
|              |                        |                        | could be one   |                                   |
|              |                        |                        | of the values: |                                   |
|              |                        |                        | critical,      |                                   |
|              |                        |                        | warning,       |                                   |
|              |                        |                        | information    |                                   |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | sdr.updated            | information    | Triggered when sdr information    |
|              |                        |                        |                | is updated.                       |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | fabricservice.updated  | information    | Triggered when fabricservice      |
|              |                        |                        |                | information is updated.           |
+--------------+------------------------+------------------------+----------------+-----------------------------------+
| graph        | the 'Id' of graph      | started                | information    | Triggered when graph started.     |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | finished               | information    | Triggered when graph finished.    |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | progress.updated       | information    | Triggered when long task's        |
|              |                        |                        |                | progress information is updated.  |
+--------------+------------------------+------------------------+----------------+-----------------------------------+
| node         | the 'Id' of node       | discovered             | information    | Triggered in node's               |
|              |                        |                        |                | discovery process,it has          |
|              |                        |                        |                | two cases:                        |
|              |                        |                        |                |                                   |
|              |                        |                        |                | - Automatic discovery             |
|              |                        |                        |                | - Passive discovery by            |
|              |                        |                        |                |   post a node by REST API         |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | removed                | information    | Triggered when node is            |
|              |                        |                        |                | deleted by REST API               |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | sku.assigned           | information    | Triggered when node's `sku`       |
|              |                        |                        |                | field is assigned.                |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | sku.unassigned         | information    | Triggered when node's `sku`       |
|              |                        |                        |                | field is unassigned.              |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | sku.updated            | information    | Triggered when node's `sku`       |
|              |                        |                        |                | field is updated.                 |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | obms.assigned          | information    | Triggered when node's `obms`      |
|              |                        |                        |                | field is assigned.                |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | obms.unassigned        | information    | Triggered when node's `obms`      |
|              |                        |                        |                | field is unassigned.              |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | obms.updated           | information    | Triggered when node's `obms`      |
|              |                        |                        |                | field is updated.                 |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | accessible             | information    | Triggered when node telemetry     |
|              |                        |                        |                | OBM service (IPMI or SNMP) is     |
|              |                        |                        |                | accessible                        |
|              |                        |                        |                |                                   |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | inaccessible           | information    | Triggered when node telemetry     |
|              |                        |                        |                | OBM service (IPMI or SNMP) is     |
|              |                        |                        |                | inaccessible                      |
+--------------+------------------------+------------------------+----------------+-----------------------------------+


Example of heartbeat event payload:

.. code-block:: JSON

    {
        "version": "1.0",
        "type": "heartbeat",
        "action": "updated",
        "typeId": "kickseed.example.com.on-tftp",
        "severity": "information",
        "createdAt": "2016-07-13T14:23:45.627Z",
        "nodeId": "null",
        "data": {
            "delivery_info": {
                "consumer_tag": "None1",
                "delivery_tag": 2479,
                "exchange": "on.heartbeat",
                "redelivered": false,
                "routing_key": "kickseed.example.com.on-tftp"
            },
            "message": {
                "value": {
                    "name": "on-tftp",
                    "pid": 26633,
                    "lastUpdate": "2016-07-13T14:23:35.784Z",
                    "currentTime": "2016-07-13T14:23:45.627Z",
                    "nextUpdate": "2016-07-13T14:23:55.627Z",
                    "cpuUsage": {
                        "system": 92810,
                        "user": 1545498
                    },
                    "memoryUsage": {
                        "heapTotal": 71938048,
                        "heapUsed": 45444240,
                        "rss": 98926592
                    },
                    "platform": "linux",
                    "release": {
                        "headersUrl": "https://nodejs.org/download/release/v6.3.0/node-v6.3.0-headers.tar.gz",
                        "name": "node",
                        "sourceUrl": "https://nodejs.org/download/release/v6.3.0/node-v6.3.0.tar.gz"
                    },
                    "title": "node",
                    "uid": 0,
                    "versions": {
                        "ares": "1.10.1-DEV",
                        "http_parser": "2.7.0",
                        "icu": "57.1",
                        "modules": "48",
                        "node": "6.3.0",
                        "openssl": "1.0.2h",
                        "uv": "1.9.1",
                        "v8": "5.0.71.52",
                        "zlib": "1.2.8"
                    }
                }
            },
            "properties": {
                "content_type": "application/json",
                "type": "Result"
            }
        }
    }

Events via AMQP
~~~~~~~~~~~~~~~~~~~~

AMQP Exchange and Routing Key
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The change of resources managed by RackHD could be retrieved from AMQP messages.

- Exchange: **on.events**
- Routing Key **<type>.<action>.<severity>.<typeId>.<nodeId>**

ALl the fields in routing key exists in the common event payloads common_event_payload_format_.

Examples of routing key:

Heartbeat event routing key of on-tftp service:

.. code-block:: REST

    heartbeat.updated.information.kickseed.example.com.on-tftp

Polleralert sel event routing key:

.. code-block:: REST

    polleralert.sel.updated.critical.44b15c51450be454180fabc.57b15c51450be454180fa460

Node discovered event routing key:

.. code-block:: REST

    node.discovered.information.57b15c51450be454180fa460.57b15c51450be454180fa460

Graph event routing key:

.. code-block:: REST

    graph.started.information.35b15c51450be454180fabd.57b15c51450be454180fa460


AMQP Routing Key Filter
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All the events could be filtered by routing keys, for example:

All services' heartbeat events:

.. code-block:: Bash

    $ sudo node sniff.js "on.events" "heartbeat.#"

All nodes' discovered events:

.. code-block:: Bash

    $ sudo node sniff.js "on.events" "#.discovered.#"

'sniff.js' is a tool located at https://github.com/RackHD/on-tools/blob/master/dev_tools/README.md


Events via Hook
~~~~~~~~~~~~~~~~~~~~

URL could be registered by API below, and the filter conditions also could be specified in the payload

.. code-block:: REST

    curl -H "Content-Type: application/json" -X POST -d @payload.json <server>/api/2.0/hooks

The payload.json format are as below:

========= ====== ============ ============================================
Attribute Type   Flags        Description
========= ====== ============ ============================================
url       String **required** The hook url that events are notified to.
name      String **optional** Any name user specified.
filters   Array  **optional** The conditions of which events are notified.
========= ====== ============ ============================================

Example of payload

.. code-block:: JSON

    {
        "name": "all events of node 57b15c51450be454180fa460",
        "url": "http://www.abc.com/def",
        "filters": [
            {
                "type": "node",
                "typeId": "*",
                "action": "*",
                "severity": "information",
                "nodeId": "57b15c51450be454180fa460"
            }
        ]
    }

