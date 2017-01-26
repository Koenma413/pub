#IP for Smart Objects

##Validation

We currently have continuous validation of the XML through Travis, in the future other automated test could be added.
[![Build Status](https://travis-ci.org/IPSO-Alliance/pub.svg?branch=master)](https://travis-ci.org/IPSO-Alliance/pub)

###If you are familiar with IPSO Objects you can fetch them from the [IPSO Registry](https://github.com/IPSO-Alliance/pub/tree/master/reg/xml) or directly check some of the [sample implementations](#implementations)

##Table of Contents
1. [Introduction](#intro)
2. [Data Model Components](#components)
3. [Composite Objects](#composite)
4. [Sample Object Definition](#definition)
5. [Object and Resource ID registry](#registry)
6. [Sample Implementations](#implementations)
7. [Creating new Objects](#newobjects)

<a name="introduction"></a>
## Introduction
Standards for constrained devices are rapidly consolidating and the availability of IP on constrained devices enabled these devices to easily connect to the Internet. The IETF has also created a set of specifications for such IP-enabled devices to work in a Web-like fashion. One such protocol is the [Constrained Application Protocol (CoAP)](https://tools.ietf.org/html/rfc7252) that provides request/response methods, ways to identify resources, discovery mechanisms, etc. similar to the [Hypertext Transfer Protocol](https://tools.ietf.org/html/rfc2616) but for use in constrained environments.

However, the use of standardized protocols does not ensure interoperability on the application layer. Therefore, there is a clear need for being able to communicate using structured data models on top of protocols like CoAP and HTTP.

IPSO Smart Objects provide a common design pattern, an object model, to provide high level interoperability between Smart Object devices and connected software applications on other devices and services. IPSO Objects are defined in such a way that they do not depend on the use of CoAP, any RESTful protocol is sufficient. Nevertheless, to develop a complete and interoperable solution the Object model is based on the [Open Mobile Alliance Lightweight Specification (OMA LWM2M)](http://www.openmobilealliance.org/release/LightweightM2M/V1_0-20160407-C/OMA-TS-LightweightM2M-V1_0-20160407-C.pdf) and the [OMNA Registry](http://www.openmobilealliance.org/wp/OMNA/LwM2M/LwM2MRegistry.html), which is a set of management interfaces built on top of CoAP in order to enable device management operations (bootstrapping, firmware updates, error reporting, etc.). While LWM2M uses objects with fixed mandatory resources, IPSO Smart Objects use a more reusable design.


<a name="components"></a>
##2. Data Model Components

The data model for IPSO Smart Objects consists of 5 parts:

1. Object Representation
3. Data types
4. Operations
5. Content formats

### 2.1. Object Representation

Objects and resources are implicitly mapped into the URI path hierarchy by following the OMA LWM2M object model, in which each URI path component sequentially represents the Object Type ID, the Object Instance ID and the Resource Type ID. More precisely the structure consists of three unsigned 16-bit integers separated by the character '/':

```
<object ID>/<object instance ID>/<resource ID>
```

This URI template approach follows the [Web Linking](https://tools.ietf.org/html/rfc5988) and the [CoRE Link Format](https://tools.ietf.org/html/rfc5988) . Objects are typed containers, which define the semantic type of instances. Instances represent specific object types at runtime, and allow Smart Object endpoints to expose multiple sensors and actuators of a particular type. Object instances are themselves containers for resources, which are the observable properties of an object.

For example, a temperature sensor example URI would be `3303/0/5700`:

```
3300  -> Temperature Sensor
0     -> instance 0 of a Temperature Sensor
5700  -> resource having the current value or a most recent reading
```

Semantically, the object type represents a single measurement, actuation, or control point for example a temperature sensor, a light (actuator), or an on-off switch (control point).
A resource specifies a particular view or active property of an object. For example, a temperature sensor object might expose the current value (most recent reading), also the minimum and maximum possible reading, the minimum and maximum reading in an interval, and attributes like engineering units and application type.

Attributes describe the metadata configuration, settings, and state of an object or resource, and are discoverable by reading the link-format data of an object or resource. Multiple attributes may be serialized in the link-format descriptors that an object exposes. Some attributes are immutable for a given object or resource type. For example, the static read, write, and execute capability attributes are derived from a Smart Object’s definition file, while other attributes, like the LWM2M Notification Attributes, are used to dynamically configure a particular object instance or resource. Attributes are represented using [CoRE Link Format](https://tools.ietf.org/html/rfc5988) or an equivalent mapping to other content formats, for example, application/json+ld.

This abstraction allows application software to use simple APIs. For complex objects, linking of an object to another object through an object link resource is allowed. This enables the recursion to be handled at the object level, using design patterns similar to web linking. An application client can consume a devices API without knowing its structure and attributes a priori.


### 2.2. Data types

There are 7 data types, same as the ones defined in OMA LWM2M [3].

1. String: A UTF-8 string, the minimum and/or maximum length of the String MAY be defined.
2. Integer: An 8, 16, 32 or 64-bit signed integer. The valid range of the value for a Resource SHOULD be defined. This data type is also used for the purpose of enumeration.
3. Float: A 32 or 64-bit floating point value. The valid range of the value for a Resource SHOULD be defined.
4. Boolean: An integer with the value 0 for False and the value 1 for True.
5. Opaque: A sequence of binary octets, the minimum and/or maximum length of the String MAY be defined.
6. Time: Unix Time. A signed integer representing the number of seconds since Jan 1st, 1970 in the UTC time zone.
7. Object Link: The object link is used to refer an Instance of a given Object. An Object link value is composed of two concatenated 16-bits unsigned integers following the Network Byte Order convention.


### 2.3. Operations
IPSO Objects and their resources have the same operations as their counterparts in the OMA LWM2M specification [3] with the same semantics.

1. Resource values: Read, Write, Execute (restricted by the Access Type field)
2. Object Instances: Create, Delete (restricted by the Multiple Instances field)
3. Objects and their instances: Read, Write
4. Attributes: Set, Discover

### 2.4. Content formats

Content formats are those specified by the OMA LWM2M specification:

1. Resource values: text/plain, tlv
2. Objects: text/senml+json, application/cbor, binary/tlv
3. Attributes: link-format, link-format+json


<a name="composite"></a>
##3. Composite Objects

As devices increase in complexity (e.g., from a sensor to an appliance, from a switch to a complex fuel control actuator) the need to link resources to create more complex objects or ”Composite Objects” arises. Such a composite object can, for example, be constructed with a single reusable type ”generic composite object” with one ID. The resources may be of a generic reusable link type, also using a single ID, with multiple instances allowed.

For example, ’4000/0/6700/0’ where 4000 is a ”composite object” and 6700 is ”generic object link”. Composite objects offer higher granularity than one large nested object would. An observer of a device represented as a composite object could reduce bandwidth utilization by observing only the linked object instances instead of the full object. Figure 3 shows an example, performing a GET operation to the IPSO thermostat composite object ”/8300/0/7100” would retrieve an object link to ”/3300/0”.

# ![IPSO Object](https://github.com/IPSO-Alliance/pub/blob/master/linking.png)

<a name="definition"></a>
##4. Sample Object Definition

### 4.1 Definition documents

For practical purposes we require a format for representing the objects in an unambiguous fashion. IPSO has used XML to the XML Schema in Appendix A, either hand-written or automatically generated from other sources. In the XML version, Integers, Floats and Time values must be represented as in the above subsection. Binary (Opaque) values must be represented in Base64-encoded form. Booleans must be represented using the lower-case strings true and false.

You can see the existing IPSO Objects in XML in the [IPSO Registry](https://github.com/IPSO-Alliance/pub/tree/master/reg/xml)

### 4.2 Humidity Sensor

The following is a example of a humidity sensor that contains the sensor value, units, min and max measured values, min and max range values and a rest of those.
# ![IPSO Object](https://github.com/IPSO-Alliance/pub/blob/master/humidity_object.png)


The following is the definition document for the Humidity Object in XML.

```
<?xml version="1.0" encoding="utf-8"?>
<LWM2M>
	<Object ObjectType="MODefinition">
		<Name>Humidity</Name>
		<Description1>Description: This IPSO object should be used with a humidity sensor to report a humidity measurement.  It also provides resources for minimum/maximum measured values and the minimum/maximum range that can be measured by the humidity sensor. An example measurement unit is relative humidity as a percentage (ucum:%).</Description1>
		<ObjectID>3304</ObjectID>
		<ObjectURN>urn:oma:lwm2m:ext:3304</ObjectURN>
		<MultipleInstances>Multiple</MultipleInstances>
		<Mandatory>Optional</Mandatory>
		<Resources>
			<Item ID="5700">
				<Name>Sensor Value</Name>
				<Operations>R</Operations>
				<MultipleInstances>Single</MultipleInstances>
				<Mandatory>Mandatory</Mandatory>
				<Type>Float</Type>
				<RangeEnumeration></RangeEnumeration>
				<Units>Defined by “Units” resource.</Units>
				<Description>Last or Current Measured Value from the Sensor</Description>
			</Item>
			<Item ID="5601">
				<Name>Min Measured Value</Name>
				<Operations>R</Operations>
				<MultipleInstances>Single</MultipleInstances>
				<Mandatory>Optional</Mandatory>
				<Type>Float</Type>
				<RangeEnumeration></RangeEnumeration>
				<Units>Defined by “Units” resource.</Units>
				<Description>The minimum value measured by the sensor since power ON or reset</Description>
			</Item>
			<Item ID="5602">
				<Name>Max Measured Value</Name>
				<Operations>R</Operations>
				<MultipleInstances>Single</MultipleInstances>
				<Mandatory>Optional</Mandatory>
				<Type>Float</Type>
				<RangeEnumeration></RangeEnumeration>
				<Units>Defined by “Units” resource.</Units>
				<Description>The maximum value measured by the sensor since power ON or reset</Description>
			</Item>
			<Item ID="5603">
				<Name>Min Range Value</Name>
				<Operations>R</Operations>
				<MultipleInstances>Single</MultipleInstances>
				<Mandatory>Optional</Mandatory>
				<Type>String</Type>
				<RangeEnumeration></RangeEnumeration>
				<Units>Defined by “Units” resource.</Units>
				<Description>The minimum value that can be measured by the sensor</Description>
			</Item>
			<Item ID="5604">
				<Name>Max Range Value</Name>
				<Operations>R</Operations>
				<MultipleInstances>Single</MultipleInstances>
				<Mandatory>Optional</Mandatory>
				<Type>Float</Type>
				<RangeEnumeration></RangeEnumeration>
				<Units>Defined by “Units” resource.</Units>
				<Description>The maximum value that can be measured by the sensor</Description>
			</Item>
			<Item ID="5701">
				<Name>Sensor Units</Name>
				<Operations>R</Operations>
				<MultipleInstances>Single</MultipleInstances>
				<Mandatory>Optional</Mandatory>
				<Type>String</Type>
				<RangeEnumeration></RangeEnumeration>
				<Units></Units>
				<Description>Measurement Units Definition e.g. “Cel” for Temperature in Celsius.</Description>
			</Item>
			<Item ID="5605">
				<Name>Reset Min and Max Measured Values</Name>
				<Operations>E</Operations>
				<MultipleInstances>Single</MultipleInstances>
				<Mandatory>Optional</Mandatory>
				<Type>Opaque</Type>
				<RangeEnumeration></RangeEnumeration>
				<Units></Units>
				<Description>Reset the Min and Max Measured Values to Current Value</Description>
			</Item>
		</Resources>
		<Description2></Description2>
	</Object>
</LWM2M>
```


<a name="registry"></a>
##5. List of registered Object IDs and Resource IDs.

Below is the set of registered Objects and their corresponding Object IDs.

| Object 				| Object ID   |
|:----------------------|:-----------:|
|    Digital 					| 3200|
|    Digital Output  			| 3201|
|    Analogue Input  			| 3202|
|    Analogue Output 			| 3203|
|    Generic Sensor  			| 3300|
|    Illuminance Sensor 		| 3301|
|    Presence sensor 			| 3302|
|    Temperature Sensor 		| 3303|
|    Humidity Sensor 			| 3304|
|    Power Measurement 			| 3305|
|    Actuation 					| 3306|
|    Set Point 					| 3308|
|    Load Control 				| 3310|
|    Light Control 				| 3311|
|    Power Control 				| 3312|
|    Accelerometer 				| 3313|
|    Magnetometer 				| 3314|
|    Barometer  	   		 	| 3315|
|    Voltage					| 3316|
|    Current					| 3317|
|    Frequency					| 3318|
|    Depth						| 3319|
|    Percentage					| 3320|
|    Altitude					| 3321|
|    Load						| 3322|
|    Pressure					| 3323|
|    Loudness					| 3324|
|    Concentration 				| 3325|
|    Acidity					| 3326|
|    Conductivity				| 3327|
|    Power						| 3328|
|    Power Factor				| 3329|
|    Rate						| 3346|
|    Distance					| 3330|
|    Energy						| 3331|
|    Direction					| 3332|
|    Time						| 3333|
|    Gyrometer					| 3334|
|    Color 						| 3335|
|    GPS Location 				| 3336|
|    Positioner 				| 3337|
|    Buzzer						| 3338|
|    Audio Clip					| 3339|
|    Timer						| 3340|
|    Addressable Text Display	| 3341|
|    On/Off Switch				| 3342|
|    Push Button				| 3347|
|    Level Controllers			| 3343|
|    Up/Down Control			| 3344|
|    Multi-state Selector		| 3348|
|    Multiple Axis Joystick		| 3345|


Below there is the set of Resources that can be used as building blocks for your Objects.

| Resource              | Resource ID   |
|:--------------------------------|:---:|
|    Digital Input State					| 5500|
|    Digital Input Counter				| 5501|
|    Digital Input Polarity				| 5502|
|    Digital Input Debounce				| 5503|
|    Digital Input Edge Selection	| 5504|
|    Digital Input Counter Reset	| 5505|
|    Current Time 	              | 5506|
|    Fractional Time  	          | 5507|
|    Min X Value	                | 5508|
|    Max X Value	                | 5509|
|    Min Y Value 	                | 5510|
|    Max Y Value	                | 5511|
|    Min Z Value 	                | 5512|
|    Max Z Value                	| 5513|
|    Latitude	                    | 5514|
|    Longitude 	                  | 5515|
|    Uncertainty 	                | 5516|
|    Velocity 	                  | 5517|
|    Timestamp                    | 5518|
|    Min Limit                  	| 5519|
|    Max Limit 	                  | 5520|
|    Delay Duration             	| 5521|
|    Clip 	                      | 5522|
|    Trigger 	                    | 5523|
|    Duration 	                  | 5524|
|    Minimum Off-time             | 5525|
|    Mode                         | 5526|
|    Text                   	    | 5527|
|    X Coordinate 	              | 5528|
|    Y Coordinate 	              | 5529|
|    Clear Display                | 5530|
|    Contrast                     | 5531|
|    Counter 	                    | 5534|
|    Current Position             | 5536|
|    Transition Time              | 5537|
|    Remaining Time               | 5538|
|    Up Counter                   | 5541|
|    Down Counter 	              | 5542|
|    Digital State 	              | 5543|
|    Cumulative Time 	            | 5544|
|    Max X Coordinate             | 5545|
|    Max Y Coordinate 	          | 5546|
|    Multi-state Input 	          | 5547|
|    Digital Output State					| 5550|
|    Digital Output Polarity			| 5551|
|    Analog Input State					  | 5600|
|    Min Measured Value           | 5601|
|    Max Measured Value           | 5602|
|    Min Range Value              | 5603|
|    Max Range Value              | 5604|
|Reset Min and Max Measured Values| 5605|
|    Sensor Value	                | 5700|
|    Sensor Units                 | 5701|
|    X Value                      | 5702|
|    Y Value                      | 5703|
|    Z Value                      | 5704|
|    Compass Direction            | 5705|
|    Colour                       | 5706|
|    Application Type	            | 5750|
|    Sensor Type	                | 5751|
|    Busy to Clear delay          | 5903|
|    Clear to Busy delay          | 5904|
|    Instantaneous active power   | 5800|
|    Min Measured active power    | 5801|
|    Max Measured active power    | 5802|
|    Min Range active power       | 5803|
|    Max Range active power       | 5804|
|    Cumulative active power      | 5805|
|    Active Power Calibration     | 5806|
|    Instantaneous reactive power | 5810|
|    Min Measured reactive power  | 5811|
|    Max Measured reactive power  | 5812|
|    Min Range reactive power     | 5813|
|    Max Range reactive power     | 5814|
|    Cumulative reactive power    | 5815|
|    Reactive Power Calibration   | 5816|
|    Power Factor                 | 5820|
|    Current Calibration          | 5821|
|    Reset Cumulative energy      | 5822|
|    Event Identifier             | 5823|
|    Start Time                   | 5824|
|    Duration In Min              | 5825|
|    Criticality Level            | 5826|
|    Avg Load Adj Pct             | 5827|
|    Duty Cycle                   | 5828|
|    Increase Input State         | 5532|
|    Decrease Input State         | 5533|
|    On/Off                       | 5850|
|    Dimmer                       | 5851|
|    On Time                      | 5852|
|    Muti-state Output            | 5853|
|    Off Time                     | 5854|
|    Set Point Value              | 5900|


<a name="implementations"></a>
## 6. Sample Implementations

* [JSON file]( https://github.com/eclipse/leshan/blob/master/leshan-core/src/main/resources/oma-objects-spec.json) of the supported LWM2M and IPSO Objects in [Leshan](http://www.eclipse.org/leshan/).

* Sample [C package](https://github.com/contiki-os/contiki/tree/master/apps/ipso-objects) for use of IPSO Objects in [Contiki](http://www.contiki-os.org).

* JS code templates of IPSO-defined devices [code templates](https://github.com/PeterEB/smartobject/blob/master/docs/templates.md). Each template gives a code snippet of how to initialize an _Object Instance_ with its `oid` and `iid`, and lists every _Resource_ the _Object Instance_ **may** have.  

* Sample [Smart Objects](https://npm.taobao.org/package/smartobject) Class can be used to create IPSO Smart Objects in your JavaScript applications. If you like to use the IPSO data model in your projects or products, you can use smartobject as the base class to abstract your hardware, sensor modules, or gadgets into plugins (node.js packages) for users convenience.

<a name="newobjects"></a>
## 7. Creating new Objects

The tools and links above provide sufficient information to get started with IPSO Objects. As the amount of devices and applications grows, you will have to define new Objects that define functionality missing on the [IPSO Registry](https://github.com/IPSO-Alliance/pub/tree/master/reg/xml). New Objects can be then created following the same design patterns and schema.

If you happen to find some basic Object that you feel should be registered, you can do a pull request on the master branch and commit the new Object to the [IPSO Registry](https://github.com/IPSO-Alliance/pub/tree/master/reg/xml) or [OMNA Registry](http://www.openmobilealliance.org/wp/OMNA/LwM2M/LwM2MRegistry.html). We will then add it there if it is needed.

If you have specific apps or gadgets that need to be covered by IPSO Objects you can create your own Application-Specific Objects.

---

Authored by [@jaimejim](https://github.com/jaimejim) - 2017 IPSO Alliance

NB: Please use the [issue tracker](https://github.com/IPSO-Alliance/pub/issues) if you find any mistakes on the content.
* Join the room here: [![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/IPSO-Alliance)
