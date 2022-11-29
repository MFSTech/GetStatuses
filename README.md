Get Statuses
============

EDI (API) Documentation for the Standard, XML-Based, Shipment Status Information for Customers

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

At this time, secure transmission using HTTPS (SSL) is not available.

These status transmissions are always available. No prior configuration by us is required. You may implement on your own schedule.

### Push

We are able to generate status transmissions based on a large number of triggers in our shipment tracking application. All of the following types of events are able to generate a status transmission:

1. Freight or Service Statuses
2. Exceptions
3. Scheduling Calls
4. Scheduling
5. Administrative Activity
6. Shipment Closure
7. Customer-Facing Shipment Note Entry
8. Extra Service Adjustments

We run a process every few minutes which generates the transmissions. The status transmissions generated from these events always use the "Full" version of the content. The content can delivered directly to an HTTP or HTTPS endpoint as an HTTP POST request. The body of the HTTP POST will contain the XML in "text/xml" format (*not* "application/x-www-form-urlencoded" format).

Alternatively, our system can generate files containing the XML content. Another process which runs every few minutes can sweep these files to an FTP site we host, or it can send the files via FTP to a site hosted elsewhere. We believe it's best practice for the receiver to host the FTP site. Typically, it is more convenient and uses fewer system resources to poll a local FTP site than a remote FTP site. Also, receiver FTP host software can often trigger processing events.

You can request that we establish Push status transmissions to you by emailing PFD.EDI@PilotDelivers.com.

Content
-------

Each status transmission has information about a single shipment. The root XML element is `StatusTransmission`. It has some transmission-related header information and then a large `Shipment` section, which contains the statuses, call log, and other items. The Full format contains more information inside the Shipment section.

### Header Elements

These elements are direct descendants of the root `StatusTransmission` node.

* `TransmissionID` - This is a unique integer created by our system to represent this particular transmission. It can be useful for debugging purposes.
* `TransmissionDate` - This is the date and time (in ISO 8601 format) the transmission content was generated, in the local time of our server.
* `TransmissionDateUTC` - This is the date and time (in ISO 8601 format) the transmission content was generated, in Coordinated Universal Time (also known as UTC or GMT).
* `Shipment` - This houses the shipment information; it is the bulk of the transmission.

### Shipment Element

Within the `Shipment` element are sections containing general shipment information along with each of the lists of various the event types. The following elements are direct descendants of the `Shipment` element.

* `ControlNumbers`
* `Milestone`
* `ServiceInfo`
* `Parties`
* `Dates`
* `Airports`
* `Pieces`
* `PieceSummary`
* `CurrentStatus`
* `StatusList`
* `ComEvents`
* `ExtraServices`
* `Activities`
* `Exceptions`
* `Charges`
* `ChargesSummary`
* `TextInfo`
* `Images`

#### ControlNumbers

This section contains the main tracking numbers for the shipment, along with some other relevant identification numbers. It is sparsely populated in the Slim format, containing only the primary BOL number (our main tracking number) and our internal transaction ID for the shipment. In the Full format, it contains the other standard customer tracking numbers, such as PO number and Reference number. It also includes the invoice number, if the customer has been invoiced for the shipment.

* `TranID` - Unique, inviolate, integer, internal transaction ID for the shipment.
* `BOL` - Bill of Lading number. This is our primary textual tracking number.
* `Ref` - Reference number.
* `PO` - Purchase Order number.
* `RMA` - Return Merchandise Authorization number.
* `SO` - Sales Order number.
* `Invoice` - Invoice number. Only populated if the customer has been invoiced for the shipment.
* `CustomCtrlNums` - See below.

##### CustomCtrlNums

Only available in the Full format, Custom Control numbers are special control numbers defined for particular processes or customers. (Technically, most standard control numbers and custom control numbers are the same from the system's perspective. We simply expose the standard numbers differently.) The type of control number and the value are supplied as follows.

* `CtrlNumType`
* `CtrlNumValue`

#### Milestone

The Milestone status is designed to provide a high level, simple summary of the status of the shipment. It corresponds with with the broad categories defined in our Clarity web tracking application. Those categories are as follows.

* Acknowledged - We have the shipment information, and some activity might have taken place, but we have not yet picked up the freight (and no later activity has occurred).
* In Transit - We have picked up the freight and it is moving to the destination market. We have not yet placed it on the final delivery vehicle.
* Out For Delivery - We have placed the freight on the final delivery vehicle. 
* Delivered - The freight has been delivered to the final recipient (aka consignee).
* Exceptions - The shipment has been closed for a reason other than a normal delivery. For example, the shipment might have been refused by the recipient.

These milestone explanations (and many of our other status explanations) are written from the perspective of freight movement, but they often have analogous steps in a service-only transaction. For example, "Delivered" would really mean "Installed" if we were only performing installation services.

When the Milestone is an Exception, additional information will be included about the Exception. The additional information is actually the shipments closure type (when not closed as Delivered). An example is "Exception: Damaged - Refused", where a recipient refused a delivery because it was damaged.

The `Milestone` element is available in the Slim and Full formats.

#### ServiceInfo

The `ServiceInfo` element contains information about the type of services we are to perform, along with the speed in transit the customer has selected.

* `ServiceTypeID` - A textual code representing the speed in transit requested for the shipment. Examples include Ground (R) and Next Day (1). 
* `ServiceType` - The name of the speed in transit selection. See also `ServiceTypeID`.
* `ShipmentTypeID` - A textual code representing the collection of services we are to perform on the special side of a shipment. Most shipments are direction Delivery and the services will be performed at destination. Some shipments are direction Pickup and the services should be performed at origin. Example service bundles include Threshold Plus (TP), White Glove (WG), and White Glove with Assembly (WA).
* `ShipmentType` - The name of the collection of services we are to perform on the special side of the shipment. See also `ShipmentTypeID`.
* `CloseType` - If the shipment is complete, this will explain the final disposition. Most of the time, it will be closed as "POD", indicating it has been delivered (or that the installation has been completed). When there has been an Exception, this will show the specific issue (such as "Damaged - Refused"). For incomplete shipments, this will read "Not Closed".

Note the shipment Direction is not currently included, but likely will be in the future, and it will likely be an element in this `ServiceInfo` section.

The `Serviceinfo` section is available in the Slim and Full formats.

#### Parties

There are four primary parties to a shipment-- Shipper, Consignee, Bill-To, and Controller. For some Service Types (e.g. True Last Mile), the Delivery Agent will also be present.

* `Shipper` - This is the origin of shipment and generally represents the location at which we took possession of it.
* `Consignee` - This is the destination of the shipment and generally represents the location at which we transfer possession to another party.
* `BillTo` - This is the party paying for the shipment. Typically, it is the same as the `Controller`, but sometimes there are different rate agreements with a customer and the specific rates which apply are based on this party.
* `Controller` - This is the party which controls the shipment. From a system perspective, it also drives the selection of available Shipment Types, Service Types, etc.
* `DeliveryAgent` - This is the supplier we have selected to perform the final delivery to the `Consignee`. It is only present for True Last Mile shipments.

In some cases, the shipment does not involve actual movement of goods (e.g. an installation only service), in which case either the origin or destination address will be largely irrelevant.

All of the parties share the same data structure.

The parties are only available in the Full format.

##### Party Structure

* `Name` - This is the primary name of the party. It is equivalent to the `Address`\`BusinessName` element.
* `ID` - This is the unique textual identifier of the party. For the `BillTo`, `Controller`, and `DeliveryAgent`, this is set by us. For the `Shipper` and `Consignee`, it can be set by the customer, but it is generally defaulted. It is also only unique across all addresses for the customer not across all customers.
* `Email` - This is the email of the primary contact for the party.
* `Address` - This contains the address (street, city, state, and zip) for the party. `Region` is generally only relevant for non-US addresses.
* `Phones` - This contains the various phone numbers for the primary contact referenced in `Address`. 

#### Dates

* `ETD` - This is date the shipment is available for pickup. It is also known as the Ready Date. It drives rate selection (in case there are different rates for different time periods). For True Last Mile shipment, this is the date the shipment will begin its journey to us.
* `ETA` - This is date the shipment is required to be available for delivery to the consignee in order to honor the `ServiceType`.
* `Schedule` - This is the date on which the services bundled in `ShipmentType` are to be performed. In that sense, it is sensitive to Direction. The time represents the end of the window as defined by `WindowStop`.
* `WindowStart` - This is the time on the `Schedule` date which represents the start of the window in which the delivery and/or services are to be performed.
* `WindowStop` - This is the time on the `Schedule` date which represents the end of the window in which the delivery and/or services are to be performed.
* `Invoice` - This is the date on which the shipment was invoiced. This is only available in the Full format.
* `Closed` - This was the date on which the shipment was closed. When the `CloseType` is "POD", it will correspond with the `StatusDateTime` of the  "Delivery" status.

The `Dates` section is available in the Slim and Full formats.

#### Airports

The `Origin` and `Destination` airport codes represent the nearest service terminals to the `Shipper` and `Consignee`, based on some proprietary routing logic. They were historically used for customer rating, but are now generally just informational. They are also available only in the Full format.

#### Pieces

`Pieces` encloses the list of `Piece` nodes. It is available only in the Full format.

A Piece is a separately identified shipping unit, which definition leaves a gray area. Two separate cartons sitting on the floor near each other are clearly two separate Pieces. Slide those boxes together; surround them with black shrink wrap; attach a "Do Not Break Down" sign; and use metal banding to adhere them to a pallet; and they would usually be considered one Piece.

The configuration of Pieces often has a pricing impact. Smaller, independently mobile Pieces allow more efficient vehicle loading. Bundled cartons, treated as a single piece, are less efficient. Those bundled cartons are considered to have dimensions equal to the cube formed by the maximum height, length, and width of the combination.

When multiple cartons are grouped into a single shipping Piece, it's often useful to identify the number of cartons contained in the Piece, in case the package integrity is compromised during transit. Customers can specify this in the `PieceSTC` (Piece Said to Contain) element of the [Shipment transmission](https://github.com/MFSTech/SendUsShipments "SendUsShipments Repository"). That quantity is not currently represented in the below Piece information, but it likely will be in the future.

We do not serialize to down to the Piece level. We allow customers to group like Pieces. All Pieces of the same product type with the same dimensions and weight can be grouped into a single Piece Line. Each Piece Line is separately stored in our system. A Piece Line is represented here as a `Piece` element. 

As a practical matter, it is unusual to have multiple, identically configured carton bundles, so generally if multiple cartons are shipping as a single Piece, it will be represented with its own Piece Line.

`Piece` sub-nodes:

* `PieceID` - This is our unique numeric identifier for the Piece Line record.
* `PieceCount` - This is the number of like Pieces represented by this Piece Line.
* `PieceLength` - This is the length-- in inches-- of Pieces in this Piece Line.
* `PieceWidth` - This is the width-- in inches-- of Pieces in this Piece Line.
* `PieceHeight` - This is the height-- in inches-- of Pieces in this Piece Line.
* `PieceWeightActual` - This is the actual (as opposed to dimensional) weight-- in pounds-- of *each* Piece in this Piece Line. The `PieceCount` should be multiplied by the `PieceWeightActual` to determine the total actual weight of the Piece Line. 
* `PieceWeightDimensional` - This is the dimensional weight of the shipment. Dimensional weight is a conceptual value designed to represent impact of the size of a Piece in a way comparable to its weight. The `PieceCount` should be multiplied by the `PieceWeightDimensional` to determine the total dimensional weight of the Piece Line. The dimensional weight is calculated by taking the product of the length, width, and height (in inches) and dividing it by the dimensional factor. The dimensional factor can vary by customer agreement and/or mode of transport. Typically, 250 is used as a dimensional factor for ground shipments. As an example, a 20x30x40 carton would have a dimensional weight of 96 pounds. Typically, a customer is charged by the greater of the actual or dimensional weight for a shipment.
* `PieceDeclaredValue` - This is the declared value of *each* Piece in this Piece Line. The `PieceCount` should be multiplied by the `PieceDeclaredValue` to determine the total declared value for the Piece Line. The declared value is generally the maximum amount for which we would be liable in the case of loss or damage. Often, customers are charged based on the amount declared.
* `ProdType` - This is a textual code for the type of product contained in the Piece. It will be one of a list of product types configured for a customer. Generally, customers shipping TVs are required to specify the product type of the TV, which will be a code representing the TV technology and size. An example would be "LED-052" for a 52" LCD display with an LED back-light. 
* `ProdID` - This is an optional field sometimes specified by customers. It often corresponds with the SKU of the product. 
* `ProdName` - This is an optional long form name of a product. An example might be "52-inch LCD Display with LED Back-Light".

Most of these elements correspond with similarly named elements in the [Shipment transmission](https://github.com/MFSTech/SendUsShipments "SendUsShipments Repository").

#### PieceSummary

`PieceSummary` contains aggregate information about the pieces on the shipment. It is only available in the Full format.

* `PieceCount` - Total number of pieces-- the sum of the values in the `PieceCount` elements.
* `WeightActual` - Total actual weight of the shipment-- the sum of the product of the values in the `PieceWeightActual` and `PieceCount` elements.
* `WeightDimensional` - Total dimensional weight of the shipment-- the sum of the product of the values in the `PieceWeightDimensional` and `PieceCount` elements.
* `WeightChargeable` - This is the greater of `WeightActual` and `WeightDimensional`.

#### CurrentStatus

This contains a copy of the `Status` element in `StatusList` which is the last status (by datetime) of the latest type. This is *not* necessarily the status or other event which triggered the transmission. It is available in both the Slim and Full versions.

#### StatusList

Available in both the Slim and Full versions, this contains a full list of all statuses logged on the shipment. A `Status` is typically a real world freight movement event, such as picking up the freight from the shipper or delivering it to the consignee.

A full list of status types are available here:

[http://edi.mfsclarity.com/EDI/XMLIn.aspx?Process=GetStatusTypes](http://edi.mfsclarity.com/EDI/XMLIn.aspx?Process=GetStatusTypes).

* `StatusID` - This is our unique numeric identifier for the status record.
* `StatusType` - This is a short textual identifier for the type of status event, such as "Pickup" or "Delivery". For non-delivery closure exceptions, this will contain "Closure".
* `StatusTypeDesc` - This is the name of the status event, such as "Proof of Pickup" or "Proof of Delivery". For non-delievry closure exceptions, this will contain the closure type name, such as "Cancelled Before Freight Activity".
* `StatusDateTime` - This is the *local* date and time at which the event occurred.
* `StatusLocation` - The system uses a variety of methods to attempt to derive the city and state around where the status event occurred.
* `StatusNotes` - Available only in the Full version, this will contain any notation entered on the status record or closure information. Typically, this will be a portion of the name of the individual receiving the freight.

#### ComEvents

Available in both the Slim and Full versions, this containsa a full list of all communication events logged on the shipment, such as a call by one of our staff or an automated call, text, or email.

A `ComEvent` contains the following:

* `ComDate` - This is the date and time at which the communication event occurred, in the Central US time zone.
* `ComType` - This is the type of communication event, such as a call, text, or email, often indicating the result of the event (such as the shipment being scheduled or a message being left).
* `ComNotes` - This includes any notes related to the event, including the email address contacted. This element is only available in the Full version.

#### ExtraServices

Available in Slim and Full versions, each `ExtraService` requested for the shipment is listed here.

* `ExtraServiceTypeID` - A unique textual identifier for the type of extra service.
* `ExtraServiceTypeName` - The name of the extra service type.
* `ExtraServiceValue` - A quantity relevant to the extra service. For most extra services, this value is 1.

#### Activities

Only available in the Full version, this contains a list of all system activities logged on the shipment. Typical activities include "Customer Processed", when a customer processed the shipment information via EDI or our web portal Clarity, and "Customer Invoiced", when our accounting system created an customer invoice for the shipment.

Each `Activity` node contains:

* `ActivityTypeNumber` - A number identifying the type of activity.
* `ActivityTypeID` - A short textual identifier for the type of activity.
* `ActivityTypeName` - The name of the type of activity.
* `ActivityDate` - The date and time at which the activity was logged in our system, in the Central US time zone.
* `ActivityNote` - Any textual information logged as part of the activity event.

#### Exceptions

Only availalbe in the Full version, this contains a list of all publicly available exceptions logged on the shipment, including when a shipment is rescheduled.

These are also known as "Service Logs" and a complete list is available here:

[http://edi.mfsclarity.com/edi/xmlin.aspx?Process=GetServiceLogTypes](http://edi.mfsclarity.com/edi/xmlin.aspx?Process=GetServiceLogTypes)

Each `Exception` contains:

* `ExceptionTypeID` - A textual identifier for the type of exception.
* `ExceptionTypeName` - The name of the exception.
* `ExceptionDate` - The date and time at which the exception was logged in our system, in the Central US time zone. (Occasionally, this value is adjusted to match another event.)
* `ExceptionNote` - Any textual information logged as part of the exception.

#### Charges

Only available in the Full version, this contains a list of the charges logged on the shipment. The charges are only estimates prior to invoicing.

Each `Charge` contains:

* `ChargeID` - A unique numeric identifier for the charge.
* `ChargeTypeID` - A unique textual identifier for the type of charge.
* `ChargeTypeName` - The name of the type of charge.
* `ChargeCategory` - The category of the charge (such as "Freight", "Fuel", "Delivery", or "Install").
* `ChargeTypeFactorID` - The textual identifier for the charge method (aka factor type).
* `ChargeTypeFactorName` - The name of the charge method (aka factor type). An example is "Freight Charge", which means that the Charge Value is equal to the Charge Amount of the charge of type "Freight Charge". Another option is "Flat", where the Charge Value is an integer, usually 1.
* `ChargeRate` - The rate by which the Charge Value is multiplied to calculate the final Charge Amount.
* `ChargeValue` - The value determined by the charge method and other shipment characteristics (such as the chargeable weight).
* `ChargeMin` - The minimum amount for this type of charge for this customer.
* `ChargeAmount` - The final charge amount-- a product of the Charge Rate and Charge Value (sometimes scaled by a factor of 0.01, especially for the Freight Charge).

#### ChargesSummary

Only available in the Full version, this contains a summary of the charges logged on the shipment.

* `Freight` - The total of all charges of category "Freight".
* `Fuel` - The total of all charges of category "Fuel".
* `Insurance` - The total of all charges of category "Insurance".
* `Transport` - The total of all charges of category "Transport".
* `Delivery` - The total of all charges of category "Delivery".
* `Service` - The total of all charges of category "Service".
* `Other` - The total of all charges of all other categories not listed above.
* `TotalCharges` - The total of all charges (the sum of all the ChargeAmount element values, which should equal the sum of all the sibling elements to TotalCharges under ChargesSummary.)

#### TextInfo

Only availalbe in the Full version, this contains collection of textual information about the shipment.

* `PackageDescription` - The Package Description provided by the customer, as modified by our staff and processes.
* `SpecialInstructions` - The Special Instructions provided by the customer, as modified by our staff and processes.
* `PubNotes` - A collection of public notes.

##### PubNote

* `PubNoteID` - A unique numeric identifier for the public note.
* `PubNoteInsertDate` - The date and time at which the public note was created.
* `PubNoteText` - The textual contents of the public note.

#### Images

Only available in the Full version, this contains a list of all images attached to the shipment.

Each `Image` contains:

* `ImageID` - A unique numeric identifier for the image.
* `ImageTypeID` - A unique textual identifier for the type of image.
* `ImageType` - The name of the image type.
* `ImagePath` - The URL to access the image.
* `ImageInserted` - The date and time at which the image was attached, in the Central US time zone.

### Examples

We have provided examples of real world shipments, scrubbed of any identifying information.

* Slim format: [SlimSample.xml](SlimSample.xml)
* Full format: [Sample.xml](Sample.xml).
