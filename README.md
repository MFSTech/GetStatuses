Get Statuses
============

EDI (API) Documentation for the Standard, XML-Based format for BOL Statuses.

Overview
--------

A status is an event that occurs in the life of a shipment. Examples include physical events such as the original pickup and final delivery, along with with administrative event such as a scheduling call or invoicing. Exceptions such as weather delays and damage are also represented.

Our standard format uses XML. Rather than limit our format to a specific, triggering event, each status transmission contains all of the available events which have occurred relevant to the shipment. So the XML is a comprehensive collection of all events and BOL information.

We support both "Push" and "Pull" triggering mechanisms. That is, you can retrieve the status transmission from us on demand, without any prior configuration on our part ("Pull"), or we can configure our system to send you the status transmission based on various events ("Push").

We have a "Full" and "Slim" version of the status format. The "Slim" version of the status format contains only that information which is available on our public status page-- basically, the normal physical statuses and a scheduling call history. The "Full" version contains everything the "Slim" format does, and much more, including address information, estimated charges, notes to the customer, and various administrative events such as invoicing.

"Push" statuses-- where our system is sending the data using an established link to your server-- always use the "Full" version of the status format. Also, because every transmission contains all of the shipment's events, you never have to decide which events you want us to tell you about. You just have to decide which events should trigger transmissions.

When using the "Pull" method, you will be accessing a particular web endpoint. By default, you will get the the "Slim" version. However, if you specify a Security Code, you will be able to receive the "Full" version of the status transmission.

Transmission
------------

### Pull

"Pull" statuses can be retrieved via HTTP GET from or POST to these URLs:

* Production: [http://EDI.MFSClarity.com/EDI/Cust/Status.aspx](http://EDI.MFSClarity.com/EDI/Cust/Status.aspx).
* Test: [http://Demo.MFSClarity.com/EDI/Cust/Status.aspx](http://Demo.MFSClarity.com/EDI/Cust/Status.aspx)

It must be fed either (1) the BOL number (aka tracking number) in the `BOL` parameter or (2) our internal transaction number (aka Trans ID) in the `Tran` parameter. It can be supplied in either the query string or in the HTTP body (when POST is used) in "application/x-www-form-urlencoded". (This is the standard encoding used by browsers when submitting an HTML form.)

The HTTP response will have a content type of "text/xml". It will contain the XML status format directly.

Sending only the BOL number or Trans ID will result in the "Slim" version of the transmission. In order to receive the "Full" version, the Security Code must be specified in the `SecCode` parameter (again, either via query string or form content).

The Security Code is a random, typically 4-digit numeric code automatically by our system for the shipment. It is a rudimentary security mechanism to prevent casual snooping for status transmissions for shipments which are not relevant to you. The security code is returned in the [our standard XML acknowledgment](https://github.com/MFSTech/SendUsShipments/blob/master/ShipAck-Commented.xml "Shipment Acknowledgment XML Example") to [your XML shipment transmission](https://github.com/MFSTech/SendUsShipments/blob/master/Shipment-Commented.xml "Shipment XML Example"). See our [SendUsShipments repository](https://github.com/MFSTech/SendUsShipments "SendUsShipments Repository")  for more information.

### Push

(To be continued...)

Content
-------

See examples:

* Slim format: [SlimSample.xml](SlimSample.xml).
* Full format: [Sample.xml](Sample.xml). 