---

copyright:
years: 2016, 2017
lastupdated: "2016-12-12"

---

{:new_window: target="blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Developing application interfaces (Beta)
{: #im_index}

**_Important: You can develop application interfaces by using the Information Management feature. The Information Management feature is currently available only as a beta feature. Future updates might include changes that are incompatible with the current version of this feature. [Register to try it out](https://www.ibm.com/software/support/trial/cst/forms/nomination.wss?id=7050) and let us know what you think._**

Use the Information Management feature of {{site.data.keyword.iot_full}} to help you organize and integrate data that is coming into and going out of {{site.data.keyword.iot_short_notm}}. You might have different types, makes, or models of device that you want to connect to {{site.data.keyword.iot_short_notm}}, and these devices might publish data in differing formats. Use the Information Management feature to normalize incoming data and to simplify your applications by decoupling them from the complexities of how your specific devices are connected.

For example, you might want to control all the light bulbs in your facility by using an application on your phone or tablet. The light bulbs might be different brands and have different features. Some of the light bulbs can be dimmed, some can change colour, and others can simply be set to on or off. The Information Management feature lets you control all these types of bulb by using just one application, rather than by using several different applications that each control a different type of bulb.

The following diagram shows an example of two environment sensors. One sensor records temperature, and the other sensor records humidity. The temperature sensor publishes a temperature reading of `{ "t" : 34.5 }` to {{site.data.keyword.iot_short_notm}}. The humidity sensor publishes a humidity reading of `{ "h" : 52 }` to {{site.data.keyword.iot_short_notm}}.

![Mapping between temperature sensor devices and an application on {{site.data.keyword.iot_short_notm}}.](images/Information Management example.svg "Mapping between temperature sensor devices and an application on {{site.data.keyword.iot_short_notm}}")

The temperature and humidity readings are published as separate events to {{site.data.keyword.iot_short_notm}}. By using the Information Management feature, you can normalize these readings into a consistent form for processing and generate an outbound state change notification event. Your application can then process the data that is contained within this outbound event, without the need to understand how the data was published.

For more detailed information about using cURL to configure a similar scenario, see the [Example scenario](#scenario).

## Overview
{: #overview}

Information Management extends the current concept of [device type](#device_type) by adding a physical interface and an application interface to better control the data that flows through {{site.data.keyword.iot_short_notm}}.

The following diagram illustrates the logical mapping between a device and an application on {{site.data.keyword.iot_short_notm}}:

![The logical mapping between a device and an application on {{site.data.keyword.iot_short_notm}}.](images/im_mapping.png "The logical mapping between a device and an application on {{site.data.keyword.iot_short_notm}}")

Information Management refers to the concept of a device state. The device state consists of a set of properties that are defined by the application interface. The current values of these properties are stored in {{site.data.keyword.iot_short_notm}}, and are provided to an application either as a push event when new values are received from a device, or are made available to the application on request.

To process inbound events from a device and map them to specific properties, you must provide the following information to {{site.data.keyword.iot_short_notm}}:

- The structure of the inbound event. The [physical interface](#physical_interface) defines this information.

    To process inbound events from a device you need to define the structure and format of these events in the physical interface. For example, you might define temperature (t) and humidity (h) fields for an environment sensor in the physical interface. You might make all these fields numeric only. The fields 't' and 'h' map to properties that are defined by the application interface.  

- The structure of the desired device state. The [application interface](#application_interface) defines this information.

    The device state consists of a set of properties. For example, you might define a property called 'temperature' and a property called 'humidity' for your environment sensor. {{site.data.keyword.iot_short_notm}} stores the current values for these properties. The current value might change because of an inbound event that is published by your environment sensor.

- Information about how to map the inbound events to the preferred device state. The [mappings resource](#mappings) defines this information.

    To map the data that is contained in an inbound event which is defined in the physical interface to the appropriate properties that are defined on the application interface, you need to create a mappings resource. The mappings resource describes how to update the properties that are defined by the application interface in response to an inbound event from a device. If any of the properties in the application interface change, an outbound state change notification event might be sent to your application. This outbound event includes the current values of all the properties. Alternatively, your application might issue a request to get the current values of the device state. For example, you might map field 't' to the property 'temperature' and the field 'h' to the property 'humidity'. When your environment sensor publishes an inbound event that results in the current value of the properties 'temperature' and 'humidity' exceeding a certain value, your application might take a specified action.



## Data flow between devices and applications
{: #mapping}

The following flow diagram shows how the different resources in the Information Management feature are used:

![The flow between a device and an application on {{site.data.keyword.iot_short_notm}}.](images/Information Management flow.jpg "The flow between a device and an application on {{site.data.keyword.iot_short_notm}}")

The event data passes through several different resources that can be managed by using REST APIs. These resources are defined in [Key concepts](#key_concepts).

The following diagram illustrates how schemas are used in this flow:

![The flow between a device and an application on {{site.data.keyword.iot_short_notm}}.](images/Information Management schema.jpg "The flow between a device and an application on {{site.data.keyword.iot_short_notm}}")

JSON schemas are used to define and validate the format of incoming and outgoing events. For more information about the schemas that are used in Information Management, see [Schemas](#schema).


## Key concepts
{: #key_concepts}

The following resources are used within the Information Management feature. You can manage the resources that are illustrated in the previous diagrams by using REST APIs. For information about the Information Management REST APIs, see the Information Management section in the [{{site.data.keyword.iot_short_notm}} HTTP REST API](https://docs.internetofthings.ibmcloud.com/swagger/info-mgmt-beta.html) documentation.


### Physical interface
{: #physical_interface}

 Devices use the physical interface to publish events to {{site.data.keyword.iot_short_notm}}. The physical interface is associated with one or more event types. The physical interface validates the inbound events that are published by a device and maps those events to properties that are defined by the application interface. If the inbound event fails the validation check, applications can still subscribe to the event by using the existing MQTT process. For more information about subscribing to device events, see [MQTT connectivity for applications](https://console.ng.bluemix.net/docs/services/IoT/applications/mqtt.html#subscribe_device_events).

For example, you might have two devices that publish events to {{site.data.keyword.iot_short_notm}}. One device is a temperature sensor, and the other device monitors light levels. These devices publish readings to {{site.data.keyword.iot_short_notm}} by using a physical interface. In this scenario, you might want to associate your physical interface with two event types: one event type that represents temperature and another event type that represents light levels.

The physical interface maps these inbound events to different event types by using an event identifier. The event identifier corresponds to the MQTT topic that the event is published to.

The following topic provides an example of an event that has an event ID of '1234':
```
iot-2/evt/1234/fmt/json.
```

### Application interface
{: #application_interface}

The application interface defines the structure of the [device state](#overview) that is stored for a device object within {{site.data.keyword.iot_short_notm}} and validates the device state that is generated when an inbound event is processed.

An application interface must reference an application interface schema file that defines the structure of the properties that will be stored for the device. For example, only numeric values might be valid for temperature and humidity properties. At least one application interface must be associated with a device type before any mappings can be defined.

Custom cards and rules can subscribe to state change messages from the application interface by using existing methods. For more information about custom cards, see [Custom cards](https://console.ng.bluemix.net/docs/services/IoT/custom_cards/custom-cards.html). For more information about rules, see [Cloud Analytics](https://console.ng.bluemix.net/docs/services/IoT/cloud_analytics.html#rules).

### Schemas
{: #schema}

JSON schemas are used to define the structure of the payload of inbound events that are published to the {{site.data.keyword.iot_short_notm}} from devices, and the state of the device. For more information about JSON Schema, see [JSON Schema](http://json-schema.org/).

In Information Management, two JSON schemas are referenced - event schemas and application interface schemas.

Event schemas are used by the physical interface and define the structure of the events that are published to {{site.data.keyword.iot_short_notm}} by a device. Incoming events are validated against the event schema. For example, you might want to create an event schema that validates that the inbound event is in JSON format, contains two fields, and that the values that are associated with these fields are numeric.

Application interfaces are used to define the structure of the device state that is stored by {{site.data.keyword.iot_short_notm}}. The device state is validated against the application interface schema file.

### Event type
{: #event_type}

You must create an event type within your organization so that {{site.data.keyword.iot_short_notm}} can process the data that is contained within a specific event. All event types must reference an event schema. For the beta, all inbound events must be in JSON format if you want to use the Information Management feature.

### Device Type
{: #device_type}

Every device that is connected to the Watson IoT Platform is associated with a device type. Device types are groups of devices that share characteristics or behaviors. In Information Management, the device type is extended to reference the physical interface for the device and to reference the application interface that is used to retrieve the device state. A device type can expose multiple application interfaces.

For more information about device types, see the "Identifiers and device types" section in [Device Model](https://console.ng.bluemix.net/docs/services/IoT/reference/device_model.html#id_and_device_types).

### Mappings
{: #mappings}

The mappings resource defines how the properties from inbound events are mapped to the properties that are defined on the application interface that is associated with a device type. The mapping object must specify the identifier for the relevant application interface, and the event identifiers that are defined by the physical interface that is associated with the device type.

## High-level workflow
{: #workflow}

Ensure that you have registered for a {{site.data.keyword.Bluemix_notm}} account and created an instance of the {{site.data.keyword.iot_short_notm}} service in your {{site.data.keyword.Bluemix_notm}} organization. You will then receive a 6 character organization ID.  This ID is required in the hostname for any API calls. The base URL for your organization can be accessed at the following address: [https://**orgId**.internetofthings.ibmcloud.com/api/v0002/](https://**orgId**.internetofthings.ibmcloud.com/api/v0002/).

After you set up an instance of {{site.data.keyword.iot_short_notm}} in your Bluemix organization and register and connect your devices, you can start using the Information Management feature. For more information about getting started with {{site.data.keyword.iot_short_notm}}, see [Getting started with Watson IoT Platform](https://console.ng.bluemix.net/docs/services/IoT/index.html).

Use the following high-level steps to help you to start using the Information Management feature.
For details about the REST API, see the [{{site.data.keyword.iot_short_notm}} HTTP REST API](https://docs.internetofthings.ibmcloud.com/swagger/info-mgmt-beta.html) documentation.
For more detailed information about each of the following steps, see the [Example scenario](#scenario).

1. [Create an event schema file](#step1). An event schema file defines the structure and format of an inbound event. The event schema file must be in JSON format.

2. [Create an event schema resource](#step2) for your event type by using the REST API POST method with the following URI: ```https://**orgId**.internetofthings.ibmcloud.com/api/v0002/schemas```   

3. [Create an event type](#step3) and add the event type to your event schema by using the REST API POST method with the following URI: ```https://**orgId**.internetofthings.ibmcloud.com/api/v0002/event/types```

  **Note** Add the event type to the event schema by providing the schema identifier in the payload of the POST method. The schema identifier is returned in response that is received when the event schema resource is created.  

4. [Create an application interface schema file](#step4). An application interface schema file defines the message format this is used by applications. The application interface schema file must be in JSON format.

5. [Create an application interface schema resource](#step5) by using the REST API POST method with the following URI: ```https://**orgId**.internetofthings.ibmcloud.com/api/v0002/schemas```   

6. [Create an application interface](#step6) and add the application interface to your application interface schema by using the REST API POST method with the following URI: ```https://**orgId**.internetofthings.ibmcloud.com/api/v0002/applicationinterfaces```

7. [Create a physical interface](#step7) by using the REST API POST method with the following URI: ```https://**orgId**.internetofthings.ibmcloud.com/api/v0002/physicalinterfaces```

8. [Add the event type](#step8) to the physical interface by using the REST API POST method with the following URI: ```https://**orgId**.internetofthings.ibmcloud.com/api/v0002/physicalinterfaces/{physicalInterfaceId}/events```

9. [Create a device type](#step9) to connect your application interface and your physical interface by using the REST API POST method with the following URI: ```https://**orgId**.internetofthings.ibmcloud.com/api/v0002/device/types```

 **Tip:** If you are using an existing device type, use the REST API PUT method with the following URI: ```https://**orgId**.internetofthings.ibmcloud.com/api/v0002/device/types/type/{typeId}```

 **Note** If you update the device type by using the REST API PUT method, all existing devices that are associated with that device type are updated.

10. [Add your application interface](#step10) to a device type by using the REST API POST method with the following URI: ```https://**orgId**.internetofthings.ibmcloud.com/api/v0002/types/{typeId}/applicationinterfaces```

11. [Define mappings](#step11) to add your inbound event to a property or properties that are in your application interface by using the REST API POST method with the following URI:        ```https://**orgId**.internetofthings.ibmcloud.com/api/v0002/device/types/{typeId}/mappings/{applicationInterfaceId}```

12. [Publish an inbound device event](step12).

13. [Check](#step13) that the state of the device is changed by using the REST API GET method with the following URI: ```https://**orgId**.internetofthings.ibmcloud.com/api/v0002/device/types/{typeId}/devices/{deviceId}/state/{applicationInterfaceId}```


## Example scenario
{: #scenario}

Use the following information to create a scenario in which a temperature sensor and a humidity sensor publish events to {{site.data.keyword.iot_short_notm}}. The events are processed, which causes a change to the device state representation. This update causes a state change notification to be sent to the application.  

### Pre-requisites

You must have a {{site.data.keyword.iot_short_notm}} organization instance and an API key or token for that organization. For more information about API keys and tokens, see [Application, device, and gateway connections to Watson IoT Platform](https://console.ng.bluemix.net/docs/services/IoT/reference/security/connect_devices_apps_gw.html).

### About this task

In this scenario, a device type that is called **EnvironmentSensor** is configured. This device type publishes two events:
- A temperature event to the topic ```iot-2/evt/tevt/fmt/json``` that has the following example payload:
```
{
  “t” : 21.3
}
```
This payload represents a temperature in degrees Celsius.

- A humidity event to the topic ```iot-2/evt/hevt/fmt/json``` that has the following example payload:
```
{
  “h” : 48.4
}
```
This payload represents a humidity percentage.

An application interface is also configured. This application interface represents the state for devices of this type in the following structure:
```
{
  “temperature” : <current temperature value>,
  “humidity” : <current humidity value>
}
```

## Steps

Use the following information to configure the example scenario.

### Create an event schema file
{: #step1}

For this scenario, create two event schema files to define the structure of each of the inbound events.

The following example shows how to create a schema file that is called *tempEventSchema.json*. This file defines the structure of an inbound event from a temperature sensor:

```
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type" : "object",
  "title" : "EnvSensor TempEvent Schema",
  "description" : “defines the structure of a temperature event",
  "properties" : {
    "t" : {
      "description" : "temperature in degrees Celsius",
      "type" : "number",
      "minimum" : -273.15,
      "default" : 0.0
    }
  },
  "required" : ["t"],
  "optional" : []
}
  ```

The following example shows how to create a schema file that is called *humEventSchema.json*. This file defines the structure of an inbound event from a humidity sensor:

```
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type" : "object",
  "title" : "EnvSensor HumEvent Schema",
  "description" : “defines the structure of a humidity event",
  "properties" : {
    "h" : {
      "description" : "percentage humidity",
      "type" : "number",
      "minimum" : 0,
      "maximum" : 100,
      "default" : 0.0
    }
  },
  "required" : ["h"],
  "optional" : []
}
  ```

### Create an event schema resource for your event type
{: #step2}

To create an event schema resource, use the following API:

```
POST /schemas
```
For more details, see the [{{site.data.keyword.iot_short_notm}} HTTP REST API](https://docs.internetofthings.ibmcloud.com/swagger/info-mgmt-beta.html) documentation.

The following example shows how to use cURL to create the event schema resource *tempEventSchema.json*:

```
curl --request POST \
  --url https://yourOrgID.internetofthings.ibmcloud.com/api/v0002/schemas \
  --header 'authorization: Basic YS1seDlobW8tOWlxdDU2a2ZpbDp2eHZ4WUIxZWxYR2J0WlVmWkI=' \
  --header 'cache-control: no-cache' \
  --header 'content-type: multipart/form-data; boundary=---011000010111000001101001' \
  --form name=tempEventSchema \
  --form 'schemaFile=@"/Users/ANOther/Documents/IoT/DeviceState/deviceStateDemo/setup/schemas/tempEventSchema.json'
```

The following example shows a response to the POST method:

```
{
  "name" : “tempEventSchema",
  "createdBy" : "a-8x7nmj-9iqt56kfil",
  "contentType" : "application/octet-stream",
  "updated" : "2016-12-06T14:38:52Z",
  "schemaFileName" : "tempEvent.json",
  "created" : "2016-12-06T14:38:52Z",
  "id" : "5846cd7c6522050001db0e0d",
  "refs" : {
      "content" : "/schemas/5846cd7c6522050001db0e0d/content"
  },
  "schemaType" : "json-schema",
  "updatedBy" : "a-8x7nmj-9iqt56kfil"
}
```

The following example show how to use cURL to create the event schema resource *humEventSchema.json*:

```
curl --request POST \
  --url https://yourOrgID.internetofthings.ibmcloud.com/api/v0002/schemas \
  --header 'authorization: Basic YS1seDlobW8tOWlxdDU2a2ZpbDp2eHZ4WUIxZWxYR2J0WlVmWkI=‘ \
  --header 'cache-control: no-cache’ \
  --header 'content-type: multipart/form-data; boundary=---011000010111000001101001’ \
  --form name=humEventSchema \
  --form 'schemaFile=@"/Users/ANOther/Documents/IoT/DeviceState/deviceStateDemo/setup/schemas/humEventSchema.json"'
```

The following example shows a response to the POST method:

```
{
  "schemaType" : "json-schema",
  "schemaFileName" : "humEvent.json",
  "updated" : "2016-12-06T14:44:51Z",
  "name" : "humEventSchema",
  "updatedBy" : "a-8x7nmj-9iqt56kfil",
  "created" : "2016-12-06T14:44:51Z",
  "id" : "5846cee36522050001db0e0e",
  "refs" : {
      "content" : "/schemas/5846cee36522050001db0e0e/content"
  },
  "contentType" : "application/octet-stream",
  "createdBy" : "a-8x7nmj-9iqt56kfil"
}
```

### Create an event type and add the event type to your event schema
{: #step3}

Each event type references the relevant event schema that was created in the previous example by using the schema identifier. The schema identifier is returned in the example response.

To create an event type, use the following API:

```
POST /event/types
```

For more details, see the [{{site.data.keyword.iot_short_notm}} HTTP REST API](https://docs.internetofthings.ibmcloud.com/swagger/info-mgmt-beta.html) documentation.


The following example shows how to use cURL to create an event type for a temperature event:

```
curl --request POST \
  --url https://yourOrgID.internetofthings.ibmcloud.com/api/v0002/event/types \
  --header 'authorization: Basic YS1seDlobW8tOWlxdDU2a2ZpbDp2eHZ4WUIxZWxYR2J0WlVmWkI=' \
  --header 'cache-control: no-cache' \
  --header 'content-type: application/json' \
  --data '{"name" : "temperatureEvent", "schemaId" : "5846cd7c6522050001db0e0d”}'
```

The following example shows a response to the POST method:

```
{
  "updated" : "2016-12-06T14:53:49Z",
  "schemaId" : "5846cd7c6522050001db0e0d",
  "refs" : {
    "schema" : "/schemas/5846cd7c6522050001db0e0d"
  },
  "name" : "temperatureEvent",
  "created" : "2016-12-06T14:53:49Z",
  "updatedBy" : "a-8x7nmj-9iqt56kfil",
  "id" : "5846d0fd6522050001db0e0f",
  "createdBy" : "a-8x7nmj-9iqt56kfil"
}
```

The following example shows how to use cURL to create an event type for a humidity event:

```
curl --request POST \
  --url https://yourOrgID.internetofthings.ibmcloud.com/api/v0002/event/types \
  --header 'authorization: Basic YS1seDlobW8tOWlxdDU2a2ZpbDp2eHZ4WUIxZWxYR2J0WlVmWkI=' \
  --header 'cache-control: no-cache' \
  --header 'content-type: application/json' \
  --data '{"name" : "humidityEvent", "schemaId" : "5846cee36522050001db0e0e"}'
```

The following example shows a response to the POST method:

```
{
  "createdBy" : "a-8x7nmj-9iqt56kfil",
  "schemaId" : "5846cee36522050001db0e0e",
  "created" : "2016-12-06T15:00:20Z",
  "id" : "5846d2846522050001db0e10",
  "updated" : "2016-12-06T15:00:20Z",
  "name" : “humidityEvent”,
  "refs" : {
    "schema" : "/schemas/5846cee36522050001db0e0e"
  },
  "updatedBy" : "a-8x7nmj-9iqt56kfil"
}
```

### Create an application interface schema file
{: #step4}

The following example shows how to create an application interface schema file that is called *envSensor.json*.

```
{
"$schema": "http://json-schema.org/draft-04/schema#",
  "type" : "object",
  "title" : "Environment Sensor Schema",
  "description" : "Schema to represent a canonical environment sensor device",
  "properties" : {
      "temperature" : {
          "description" : "temperature in degrees Celsius",
          "type" : "number",
          "minimum" : -273.15,
          "default" : 0.0
      },
      "humidity" : {
          "description" : "percentage humidity",
          "type" : "number",
          "minimum" : 0,
          "maximum" : 100,
          "default" : 0.0
     }
  },
  "required" : ["temperature", "humidity"],
  "optional" : []
}
```

### Create an application interface schema resource
{: #step5}

To create an application interface schema resource, use the following API:

```
POST /schemas
```
For more details, see the [{{site.data.keyword.iot_short_notm}} HTTP REST API](https://docs.internetofthings.ibmcloud.com/swagger/info-mgmt-beta.html) documentation.

The following example shows how to use cURL to create the application interface schema:

```
curl --request POST \
  --url https://yourOrgID.internetofthings.ibmcloud.com/api/v0002/schemas \
  --header 'authorization: Basic YS1seDlobW8tOWlxdDU2a2ZpbDp2eHZ4WUIxZWxYR2J0WlVmWkI=' \
  --header 'cache-control: no-cache' \
  --header 'content-type: multipart/form-data; boundary=—011000010111000001101001' \
  --form name=humEventSchema \
  --form 'schemaFile=@"/Users/ANOther/Documents/IoT/DeviceState/deviceStateDemo/setup/schemas/envSensor.json"'
```

The following example shows a response to the POST method:

```
{
  "created" : "2016-12-06T16:51:14Z",
  "name" : "humEventSchema",
  "createdBy" : "a-8x7nmj-9iqt56kfil",
  "updated" : "2016-12-06T16:51:14Z",
  "updatedBy" : "a-8x7nmj-9iqt56kfil",
  "schemaType" : "json-schema",
  "contentType" : "application/octet-stream",
  "schemaFileName" : "envSensor.json",
  "refs" : {
    "content" : "/schemas/5846ec826522050001db0e11/content"
  },
  "id" : "5846ec826522050001db0e11"
}
```


### Create an application interface and add the application interface to your application interface schema
{: #step6}

To create an application interface, use the following API:

```
POST /applicationinterfaces
```
For more details, see the [{{site.data.keyword.iot_short_notm}} HTTP REST API](https://docs.internetofthings.ibmcloud.com/swagger/info-mgmt-beta.html) documentation.

Use the schema identifier that was returned in the previous response to add the application interface to the application interface schema.

The following example shows how to use cURL to create an application interface:

```
curl --request POST \
  --url https://yourOrgID.internetofthings.ibmcloud.com/api/v0002/applicationinterfaces \
  --header 'authorization: Basic YS1seDlobW8tOWlxdDU2a2ZpbDp2eHZ4WUIxZWxYR2J0WlVmWkI=' \
  --header 'cache-control: no-cache' \
  --header 'content-type: application/json' \
  --data '{"name" : "env sensor interface", "schemaId" : "5846ec826522050001db0e11"}'
```

The following example shows a response to the POST method:

```
{
  "createdBy" : "a-8x7nmj-9iqt56kfil",
  "refs" : {
      "schema" : "/schemas/5846ec826522050001db0e11"
  },
  "schemaId" : "5846ec826522050001db0e11",
  "created" : "2016-12-06T16:53:27Z",
  "updatedBy" : "a-8x7nmj-9iqt56kfil",
  "id" : "5846ed076522050001db0e12",
  "updated" : "2016-12-06T16:53:27Z",
  "name" : "env sensor interface"
}
```

### Create a physical interface
{: #step7}

To create a physical interface, use the following API:

```
POST /physicalinterfaces
```
For more details, see the [{{site.data.keyword.iot_short_notm}} HTTP REST API](https://docs.internetofthings.ibmcloud.com/swagger/info-mgmt-beta.html) documentation.

The following example shows how to use cURL to create a physical interface:

```
curl --request POST \
  --url https://yourOrgID.internetofthings.ibmcloud.com/api/v0002/physicalinterfaces \
  --header 'authorization: Basic YS1seDlobW8tOWlxdDU2a2ZpbDp2eHZ4WUIxZWxYR2J0WlVmWkI=‘ \
  --header 'cache-control: no-cache’ \
  --header 'content-type: application/json’ \
  --data '{"name" : "Env sensor physical interface"}'
```

The following example shows a response to the POST method:

```
{
  "updatedBy" : "a-8x7nmj-9iqt56kfil",
  "refs" : {
    "events" : "/physicalinterfaces/5847d1df6522050001db0e1a/events"
  },
  "id" : "5847d1df6522050001db0e1a",
  "name" : "Env sensor physical interface",
  "created" : "2016-12-07T09:09:51Z",
  "updated" : "2016-12-07T09:09:51Z",
  "createdBy" : "a-8x7nmj-9iqt56kfil"
}
```

### Add a temperature event type to the physical interface
{: #step8}

Add the temperature event to the physical interface by using the *eventId* from the topic and the *eventTypeId* from the creation of the resource.

To add an event to your physical interface, use the following API:

```
POST /physicalinterfaces/{physicalInterfaceId}/events
```
For more details, see the [{{site.data.keyword.iot_short_notm}} HTTP REST API](https://docs.internetofthings.ibmcloud.com/swagger/info-mgmt-beta.html) documentation.

The following example shows how to use cURL to add a temperature event to the physical interface:

```
curl --request POST \
  --url https://yourOrgID.internetofthings.ibmcloud.com/api/v0002/physicalinterfaces/5847d1df6522050001db0e1a/events \
  --header 'authorization: Basic YS1seDlobW8tOWlxdDU2a2ZpbDp2eHZ4WUIxZWxYR2J0WlVmWkI=' \
  --header 'cache-control: no-cache' \
  --header 'content-type: application/json' \
  --data '{"eventId" : “tevt", "eventTypeId" : "5846d0fd6522050001db0e0f"}'
```

The following example shows a response to the POST method:

```
{
  "eventTypeId" : "5846d0fd6522050001db0e0f",
    "eventId" : "tevt"
}
```

### Add a humidity event type to the physical interface

Add the humidity event to the physical interface by using the *eventId* from the topic and the *eventTypeId* from the creation of the resource.

The following example shows how to use cURL to add a temperature event to the physical interface:

```
curl --request POST \
  --url https://yourOrgID.internetofthings.ibmcloud.com/api/v0002/physicalinterfaces/5847d1df6522050001db0e1a/events \
  --header 'authorization: Basic YS1seDlobW8tOWlxdDU2a2ZpbDp2eHZ4WUIxZWxYR2J0WlVmWkI=' \
  --header 'cache-control: no-cache' \
  --header 'content-type: application/json' \
  --data '{"eventId" : "hevt", "eventTypeId" : "5846d2846522050001db0e10"}'
```

The following example shows a response to the POST method:

```
{
  "eventTypeId" : "5846d2846522050001db0e10",
    "eventId" : “hevt"
}
```

### Create a device type to connect your application interface and your physical interface
{: #step9}

To add a device type, use the following API:

```
POST /device/types
```

If you are using an existing device type, use the following API to update that device type to connect your application interface and your physical interface:
```
PUT /device/types/{typeId}  
```
**Note** If you update the device type in this way, all existing devices that are associated with that device type are updated.

For more details, see the [{{site.data.keyword.iot_short_notm}} HTTP REST API](https://docs.internetofthings.ibmcloud.com/swagger/info-mgmt-beta.html) documentation.

The following example shows how to use cURL to create a device type:

```
curl --request POST \
--url https://yourOrgID.internetofthings.ibmcloud.com/api/v0002/device/types \
  --header 'authorization: Basic YS1seDlobW8tOWlxdDU2a2ZpbDp2eHZ4WUIxZWxYR2J0WlVmWkI=' \
  --header 'cache-control: no-cache' \
  --header 'content-type: application/json' \
  --data '{"id" : "EnvSensor","description" : "an env sensor","classId" : "Device","deviceInfo" : {},"metadata" : {}, "physicalInterfaceId" : "5847d1df6522050001db0e1a”}’
```

The following example shows a response to the POST method:

```
{
  "deviceInfo" : {},
  "physicalInterfaceId" : "5847d1df6522050001db0e1a",
  "updatedDateTime" : "2016-12-07T09:49:52+00:00",
  "refs" : {
    "mappings" : "/device/types/EnvSensor/mappings",
    "applicationInterfaces" : "/device/types/EnvSensor/applicationinterfaces",
    "physicalInterface" : "/physicalinterfaces/5847d1df6522050001db0e1a"
   },
  "id" : "EnvSensor",
  "description" : "an env sensor",
  "metadata" : {},
  "classId" : "Device",
  "createdDateTime" : "2016-12-07T09:49:52+00:00"
}
```

### Add the application interface to the device type
{: #step10}

Add the application interface to the device type by using the output of the POST method that was used to create the application interface.

To add an application interface to a device type, use the following API:

```
POST /device/types/{typeId}/applicationinterfaces
```
For more details, see the [{{site.data.keyword.iot_short_notm}} HTTP REST API](https://docs.internetofthings.ibmcloud.com/swagger/info-mgmt-beta.html) documentation.

The following example shows how to use cURL to add the application interface to the device:

```
curl --request POST \
--url https://yourOrgID.internetofthings.ibmcloud.com/api/v0002/device/types/EnvSensor/applicationinterfaces \
--header 'authorization: Basic YS1seDlobW8tOWlxdDU2a2ZpbDp2eHZ4WUIxZWxYR2J0WlVmWkI=' \
--header 'cache-control: no-cache' \
--header 'content-type: application/json' \
--data '{"createdBy" : "a-8x7nmj-9iqt56kfil", \
          "refs" : {
              "schema" : "/schemas/5846ec826522050001db0e11"
          },
          "schemaId" : "5846ec826522050001db0e11", "created" : "2016-12-06T16:53:27Z", \
          "updatedBy" : "a-8x7nmj-9iqt56kfil","id" : "5846ed076522050001db0e12","updated" : "2016-12-06T16:53:27Z","name" : "env sensor interface”
        }'
```

The following example shows a response to the POST method:

```
{
  "refs" : {
      "schema" : "/schemas/5846ec826522050001db0e11"
  },
  "updated" : "2016-12-06T16:53:27Z",
  "updatedBy" : "a-8x7nmj-9iqt56kfil",
  "createdBy" : "a-8x7nmj-9iqt56kfil",
  "name" : "env sensor interface",
  "created" : "2016-12-06T16:53:27Z",
  "id" : "5846ed076522050001db0e12",
  "schemaId" : "5846ec826522050001db0e11"
}
```

### Define mappings to add your inbound event to a property in your application interface.
{: #step11}

Add a mapping to the device from the two events to the application interface.

To create mappings, use the following API:

```
POST /device/types/{typeId}/mappings
```
For more details, see the [{{site.data.keyword.iot_short_notm}} HTTP REST API](https://docs.internetofthings.ibmcloud.com/swagger/info-mgmt-beta.html) documentation.


The following example shows how to use cURL to add a mapping to the device from the two events to the application interface:

```
curl --request POST \
  --url https://yourOrgID.internetofthings.ibmcloud.com/api/v0002/device/types/EnvSensor/mappings \
  --header 'authorization: Basic YS1seDlobW8tOWlxdDU2a2ZpbDp2eHZ4WUIxZWxYR2J0WlVmWkI=' \
  --header 'cache-control: no-cache' \
  --header 'content-type: application/json' \
  --data '{"applicationInterfaceId" : "5846ed076522050001db0e12","propertyMappings" : {
              "tevt" : {
                  "temperature" : "$event.t"
              },
              "hevt" : {
                    "humidity" : "$event.h"
              }
            }
          }'
```

The following example shows a response to the POST method:

```
{
  "propertyMappings" : {
    "hevt" : {
      "humidity" : "$event.h"
    },
    "tevt" : {
      "temperature" : "$event.t"
    }
  },
  "applicationInterfaceId" : "5846ed076522050001db0e12"
}
```

### Publish an inbound device event
{: #step12}

For information about publishing an inbound event from a device, see [MQTT connectivity for applications](https://console.ng.bluemix.net/docs/services/IoT/applications/mqtt.html#publishing_device_events).


### Check that the state of the device is changed
{: #step13}

For the beta release, applications can retrieve the device state by using an HTTP API that is authorized by using API keys, in the same way that other APIs are currently authorized in {{site.data.keyword.iot_short_notm}}. For more information about API keys and tokens, see [Application, device, and gateway connections to Watson IoT Platform](https://console.ng.bluemix.net/docs/services/IoT/reference/security/connect_devices_apps_gw.html).


To check the state of the device, use the following API:

```
GET /device/types/{typeId}/devices/{deviceId}/state/{applicationInterfaceId}
```
For more details, see the [{{site.data.keyword.iot_short_notm}} HTTP REST API](https://docs.internetofthings.ibmcloud.com/swagger/info-mgmt-beta.html) documentation.

In this scenario, a device instance that has an identifier of **myDevice1** is assumed.

The following example shows how to use cURL to retrieve the current state of the device by referencing the identifier of the application interface that was created:

```
curl --request GET \
  --url https://yourOrgID.internetofthings.ibmcloud.com/api/v0002/device/types/EnvSensor/devices/myDevice1/state/5846ed076522050001db0e12 \
  --header 'authorization: Basic TGS04NXg5dHotKNBzbGZ5eWdiaToxX543S0lKOmE3Tk5Mc0xMu6n=' \
  --header 'cache-control: no-cache'
```
The following example shows a response to the GET method:

```
{
  "temperature":21.3,"humidity":48.4
}
```