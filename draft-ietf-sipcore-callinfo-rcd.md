---
title: SIP Call-Info Parameters for Rich Call Data
abbrev: Call-Info Rich Call Data
docname: draft-ietf-sipcore-callinfo-rcd-latest
submissiontype: IETF
category: std

ipr: trust200902
area: art
keyword: sipcore
keyword: SIP
keyword: Identity

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
-
  ins: C. Wendt
  name: Chris Wendt
  org: Somos Inc.
  email: chris-ietf@chriswendt.net
  country: US
-
  ins: J. Peterson
  name: Jon Peterson
  org: Neustar Inc.
  email: jon.peterson@neustar.biz
  street: 1800 Sutter St Suite 570
  city: Concord, CA  94520
  country: US

normative:
  RFC2392:
  RFC3261:
  RFC3325:
  RFC3966:
  RFC3968:
  RFC4627:
  RFC5234:
  RFC6350:
  RFC7095:
  RFC7340:
  RFC7519:
  RFC7852:
  RFC8224:
  RFC8225:
  I-D.ietf-stir-passport-rcd:
  W3C-SVGTiny1.2:
    title: Scalable Vector Graphics (SVG) Tiny 1.2 <https://www.w3.org/TR/SVGMobile/>
    author:
      organization: W3C
    date: 2008-12-22
informative:

--- abstract

This document describes a usage of the SIP Call-Info header field that incorporates Rich Call Data (RCD) associated with the identity of the calling party in order to provide to the called party a description of the caller or details about the reason for the call. RCD includes information about the caller beyond the telephone number such as a calling name, or a logo, photo, or jCard object representing the caller, which can help the called party decide whether to answer the phone. The elements defined for this purpose are intended to be extensible in order to accommodate related information about calls and to be compatible and complimentary with the STIR/PASSporT RCD framework.

This document defines a new parameter ('call-reason') for the SIP Call-Info header field and also a new token ("rcd-jcard") for the 'purpose' parameter of the Call-Info header field. It also provides guidance on the use of the Call-Info 'purpose' parameter token, "icon".

--- middle

# Introduction

Signaling protocols in telephone networks have long supported the delivery of a 'calling name' from the originating side to the terminating side, though in practice, the terminating side is often left to derive a name from the calling-party number by consulting a local address book or an external database. SIP {{RFC3261}} similarly can carry a 'display-name' in the From header field value from the originating to terminating side, though it is an unsecured field that is not commonly trusted and is often replaced or ignored. The same can be considered true of information in the Call-Info header field in SIP.

To allow calling parties to initiate, and called parties to receive, a more comprehensive, deterministic, and extensible Rich Call Data (RCD) for incoming calls, this document defines a new parameter ('call-reason') for the SIP Call-Info header field {{RFC3261}} and also a new token ("rcd-jcard") for the 'purpose' parameter of the Call-Info header field. For this document and depending on the policies of the communications system, a calling party could be either the end user device (e.g., a SIP user agent (UA)) or a network service as part of a telephone service provider. Similarly, a called party could be an end user device or the network telephone service provider acting on behalf of the recipient of the call.

Used on its own, this specification assumes that the called party UA can trust the SIP network or the SIP provider to assign, deliver, and protect the correct RCD information as an end-to-end security policy.  However, as is true in many interconnected communications services, this end-to-end trust cannot be guaranteed. Therefore, the recommended approach is that the entity inserting the Call-Info header field should also sign the caller information via STIR-defined protocol tools {{RFC7340}} for SIP {{RFC8224}} and specifically through the use of RCD or the "rcd" PASSporT defined in {{I-D.ietf-stir-passport-rcd}}.

Alternatively, this specification can be utilized in conjunction with the protocols defined in {{I-D.ietf-stir-passport-rcd}} as part of the communications signaling path, specifically in the trusted UNI device interface at the terminating side as part of an authenticated, network-to-device, trusted signaling where a device may not have the ability to verify the "rcd" PASSporT, but it can receive the RCD information from the Call-Info header field as defined in this specification.

{{RFC7852}} provides a means of carrying additional data about callers for the purposes of emergency services (especially {{Section 4.4 (Owner/Subscriber Information) of RFC7852}}).  This specification provides an overlapping functionality for non-emergency cases.  Rather than overloading its "EmergencyCallData" Call-Info 'purpose' parameter value, this document defines a separate 'purpose' parameter for the more generic delivery of information via jCard {{RFC7095}}.  This document borrows from {{RFC7852}} the capability to carry a data structure as a body, through the use of the "cid" URI scheme {{RFC2392}}.

# Terminology {#terminology}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Overview

In this document, we provide a framework for the use of Call-Info header field to carry RCD in SIP {{RFC3261}}. The Call-Info header field (defined in {{RFC3261, Section 20.9}}) defines a 'purpose' parameter. In addition to providing guidance on calling name practices and the use of the existing 'purpose' parameter token,	"icon", this document expands on other types of RCD by defining a new 'purpose' token, "rcd-jcard", and a new generic parameter, 'call-reason', for the Call-Info header field to align with RCD as defined in the STIR framework {{RFC8224}} and with "rcd" PASSporTs defined in {{I-D.ietf-stir-passport-rcd}}.

The 'purpose' parameter token "rcd-jcard" is used to associate RCD related to the identity of the calling party in the form of a jCard {{RFC7095}}. While there is a "card" token defined in {{RFC3261}} which could be considered to have an overlapping purpose, the "rcd-jcard" token is intended to denote the jCard profile defined in this document for use in the Call-Info header field for RCD. The choice of jCard in this specification is guided by two things. First, JSON has become the default and is generally the widely accepted, optimally supported format for transmission, parsing, and manipulation of data on IP networks, and jCard represents an extensible method of providing information about a person or business associated with a call. Second, jCard has been defined in {{I-D.ietf-stir-passport-rcd}} and has been adopted by PASSporT {{RFC8225}} because of the usage of JSON Web Tokens (JWT) {{RFC7519}}.

The new Call-Info header field parameter 'call-reason' provides a string or other object that conveys the caller's intent or reason for calling to help the called party understand the context and intent of the call and why they may want to answer the call.

# A Call-Info Framework for Carrying Rich Call Data

This specification extends the Call-Info header field to be compatible and complimentary to the RCD framework defined in {{I-D.ietf-stir-passport-rcd}}. Typically, a SIP-based call involves multiple hops through different trusted and untrusted networks. The STIR framework {{RFC7340}} addresses the protection of the carriage of call information and identities over untrusted networks, which wasn't addressed in the core SIP specifications.  {{RFC3261, Section 20.9}} defines the Call-Info header field as the mechanism for carrying call- and caller-related information and also provides procedures for defining new 'purpose' parameter tokens. This document discusses the use of existing tokens and defines a new 'purpose' token to correspond to the RCD framework.

There are a number of RCD information types that can be transmitted in the Call-Info header field of a SIP request.  The STIR RCD specification {{I-D.ietf-stir-passport-rcd}} defines calling name, a logo or icon associated with the caller, and a call reason string. It also discusses an extensible way of carrying caller information using jCard {{RFC7095}}. It may be that future specifications extend information types and, similar to how this document extends the Call-Info header field to provide corresponding functionality to STIR RCD, it is RECOMMENDED that future specifications also provide corresponding Call-Info extensions.

The RCD framework defined both in this document as well as in {{I-D.ietf-stir-passport-rcd}} carries call-specific information. The insertion of RCD is intended to be singular in that the receiving party should not be required to make any call-specific decisions based on redundant, duplicate, or conflicting RCD. With the use of the Call-Info header field for the transmission of RCD, any RCD-related information defined in this specification or future specifications that extend this mechanism MUST be contained in a single Call-Info header field containing all URI and 'purpose' tokens and parameters related to RCD. The RCD information is either intended to be added by a party that is authoritative over that information or to have been translated from a verified STIR RCD PASSporT and unmodified once in a trusted domain. Any additional parties involved in the call path SHOULD NOT modify the Call-Info header field or add additional Call-Info header fields related to RCD. The insertion of the RCD Call-Info header field should be considered a trusted action based on trusted information, and the information SHOULD NOT be considered modifiable as a best practice.

As discussed in {{I-D.ietf-stir-passport-rcd}}, the calling name uses the display-name value of the From header field {{RFC3261}} of the request. Alternatively, for some calls, the calling name may come from the P-Asserted-ID header field {{RFC3325}}.  This is out of scope for Call-Info header field, so will not be covered in this document further.

For logos or icons that can represent the calling party, the 'purpose' token "icon" {{RFC3261}} is used to indicate a URI for an image resource that can be displayed to the user receiving the SIP request.  For the purpose of this document and the transmission of RCD, the "icon" 'purpose' token should be used as defined.  Section 8.2 provides high-level guidance on image formatting and related information.

This document defines 'call-reason' as a new parameter for the Call-Info header field. This parameter carries a string indicating the reason for the call.

jCard is a comprehensive and extensible mechanism defined in the STIR RCD framework. While {{RFC3261}} specifies a "card" 'purpose' token, the intent of defining a new "rcd-jcard" 'purpose' token is to use the JSON jCard format {{RFC7095}} and to provide guidance for the use and non-use of jCard attributes to describe the calling party in a communications session as well to provide some security considerations around that information.  These topics are covered in the next sections.

# "rcd-jcard" Call-Info 'purpose' Token

The Call-Info 'purpose' token "rcd-jcard" indicates support of RCD associated with the identity of a calling party in a SIP call {{RFC3261, Section 20.9}}.  The format of a Call-Info header field when using the "rcd-jcard" token is as follows.

The Call-Info header field is defined to include a URI that points to a resource that is a jCard JSON object {{RFC7095}}. The media type for the JSON text MUST be set as application/json with a default encoding of UTF-8 {{RFC4627}}. This MAY be carried directly in the Call-Info header field URI using the "data" URI scheme. A jCard also MAY be carried in the body of the SIP request bearing this Call-Info header field via the "cid" URI scheme {{RFC2392}}. Alternatively, the URI MUST define the use HTTPS or a transport that can validate the integrity of the source of the resource as well as the transport channel through which the resource is retrieved. If, in the specific deployment environment of SIP, the source or integrity of the RCD information cannot be trusted, then the use of the STIR RCD framework defined in {{I-D.ietf-stir-passport-rcd}} should be considered.

The jCard is intended to contain multiple information elements about the calling party.  A call and its corresponding single RCD-related Call-Info header field MUST only contain a single "rcd-jcard" token.

The fields like "fn", "photo", or "logo" if used with the use of "icon" calling name in From or P-Asserted-ID header field or purpose token, as described in the previous section, MUST either match or be avoided to allow the called party to clearly determine the intended calling name or icon.

An example of a Call-Info header field is:

~~~~~~~~~~~~
Call-Info: <https://example.com/qbranch.json>;purpose=rcd-jcard
~~~~~~~~~~~~

An example of the contents of a URL-linked jCard JSON file is shown as follows:

~~~~~~~~~~~~
["vcard",
  [
    ["version",{},"text","4.0"],
    ["fn",{},"text","Q Branch"],
    ["org",{},"text","MI6;Q Branch Spy Gadgets"],
    ["photo",{},"uri","https://example.com/photos/q-256x256.png"],
    ["logo",{},"uri","https://example.com/logos/mi6-256x256.jpg"],
    ["logo",{},"uri","https://example.com/logos/mi6-64x64.jpg"]
  ]
]
~~~~~~~~~~~~

An example SIP INVITE using the "data" URI scheme is as follows:

~~~~~~~~~~~~
INVITE sip:alice@example.com SIP/2.0
Via: SIP/2.0/TLS pc33.atlanta.example.com;branch=z9hG4bKnashds8
To: Alice <sip:alice@example.com>
From: Bob <sip:12155551000@example.com;user=phone>;tag=1928301774>
Call-ID: a84b4c76e66710
Call-Info: <data:application/json,["vcard",[["version",{},"text",
"4.0"],["fn",{},"text","Q Branch"],["org",{},"text","MI6;Q Branch
Spy Gadgets"],["photo",{},"uri","https://example.com/photos/quart
ermaster-256x256.png"],["logo",{},"uri","https://example.com/log
os/mi6-256x256.jpg"],["logo",{},"uri","https://example.com/logos/
mi6-64x64.jpg"]]]\>;purpose=rcd-jcard;call-reason="Rendezvous for
Little Nellie"
CSeq: 314159 INVITE
Max-Forwards: 70
Date: Fri, 25 Sep 2015 19:12:25 GMT
Contact: <sip:12155551000@gateway.example.com>
Content-Type: application/sdp

v=0
o=UserA 2890844526 2890844526 IN IP4 pc33.atlanta.example.com
s=Session SDP
c=IN IP4 pc33.atlanta.example.com
t=0 0
m=audio 49172 RTP/AVP 0
a=rtpmap:0 PCMU/8000
~~~~~~~~~~~~

An example SIP INVITE using the "cid" URI scheme is as follows:

~~~~~~~~~~~~
INVITE sip:alice@example.com SIP/2.0
Via: SIP/2.0/TLS pc33.atlanta.example.com;branch=z9hG4bKnashds8
To: Alice <sip:alice@example.com>
From: Bob <sip:12155551000@example.com;user=phone>;tag=1928301774>
Call-ID: a84b4c76e66710
Call-Info: <cid:12155551000@example.com>;purpose=rcd-jcard;call-reason=
  "Rendezvous for Little Nellie"
CSeq: 314159 INVITE
Max-Forwards: 70
Date: Fri, 25 Sep 2015 19:12:25 GMT
Contact: <sip:12155551000@gateway.example.com>
Content-Type: multipart/mixed; boundary=boundary1
Content-Length: ...

--boundary1

Content-Type: application/sdp

v=0
o=UserA 2890844526 2890844526 IN IP4 pc33.atlanta.example.com
s=Session SDP
c=IN IP4 pc33.atlanta.example.com
t=0 0
m=audio 49172 RTP/AVP 0
a=rtpmap:0 PCMU/8000

--boundary1

Content-Type: application/json
Content-ID: <12155551000@example.com>

["vcard",[["version",{},"text","4.0"],["fn",{},"text","Q Branch"],
["org",{},"text","MI6;Q Branch Spy Gadgets"],["photo",{},"uri","ht
tps://example.com/photos/quartermaster-256x256.png"],["logo",{},"u
ri","https://example.com/logos/mi6-256x256.jpg"],["logo",{},"uri",
"https://example.com/logos/mi6-64x64.jpg"]]]
~~~~~~~~~~~~

# 'call-reason' Call-Info Parameter

This specification defines a new parameter that extends the overall content of the RCD-related Call-Info header field.  As other parameters may be defined in the future, this parameter is intended to be separate and distinct from the other URI and 'purpose' tokens that may proceed these parameters.

This new parameter of the Call-Info header field is called 'call-reason'. The 'call-reason' parameter is intended to convey a short textual message suitable for display to an end user during call alerting. As a general guideline, this message SHOULD be no longer than 64 characters; displays that support this specification may be forced to truncate messages that cannot fit onto a screen. This message conveys the caller's intention in contacting the callee. It is an optional parameter, and the sender of a SIP request cannot guarantee that its display will be supported by the terminating endpoint. The manner in which this reason is set by the caller is outside the scope of this specification.

An alternative approach would have been to use the value of Subject header field {{RFC3261}} to convey the reason for the call. However, because the Subject header field has seen little historical use in SIP implementations and its specification describes its potential use in filtering, it seemed prudent to define a new means of carrying a call reason indication.

An example of a Call-Info header field value with the "call-reason" parameter follows:

~~~~~~~~~~~
Call-Info: <https://example.com/jbond.json>;purpose=rcd-jcard;
  call-reason="For your ears only"
~~~~~~~~~~~

In the case that there is only a 'call-reason' parameter or any future parameters that may be defined and no need for a purpose parameter with no associated URI, it is RECOMMENDED to include a null data URI, "data:" as the URI. That purpose parameter MUST be "rcd-jcard" defined in this document to avoid any conflicts with existing implementations and previously defined purpose parameters.  As an example:

~~~~~~~~~~~
Call-Info: <data:>;purpose=rcd-jcard;
  call-reason="For your ears only"
~~~~~~~~~~~

# Usage and an Example of Call-Info for RCD

The procedures for the usage of URIs and 'purpose' parameter tokens should generally follow the procedures defined in {{RFC3261}}. The following example of a Call-Info header field shows the use of multiple 'purpose' parameters to indicate a jCard and an icon and also a 'call-reason' parameter:

~~~~~~~~~~~
Call-Info: <https://example.com/jbond.json>;purpose=rcd-jcard,
  <https://example.com/jbond.png>;purpose=icon;
  call-reason="For your ears only"
~~~~~~~~~~~


# Usage of jCard and Property-Specific Usage

Beyond the definition of the specific properties or JSON arrays associated with each property, this specification defines a few rules above and beyond {{RFC7095}} that are specific to the use of jCard for Call-Info and RCD to ensure there is a minimum level of supported properties to which every implementation of this specification should adhere. This includes support for interpreting the value of these properties and the ability to render in some appropriate form the display capabilities of common telephone devices as well as applications, and also includes requirements specific to textual and graphics-capable displays.

## Usage of URIs in jCard

When one or more URIs are used in a jCard, it is important to note that any URI-referenced data, with the exception of the top-level usage of "jcl" as a URI to the jCard itself (unless updated by any future extensions of this specification) MUST NOT contain any URI references. In other words, the jCard can have URI references as defined in the jCard specification and this document, but the content referenced by those URIs MUST NOT have any URIs, and therefore MUST be enforced by the client to not follow those URI references or not render that content to the user if any URI are present in that specific URI linked content. The purpose of this is to control the security and more specifically to align with the content-integrity mechanism defined in {{I-D.ietf-stir-passport-rcd}}. The authors do not believe there is a scenario for which deeper URI references would be required or even supported by the typical use of current jCard properties. However, because jCard is extensible, this rule is set to restrict further extension without the proper consideration of security and integrity properties of both Call-Info usage as well as the RCD and STIR signing of the data {{I-D.ietf-stir-passport-rcd}} {{RFC8224}}.

## Usage of Multimedia Data in jCard or with Icon {#multimedia-data}

With the use of the purpose token "icon" or for the cases where jCards incorporate URIs or directly include via Base64 encoding of digital images and sounds. We specify a few recommended conventions to facilitate more consistent support of the successful decoding and rendering of these images or media formats.

For images, such as for the photo and logo properties, the default image formats SHOULD be png or jpg. These files are mostly commonly used to support 24-bit RGB images which should consequently be the default.  There are some older telephone devices that may only support bmp type of images with lower bit-range (e.g., 16-bit or 8-bit or 1-bit), also with potentially only grayscale or 1-bit black and white color displays.  These exceptions are should considered optional to support or even recommended not to support and at least at the time of writing this document are becoming increasingly rare (i.e., typically displays on devices are either color or color-aware graphical displays that support png or jpg formats or exclusively textual displays).

In addition, vector images are increasingly popular in their use for icons and the need for scalable images without having to send multiple resolutions. SVG format and a minimum of support for {{W3C-SVGTiny1.2}} specifically appropriate for this specification has gained wide support as of the writing of this document, as a common format for vector images and should be supported as an additional default format for devices that support this specification.

For the cases where image files are referenced by URIs as file resources, this document defines a character string that SHOULD be concatenated on to the end of a file name, before the file extension that signals the height and width of the image to the end device for the convenience of determining the appropriate resolution to retrieve without the need to retrieve all the image files. It is also recommended that images are square ratio formatted with equal height and width and with a power of two value for the number of pixels (e.g., 32x32, 128x128, 512x512). The format of the string should be "filename-HxW" where filename represents the unique string representing the file and H represents the height in pixels and W represents the width in pixels.

It is appropriate and useful to include multiple versions of images or sounds so that endpoints that may not support all formats or resolutions can select the format they support.  The convention that is RECOMMENDED is that files that refer to the same content should use the same filename portion.  If it is an image format that has a specific resolution, the HxW portion of the filename should correspond to the pixel resolution. The file extension should reference the file type (e.g., filename.png, filename.svg, or filename.jpg) or (e.g., filename-32x32.png, filename-64x64.png, filename.svg, filename-32x32.jpg, or filename-64x64.jpg).

Because this is a complex and often debated topic that has evolved over the many years of advances in image coding and display technologies, we likely suggest relying on either future specifications or industry forum specifications that might correspond to supporting particular classes of devices to further define how URIs can reference appropriate image formats and files.

For audio files, the recommendation is to provide mp3, m4a, or wav files, although the usage of sound is not well defined in this specification as, for example, a special ring tone for a particular caller, and future documents should consider both usage and potential security risks of playing sounds that are not specifically authorized by a device user.

## Cardinality

Property cardinalities are indicated, for convenience, using the following notation and follow the guidance of jCard {{RFC7095}} and vCard {{RFC6350}}, which is based on ABNF (see {{RFC5234, Section 3.6}}):

~~~~~~~~~~~
  +-------------+--------------------------------------------------+
  | Cardinality | Meaning                                          |
  +-------------+--------------------------------------------------+
  |      1      | Exactly one instance per jCard MUST be present.  |
  |      *1     | Exactly one instance per jCard MAY be present.   |
  |      1*     | One or more instances per jCard MUST be present. |
  |      *      | One or more instances per jCard MAY be present.  |
  +-------------+--------------------------------------------------+
~~~~~~~~~~~

## Identification Properties
These types are used to capture information associated with the identification and naming of the entity associated with the jCard. They are initially defined in {{RFC6350}}, but the following list of properties included and repeated in this Section is a subset of the properties defined for jCard with properties selected for this document that have relevance to telephone and messaging applications. jCard is an extensible object and therefore, there may also be future specifications that extend the set of properties that may be relevant to the set of communications applications that utilize this specification.

### "fn" Property

The "fn" property has the intent of providing a formatted text corresponding to the name of the object the jCard represents.  Reference {{RFC6350, Section 6.2.1}}.

Value type:  A single text value.

Cardinality:  1*

~~~~~~~~~~~~
Example:
["fn", {}, "text", "Mr. John Q. Public\, Esq."]
~~~~~~~~~~~~

### "n" Property

The "n" property has the intent of providing the components of the name of the object the jCard represents. Reference {{RFC6350, Section 6.2.2}}.

Value type:  A single structured text value. Each component can have multiple values.

Cardinality:  *1

~~~~~~~~~~~~
Example:
["n", {}, "text", "Public;John;Quinlan;Mr.;Esq."]
["n", {}, "text", "Stevenson;John;Philip,Paul;Dr.;Jr.,M.D.,A.C.P."]
~~~~~~~~~~~~

### "nickname" Property

The "nickname" property has the intent of providing the text corresponding to the nickname of the object the jCard represents. Reference {{RFC6350, Section 6.2.3}}.

Value type:  One or more text values separated by a COMMA character (U+002C).

Cardinality:  *

~~~~~~~~~~~~
Example:
["nickname", {}, "text", "Robbie"]
["nickname", {}, "text", "Jim,Jimmie"]
["nickname", {}, "text", "TYPE=work:Boss"]
~~~~~~~~~~~~

### "photo" Property

The "photo" property has the intent of supplying an image or photograph information that annotates some aspect of the object the jCard represents. Reference {{RFC6350, Section 6.2.4}}.

In addition to the definition of jCard, and to promote interoperability and proper formatting and rendering of images, the photo SHOULD correspond to a square image size of the sizes 128x128, 256x256, 512x512, or 1024x1024 pixels.

Value type:  A single URI.

Cardinality:  *

~~~~~~~~~~~~
Example:
["photo", {}, "uri", "http://www.example.com/jqpublic-256x256.png"]
~~~~~~~~~~~~

## Delivery Addressing Properties

These properties are concerned with information related to the delivery addressing or label for the jCard object.

### "adr" Property

The "adr" property has the intent of providing the delivery address of the object the jCard represents. Reference {{RFC6350, Section 6.3.1}}.

Value type:  A single structured text value, separated by the SEMICOLON character (U+003B).

Cardinality:  *

~~~~~~~~~~~~
Example:
["adr", {"type":"work"}, "text",
  ["", "", "3100 Massachusetts Avenue NW", "Washington", "DC",
  "20008", "USA"]
~~~~~~~~~~~~

## Communications Properties

These properties describe information about how to communicate with the object the jCard represents.

### "tel" Property

The "tel" property has the intent of providing the telephone number for telephony communication of the object the jCard represents. Reference {{RFC6350, Section 6.4.1}}.

Relative to the SIP From header field value this information may provide alternate telephone number or other related telephone numbers for other uses.

It is important to note that any of the potential instances of the "tel" property should not be considered part of the authentication or verification part of STIR {{RFC8224}} or required to match the "orig" claim in the PASSporT {{RFC8225}}.  These telephone numbers should be considered for contact, fax, or other purposes aligned with the general usage of jCard and vCard, although consideration of confusing the caller with different contact telephone number information versus the actual verified telephone number should be made from a general policy point of view.

Value type:  By default, it is a single free-form text value (for backward compatibility with vCard 3), but it SHOULD be reset to a URI value.  It is expected that the URI scheme will be "tel", as specified in [RFC3966], but other schemes MAY be used.

Cardinality:  *

~~~~~~~~~~~~
Example:
["tel", { "type": ["voice", "text", "cell"], "pref": "1" }, "uri",
  "tel:+1-202-555-1000"]
["tel", { "type": ["fax"] }, "uri", "tel:+1-202-555-1001"]
~~~~~~~~~~~~

### "email" Property

The "email" property has the intent of providing the electronic mail address for communication of the object the jCard represents. Reference {{RFC6350, Section 6.4.2}}.

Value type: A single text value.

Cardinality: *

~~~~~~~~~~~~
Example:
["email", {"type":"work"}, "text", "jqpublic@xyz.example.com"]
["email", {"pref":"1"}, "text", "jane_doe@example.com"]
~~~~~~~~~~~~

### "lang" Property

The "lang" property has the intent of providing the language(s) that may be used for contacting of the object the jCard represents. Reference {{RFC6350, Section 6.4.4}}.

Value type:  A single language-tag value.

Cardinality:  *

~~~~~~~~~~~~
Example:
["lang", {"type":"work", "pref":"1"}, "language-tag", "en"]
["lang", {"type":"work", "pref":"2"}, "language-tag", "fr"]
["lang", {"type":"home"}, "language-tag", "fr"]
~~~~~~~~~~~~

## Geographical Properties

These properties are concerned with information associated with geographical positions or regions associated with the object the jCard represents.

### "tz" Property

The "tz" property has the intent of providing the time zone of the object the jCard represents. Reference {{RFC6350, Section 6.5.1}}.

Note: the up-to-date reference for where time-zone names are maintained is, at the authoring of this document, at this web address, https://www.iana.org/time-zones.

Value type:  The default is a single text value.  It can also be
   reset to a single URI or utc-offset value.

Cardinality:  *

~~~~~~~~~~~~
Example:
["tz", {}, "text", "Raleigh/North America"]
~~~~~~~~~~~~

### "geo" Property

The "geo" property has the intent of providing the global positioning of the object the jCard represents. Reference {{RFC6350, Section 6.5.2}}.

Value type:  A single URI.

Cardinality:  *

~~~~~~~~~~~~
Example:
["geo", {}, "uri", "geo:37.386013,-122.082932"]
~~~~~~~~~~~~

## Organizational Properties

These properties are concerned with information associated with characteristics of the organization or organizational units of the object that the jCard represents.

### "title" Property

The "title" property has the intent of providing the position or job of the object the jCard represents. Reference {{RFC6350, Section 6.6.1}}.

Value type:  A single text value.

Cardinality:  *

~~~~~~~~~~~~
Example:
["title", {}, "text", "Research Scientist"]
~~~~~~~~~~~~

### "role" Property

The "role" property has the intent of providing the position or job of the object the jCard represents. Reference {{RFC6350, Section 6.6.2}}.

Value type:  A single text value.

Cardinality:  *

~~~~~~~~~~~~
Example:
["role", {}, "text", "Project Leader"]
~~~~~~~~~~~~

### "logo" Property

The "logo" property has the intent of specifying a graphic image of a logo associated with the object the jCard represents. Reference {{RFC6350, Section 6.6.3}}.

Value type:  A single URI.

Cardinality:  *

~~~~~~~~~~~~
Example:
["logo", {}, "uri", "http://www.example.com/abccorp-512x512.jpg"]

["logo", {}, "uri", "data:image/jpeg;base64,MIICajCCAdOgAwIBAgIC
      AQEEBQAwdzELMAkGA1UEBhMCVVMxLDAqBgNVBAoTI05ldHNjYXBlIENvbW11bm
      ljYXRpb25zIENvcnBvcmF0aW9uMRwwGgYDVQQLExNJbmZvcm1hdGlvbiBTeXN0
      <...the remainder of base64-encoded data...>"]
~~~~~~~~~~~~

### "org" Property

The "org" property has the intent of specifying the organizational name and units of the object the jCard represents. Reference {{RFC6350, Section 6.6.4}}.

Value type:  A single structured text value consisting of components separated by the SEMICOLON character (U+003B).

Cardinality:  *

~~~~~~~~~~~~
Example:
["org", {}, "text", "ABC\, Inc.;North American Division;Marketing"]
~~~~~~~~~~~~

## Explanatory Properties

These properties are concerned with additional explanations, such as that related to informational notes or revisions specific to the jCard.

### "categories" Property

The "categories" property has the intent of specifying application category information about the object the jCard represents. Reference {{RFC6350, Section 6.7.1}}.

Value type:  One or more text values separated by a COMMA character
   (U+002C).

Cardinality:  *

~~~~~~~~~~~~
Example:
["categories", {}, "text", "TRAVEL AGENT"]

["categories", {}, "text", "INTERNET,IETF,INDUSTRY"]
~~~~~~~~~~~~

### "note" Property

The "note" property has the intent of specifying supplemental information or a comment about the object the jCard represents. Reference {{RFC6350, Section 6.7.2}}.

Value type:  A single text value.

Cardinality:  *

~~~~~~~~~~~~
Example:
["note", {}, "text", "This fax number is operational 0800 to 1715
             EST\, Mon-Fri."]
~~~~~~~~~~~~

### "sound" Property

The "sound" property has the intent of specifying a digital sound content information that annotates some aspect of the object the jCard represents. This property is often used to specify the proper pronunciation of the name property value of the jCard. Reference {{RFC6350, Section 6.7.5}}.

Value type:  A single URI.

Cardinality:  *

~~~~~~~~~~~~
Example:
["sound", {}, "uri", "http://www.example.com/pub/logos/abccorp.mp3"]

["sound", {}, "uri", "data:audio/basic;base64,MIICajCCAdOgAwIBAgICBE
      AQEEBQAwdzELMAkGA1UEBhMCVVMxLDAqBgNVBAoTI05ldHNjYXBlIENvbW11bm
      ljYXRpb25zIENvcnBvcmF0aW9uMRwwGgYDVQQLExNJbmZvcm1hdGlvbiBTeXN0
      <...the remainder of base64-encoded data...>"]
~~~~~~~~~~~~

### "uid" Property

The "uid" property has the intent of specifying a globally unique identifier corresponding to the object the jCard represents. Reference {{RFC6350, Section 6.7.6}}.

Value type:  A single URI value.  It MAY also be reset to free-form text.

Cardinality: *1

~~~~~~~~~~~~
Example:
["uid", {}, "uri", "urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6"]
~~~~~~~~~~~~

### "url" Property

The "url" property has the intent of specifying a uniform resource locator associated with the object the jCard represents. Reference {{RFC6350, Section 6.7.8}}.

There is potential security and privacy implications of providing URLs with telephone calls. The end client receiving a jCard with a URL property MUST only display the URL and not automatically follow the URL or provide automatic preview of the URL, and generally provide good practices in making it clear to the user it is their choice to follow the URL in a browser context consistent with all of the common browser security and privacy practices available on most consumer OS environments.

Value type:  A single uri value.

Cardinality:  *

~~~~~~~~~~~~
Example:
["url", {}, "uri", "https://example.org/french-rest/chezchic.html"]
~~~~~~~~~~~~

### "version" Property

The "version" property MUST be included and is intended to specify the version of the vCard specification used to format this vCard. Reference {{RFC6350, Section 6.7.9}}.

Value type:  A single text value.

Cardinality:  1

~~~~~~~~~~~~
Example:
["version", {}, "text", "4.0"]
~~~~~~~~~~~~

# Extension of jCard

Part of the intent of the usage of jCard is that it has its own extensibility properties where new properties can be defined to relay newly defined information related to a caller.  This capability is inherently supported as part of standard extensibility.  However, usage of those new properties should be published and registered following {{RFC7095, Section 3.6}} or new specifications.

# IANA Considerations {#IANA}

## SIP Call-Info Header Field 'purpose' Parameter Token

This document defines the token "rcd-jcard" as a new value for the 'purpose' parameter of the Call-Info header field in the "Header Field Parameters and Parameter Values" registry defined by {{RFC3968}}.

~~~~~~~
  +--------------+----------------+-------------------+----------------+
  | Header Field | Parameter Name | Predefined Values | Reference      |
  +--------------+----------------+-------------------+----------------+
  | Call-Info    | purpose        | Yes               | [this RFC]     |
  +--------------+----------------+-------------------+----------------+
~~~~~~~

## SIP Call-Info Header Field 'call-reason' Parameter

This document defines the 'call-reason' generic parameter for use as a new parameter in the Call-Info header field in the "Header Field Parameters and Parameter Values" registry defined by {{RFC3968}}. The parameter's token is "call-reason", and it takes the value of a quoted string.

~~~~~~~
  +--------------+----------------+-------------------+----------------+
  | Header Field | Parameter Name | Predefined Values | Reference      |
  +--------------+----------------+-------------------+----------------+
  | Call-Info    | call-reason    | No                | [this RFC]     |
  +--------------+----------------+-------------------+----------------+
~~~~~~~

# Security Considerations {#Security}

Revealing information such as the name, location, and affiliation of a person necessarily entails certain privacy risks. SIP and Call-Info has no particular confidentiality requirement, as the information sent in SIP is in the clear anyway. Transport-level security can be used to hide information from eavesdroppers, and the same confidentiality mechanisms would protect any Call-Info or jCard information carried or referred to in SIP.

The security framework of signing and providing integrity to this data should be followed {{I-D.ietf-stir-passport-rcd}}, with the idea that the use of constraints and other certificate based associations should be considered. This includes considerations around information about the calling party being generally constant vs per call data being more temporal. This also includes the relationship that certificates with constraints presents to how they relate to each other and how that information is managed, protected, and associated with the correct call corresponding to a calling party.

--- back

# Acknowledgements {#Acknowledgements}
{: numbered="false"}

We would like to thank David Hancock, Alec Fenichel and other members of the SIPCORE and STIR working group for helpful suggestions and comments for the creation of this draft.
