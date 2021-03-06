---
title: WebRTC JavaScript Object RTC
# abbrev: WebRTCJSObj
docname: draft-raymond-rtcweb-webrtc-js-obj-rtc-00
date: 2013-08-12
category: std

ipr: trust200902
area: General
workgroup: Network Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: R. Raymond
    name: Robin Raymond
    org: Hookflash
    street: 436, 3553 31 St. NW
    city: Calgary
    region: Alberta
    code: T2L 2K7
    email: robin@hookflash.com
 -
    ins: C. Dorn
    name: Christoph Dorn
    org: Independent
    street: 1999 Highway 97 South
    city: West Kelowna
    region: BC
    code: V1Z 1B2
    country: Canada
    email: christoph@christophdorn.com
 -
    ins: E. Lagerway
    name: Erik Lagerway
    org: Hookflash
    street: 436, 3553 31 St. NW
    city: Calgary
    region: Alberta
    code: T2L 2K7
    country: Canada
    email: erik@hookflash.com
 -
    ins: I. Baz Castillo
    name: Inaki Baz Castillo
    org: Versatica
    street: Barakaldo
    city: Basque Country
    country: Spain
    email: ibc@aliax.net
 -
    ins: R. Shpount
    name: Roman Shpount
    org: TurboBridge
    street: 4905 Del Ray Ave Suite 300
    city: Bethesda
    region: MD
    country: USA
    code: 20814
    email: rshpount@turbobridge.com


normative:
  I-D.ietf-rtcweb-use-cases-and-requirements: <!-- Web Real-Time Communication Use-cases and Requirements -->
  I-D.ietf-rtcweb-rtp-usage: <!-- Web Real-Time Communication (WebRTC) - Media Transport and Use of RTP -->
  RFC3550:  <!-- RTP -->
  RFC5245:  <!-- ICE -->
  I-D.rescorla-mmusic-ice-trickle: <!-- trickle ICE -->
  MediaStreams:
    target: http://www.w3.org/TR/2013/WD-mediacapture-streams-20130516/
    title: Media Capture and Streams
    author:
      name: Daniel C. Burnett
      ins: D. Burnett
    date: 2013-03-16

informative:
  I-D.ietf-rtcweb-jsep: <!--JSEP -->
  I-D.jesup-rtcweb-data-protocol: <!-- WebRTC Data Channel Protocol -->
  I-D.raymond-rtcweb-webrtc-js-obj-api-rationale: <!-- WebRTC JavaScript Object API Rationale -->
  I-D.ietf-avtcore-6222bis: <!-- Guidelines for Choosing RTP Control Protocol (RTCP) Canonical Names (CNAMEs) -->
  I-D.ietf-mmusic-sdpng: <!-- Requirements for Session Description and Capability Negotiation -->
  I-D.roach-rtcweb-plan-a: <!-- Using SDP with Large Numbers of Media Flows -->
  I-D.uberti-rtcweb-plan: <!-- Plan B a proposal for signaling multiple media sources in WebRTC. -->
  I-D.ivov-rtcweb-noplan: <!-- No Plan - Economical Use of the Offer/Answer Model in WebRTC Sessions with Multiple Media Sources -->
  WebRTC10:
    target: http://www.w3.org/TR/2012/WD-webrtc-20120821/
    title: WebRTC 1.0 Real-time Communication Between Browsers
    author:
      name: Adam Bergkvist
      ins: A. Bergkvist
    date: 2012-08-21
  MediaCapture:
    target: http://www.w3.org/TR/2013/WD-mediacapture-streams-20130516/
    title: Media Capture and Streams
    author:
      name: Daniel C. Burnett
      ins: D. Burnett
    date: 2013-05-29
  WebRTCJSObjExamples:
    target: http://tools.ietf.org/html/draft-raymond-rtcweb-webrtc-js-obj-rtc-examples-00
    title: WebRTC JavaScript Object RTC Examples
    author:
      name: Robin Raymond
      ins: R. Raymond
    date: 2013-07-05
  Jingle:
    target: http://xmpp.org/extensions/xep-0166.html
    title: XEP-0166 Jingle
    author:
      name: XMPP Standards Foundation
      ins: XMPP Standards Foundation
    date: 2009-12-23
  RFC3261: <!-- SIP -->
  RFC4566: <!-- SDP -->
  RFC3264: <!-- An Offer/Answer Model with the Session Description Protocol (SDP) -->
  RFC3711: <!-- srtp -->
  RFC5389: <!-- stun -->
  RFC5766: <!-- turn -->
  RFC6347: <!-- dtls 1.2 -->
  RFC4960: <!-- SCTP -->
  RFC4568: <!-- sdes -->
  RFC2119: <!-- keywords -->
  RFC4733: <!-- DTMF -->


--- abstract

This draft outlines an alternative approach to the current JSEP {{I-D.ietf-rtcweb-jsep}} / WebRTC 1.0 draft {{WebRTC10}} and removes the dependency requirements on the SDP {{RFC4566}} and Offer / Answer {{RFC3264}} state machine or reliance upon an all-encompassing master media description format that describes the entire RTP transport and media states. This draft describes how the WebRTC Use-cases and Requirements {{I-D.ietf-rtcweb-use-cases-and-requirements}} and Media Transport and Use of RTP {{I-D.ietf-rtcweb-rtp-usage}} are satisfied by such an alternative approach (where the use cases are applicable). The rationale for needing an alternative approach is described in the WebRTC JS Object API Rationale draft {{I-D.raymond-rtcweb-webrtc-js-obj-api-rationale}}.

This draft outlines how WebRTC signaling negotiation information required to perform RTC on-the-wire can be organized into object-like constructs to minimize cross-dependencies between negotiated information, round-trip signaling, and co-dependency in negotiated state transitions. This results in greater flexibility for signaling protocols other than those based on SDP/SIP {{RFC3261}} (but not to exclude SDP/SIP). The flexibility to implement other signaling options is achieved by not imposing an all-encompassing session description format and the offer / answer state machine, such as described in JSEP {{I-D.ietf-rtcweb-jsep}}. Finally, this draft outlines how a reference shim can be implemented on top of this proposal for those who would prefer a JSEP / SDP model for their signaling.

--- middle

Introduction        {#problems}
============

This draft describes how an object-like construct model can be used to control the setup, manage, negotiate and control Real-Time Communications (RTC) from the Browser and how these concepts are transform into on-the-wire constructs, as an alternative to JSEP's approach. The provided example-only API is presented merely as input guidelines for the W3C to ensure their API constructs allow for the correct interpretation to on-the-wire RTC control and signaling constructs needed as defined in the various use case drafts related to RTCWEB. The draft outlines an alternative JavaScript RTC control model that does not require a complex all-encompassing session description format, like SDP, or related offer / answer state machine to describe the entire transport and media states. This allows for greater flexibility with other non-SDP offer / answer based signaling protocols. The actual final API is entirely within the scope of the W3C (as it should be).


General Design of WebRTC JS Object Model
----------------------------------------

The design of this object model is to split the information required to setup a RTC session into logical object components, information and relationships where they are needed rather than on requiring a single all-encompassing RTC session description format and its related offer / answer state machine. Further, the coordination of the information that must be exchanged in signaling is left to the decision of the JavaScript developer to allow for a variety of signaling approaches, rather than imposing a single model with offer / answer.

By splitting the RTC concepts into logical objects components, the choice of approaches for the signaling path becomes even more flexible and the information needed for each component becomes minimized and much easier to standardize. The local object components can be replaced in the future should the information required for each component change, or if a new type of object is needed for different types of media or RTP engine features.

This WebRTC JavaScript object model approach still exposes all the key information required for various signaling models, but limits the information needed to only the information required to actually control the media engine and nothing more. For the average JavaScript developer, the objects model is simplistic but still allows those developers who require greater control and flexible access to the constructs they require from an RTC engine inside the browser without needing to perform extensive parsing, mangling and regeneration of a master know-everything session format (such as SDP).

The main design goals are to:

1. allow multiple alternative on-the-wire signaling alternatives without imposing a know-everything media description format and related offer / answer state machine.
2. mandate only the use of the RTC on-the-wire protocols needed to satisfy the use cases and requirements, as described in {{I-D.ietf-rtcweb-use-cases-and-requirements}} and related documents.
3. allow granular information to be exchanged based upon logical objects in a method and timing decided best by the implementor.
4. allow for control of the RTC engine with maximum flexibility, while still adhering to the unique security requirements of browsers.
5. minimize the requirements needed for basic and simple JavaScript use cases.
6. resolve the PlanA {{I-D.roach-rtcweb-plan-a}}, PlanB {{I-D.uberti-rtcweb-plan}} and NoPlan {{I-D.ivov-rtcweb-noplan}} conflicts arising from handling multiple media sources.
7. borrow liberally from the concepts already proposed in other places where possible to merge the various concepts.
8. resolve or limit as many of the concerns presented in the accompanying rationale document {{I-D.raymond-rtcweb-webrtc-js-obj-api-rationale}} as possible.


Other Approaches Considered
-------------------------

By not imposing a master transport and media description format, a JavaScript developer will not be exposed to a complex unfamiliar session description format with little high level control over what's happening at protocol layers. The current SDP JSEP approach requires developers to perform extensive parsing, interpret or transform this master session description when more control is required. Decoupling a single describe-everything format and the state machine into logical well-defined components with no specific requirements on how the information must be exchanged allows for greater flexibility in signaling.

SDP is a complex format that describes many more things than are actually required to control an RTP engine. By not imposing a particular format, the net result should be less brittle implementations that are not dependent upon an ever evolving, adaptable and expanding master format.

Contrary to the assertions stated in JSEP {{I-D.ietf-rtcweb-jsep}} "1.2 Other Approaches Considered", the WebRTC JavaScript object model proposed does not result in overly complex JavaScript code to perform simple and basic WebRTC implementations. The simple scenarios for RTC control from JavaScript are no more complex than the simple JSEP examples and do not require dependency on additional libraries to create a reasonable reference implementation. Whereas the complexity required to implement other more advanced scenarios that do not follow the specific offer / answer state machine model is greatly reduced relative to JSEP's requirements. The manual manipulation and transformation of a complex session description format generated by the browser is all but eliminated. Only those who must implement a particular type of session format will be required to parse and generate that format without imposing that format upon others, and amongst those who need a particular format, their output will be tailored to their exacting needs.

The WebRTC JS Object Model does not implement or add any features other than those already existing in WebRTC 1.0, but because of it's granular object based approach, it allows for a variety of alternative signaling scenarios that are difficult to achieve with the current WebRTC API {{WebRTC10}}. In fact, the requirements for a browser are eased by virtue of not having to handle an all-encompassing media description format with an offer / state machine. Both a JSEP or object based model would need to perform all the same logic routines internally, including all the RTP data muxing rules, but the WebRTC object model removes many of the unneeded restrictions and burdens placed on a JavaScript developer and the browser vendor on how a signaling protocol must work.


Terminology
===========

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 {{RFC2119}}.



Object, Data and Control Model
==============================


RTC Socket {#objsocket}
----------

This object represents a local media IP port opened to send and receive media or data. Each port is capable of carrying RTP, RTCP, or DTLS/SCTP streams but the mapping of what each socket must be under the control of the Javascript layer.

The developer must be able to provide each of these sockets a list of stun and turn servers. This allows the socket to discover other network-reflective port mappings and open up relay network sockets to allow the socket to be "remoted" when direct media connections to the socket are not possible.

The socket must support DTLS {{RFC6347}} / SCTP {{RFC4960}}), the developer's JavaScript must be able to must be able to obtain the fingerprint utilized in the private / public certificate pair generated by the browser (for further discussion on this issue see the Security Section ({{fingerprintresponsibility}})).
354

RTC Connection {#objconnection}
--------------

A connection is an object that represents the logical connection between two Internet peers. Each peer constructs a connection object that gathers candidates (i.e. possible methods to connect over the Internet) from an RTC socket and allows an external signaling mechanism to exchange these candidates to each other. The peers test the candidates for connectivity and establish a connection once a candidate pairing discovers connectivity (see {{I-D.rescorla-mmusic-ice-trickle}}).

Once a connection is established on all sockets, an event must be fired to the developer's JavaScript / application to indicate connectivity. The connection remains established until disconnected by programmatic intent or by timeout failure, upon which time a disconnect event must be fired to indicate the disconnection state with the appropriate listed reason. Should the connection fail to connect, a disconnection event must fire with the appropriate listed reason.

Once a connection is established between peers, a connection can be used to transmit media or data utilizing the associated media or data socket. Appropriate capabilities ({{objcapabilities}} / stream mappings) for each connection must be exchanged before media streams may flow between peers.


### Relationship to Socket ###

Each connection {#objconnection} object must be related to a single socket ({#objsocket}), and multiple connections can share the same socket objects. This allows a single socket to be connected to multiple peers with a unique connection to each peer. A socket represents a single logical connection binding on a network interface, thus a unique ICE ({{objice}}) negotiation occurs for each connection since ICE negotiation represents a local to remote socket connection negotiation.

### Connection Description ### {#objconnectiondescription}

While a connection might have a variety of other properties, every connection object has a description structure that must have the following properties:

  * connection-id - an identifier to distinguish between connection objects in stream/track mappings.
  * iceUsernameFrag - a cryptographic random identifier for the connection auto-generated by the browser for use as the ICE userFrag.
  * icePassword - a cryptographic random string serving as the ICE password.
  * fingerprint - the hash fingerprint of any public keys certificate used in the sockets (see {{fingerprintresponsibility}}).

The ICE username frag, password and fingerprint must remain identical for every connection using the same RTC Socket object to allow for forking scenarios. This allows the developer to correlate forked connections as appropriate by a JavaScript developer.

The information contained in a connection's description must be exchanged with a remote peer's connection object before any candidate connectivity checks may be performed by the browser. The exact method and timing of the exchange must be left to the discretion of the JavaScript developer.

#### Connection Description Expressed as a Structure ####

This is an example-only connection description expressed as a JavaScript structure. This example is not a signaling on-the-wire format mandate.

    var myConnectionDescription = {
        connection-id: "<value>",
        iceUsernameFrag: "<random-id>",
        icePassword: "<random>",
        fingerprint: "<hash>"
    };

#### Connection Description Negotiation ####

The local peer has a connection description. The remote peer has a connection description. Each peer must exchange the information in the connection description with the other through signaling. The the iceUsernameFrag and the icePassword are used in the for ICE signaling process (see {{RFC5245}}).

Each connection has a fingerprint for each socket DTLS {{RFC6347}} / SCTP {{RFC4960}}. The fingerprint must be signaled in order to for each connection to ensure it's connected to the correct remote peer. The connection object is responsible for ensuring any remotely connected peer's fingerprint during DTLS exchange matches the fingerprint provided in the connection description of the remote peer. If the fingerprint does not match, the connection must be closed.

Example pseudo-code of how connection negotiated could be accomplished:

    // alice makes offer
    var alice_connection = new ORTC.Connection();
    var aliceDesc = alice_connection.getDescription();
    
    mysignal("to:bob", JSON.stringify(aliceDesc));
    
    var bob_connection = new ORTC.Connection();
    bob_connection.setDescription(JSON.parse(aliceDesc), "remote");


### ICE ###      {#objice}

ICE {{RFC5245}} is a standard used for discovering peer connectivity. A browser must support both ICE, and the ICE modification extension trickle ICE {{I-D.rescorla-mmusic-ice-trickle}} connectivity discovery mechanisms. The browser may support additional peer connectivity methods other than ICE.

When the browser detects a candidate is available, the candidate must be sent as an event to the developer's JavaScript. If all possible candidates have been discovered, the browser must send an end of candidates event to the developer's JavaScript.


#### ICE Candidate Trickling ####

The browser must support trickle ICE {{I-D.rescorla-mmusic-ice-trickle}} discovery mechanism. This mechanism allows the browser to provide candidate information as they are discovered, thus allowing the developer's JavaScript to exchange this information to a remote peer via signaling. As candidates are exchanged, the browser must perform connectivity checks as defined by trickle-ICE even while other candidates are still being discovered and exchanged via signaling (to allow for faster connection times).

Before any connectivity checks may be performed by a browser, a connection's description information ({{objconnectiondescription}}) must be received from a remote peer's connection.


#### ICE Candidate Description ### {#objcandidatedescription}

Each ICE candidate has description information containing the following:

  * connection id - the connection identifier where the candidate was discovered, matching the identifier specified in the RTC Connection ({{objconnection}})
  * foundation (as as defined in RFC5245 {{RFC5245}})
  * component id (as as defined in RFC5245)
  * transport (as as defined in RFC5245)
  * priority (as as defined in RFC5245)
  * connection address (as as defined in RFC5245)
  * connection port (as as defined in RFC5245)
  * type (defined as "host", srflx", "prflx", or "relay" whose meaning is derived from RFC5245)
  * related address (as as defined in RFC5245)
  * related port (as as defined in RFC5245)

The developer's JavaScript should be able to transport the candidate description information 'as is' even if the description information is not understood so long as the developer's signaling protocol allows for alternative candidate types.

The candidate descriptions may be relayed to the remote peer independent of other description information or bundled together with other descriptions as determined by the needs of the JavaScript developer. The browser must not impose restrictions on external signaling protocols related to the exchange of candidates, or require the bundling of other types of descriptions, such as capabilities descriptions ({{objcapabilities}}) or stream descriptions ({{objstreamdescriptions}}).


##### Candidate Description Expressed as a Structure #####

This is an example-only candidate description expressed as a JavaScript structure. This example is not a signaling on-the-wire format mandate.

    var myCandidateDescription = {
        connection-id: "<value>",
        foundation: 1,
        component: 1,
        transport: "udp",
        priority: 1694498815,
        connectionAddress: "192.0.2.33",
        connectionPort: 10000,
        type: "host"
        // relatedAddress: "1.2.3.4",
        // relatedPort: 34232
    };


### Capabilities ### {#objcapabilities}

Capabilities consist of a list of media codecs and RTC options allowed to be used as part of the transmission of media. Capabilities can consist of additional rules or limitations, for example; video size or bandwidth limitations. A developer's JavaScript must be able to obtain the types of capabilities supported by the browser ({{objfeaturediscovery}}), and the default set of codecs and algorithms available.

#### Capabilities Description Expressed as a Structure ####

This is an example-only capabilities description expressed as a JavaScript structure. This example is not a signaling on-the-wire format mandate. Individual codecs may have varying properties according to the definition of each codec type.

    var myCapabilities = {
        codecs: [
            {
                payloadId: 96,
                kind: "audio",
                name: "<name>",
                hzRate: 32000,
                channels: 1
                // ...
            },
            {
                payloadId: 96,
                kind: "video",
                name: "<name>",
                hzRate: 96000
                // ...
            }
        ],
        required: {
        },
        optional: {
            video: {
                maxWdith: 1280,
                maxHeight: 720
            }
        }
    };


#### Capability Negotiation ####

The media capabilities must be exchanged via some external signaling method by the developer's JavaScript. The local capabilities must be (at minimal) a subset of the available capabilities of the local and remote peer. Should the browser be unable to fulfill the provided capabilities for any reason (for example, none of the codecs were compatible), an error most be propagated to the developer's JavaScript, which should handle this condition rather than silently fail. The RTCWEB Working Group must define a mandatory common set of codecs to help alleviate the possibility of codec and algorithm failure conditions.

The browser must not dictate the timing or format of how capabilities are exchanged, nor the rules of how each peer must provide the information, nor how the information is signaled.

The browser should only support codecs that are well documented standards. The browser must namespace non-standard codecs using a method that will not cause confusion with codecs that might become standardized in the future. The properties for each standardized codec must be defined clearly. By defining each codecs' properties, alternative signaling protocols may translate the standardized codecs into their respective signaling protocols. If a browser does not have specific knowledge of a codec supplied by a developer's JavaScript, it must ignore the non-understood codecs. A JavaScript developer may encode and signal non-standard codecs so long as none of the browser provided properties are lost during signaling exchange.

Other non-codec capabilities and properties may be set for a connection. For example; a developer may wish to set the maximum video dimensions for a connection. The browser should only implement standardized capabilities and namespace non-standard capabilities to ensure that no conflict arises should capabilities become standardized.

A developer's JavaScript may choose what capabilities to signal to a remote peer, or not. The developer must only set capabilities that are either understood, or were generated by the browser and signaled 'as is' to the remote party. A developer should attempt to signal all capabilities generated by the browser to the remote party, but may choose to filter only the understood capabilities.

The browser should only implement standardized capabilities and must namespace non-standardized capabilities appropriately to avoid conflict with capabilities that might be standardized in the future.

Example pseudo-code of how capabilities could be negotiated:

    // alice signals capabilities to bob
    var aliceCapabilities = JSON.stringify(
      alice_session.getCapabilities("receive"));
    mysignal("to:bob", aliceCapabilities);
    
    // bob changes sending capabilities
    bob_connection.setCapabilities(
      JSON.parse(aliceCapabilities), "send");
    
    // bob signals capabilities to alice
    var bobCapabilities = JSON.stringify(
      bob_session.getCapabilities("receive"));
    mysignal("to:alice", bobCapabilities);
    
    // alice changes sending capabilities
    alice_session.setCapabilities(JSON.parse(bobCapabilities), "send");


### Media Stream (and Media Stream Track) Descriptions ### {#objstreamdescriptions}

RTP is the current standard for performing real-time media transmission. While RTP is effective at media transmission, the RTP transport lacks sufficient details to convert a WebRTC stream object into an RTP transmission and back into an WebRTC stream on the remote peer (where stream objects consist potentially of multiple audio, video (or other) media tracks).

Further, the exact mapping of which sockets and codecs are used is unspecified. RTP does have a SSRC (Synchronization source identifier) for every media stream sent over it which can be used to coordinate the RTP streams. A signaling layer must describe this coordination to be able to synchronize the streams from one peer to another.

Other features, like FEC (Forward Error Correction), are not described in WebRTC stream objects because they are a feature of a transport. Each stream could allow a media track to be sent redundantly on the wire so if part of the stream was damaged during the transmission it could be reconstructed from the redundant stream. Another case is simulcasting where the same media track is encoded multiple ways, for example video layering effects or various encoding sizes (thumbnails vs full streams).

RTP lacks sufficient information to be able to describe FEC or simulcasting as part of its transmission, thus a higher layer must signal this information to a remote peer if these features are to be supported.

The browser must provide a method to create a stream description for the features it supports, to allow for stream and track synchronization (and correlation), and other transport features like FEC or simulcast. The exact mapping structures for the stream must be clearly defined in exacting detail, describing the expected behaviors and use cases a browser must support.

For each track in a stream the browser must supply the following information:

  * track-id - the identifier of the track (i.e. from MediaStreamTrack.id {{MediaCapture}})
  * media-stream-id - the identifier of the associated media stream (i.e. from MediaStream.id {{MediaCapture}})
  * ssrc - the SSRC of the track as defined by RFC3550 {{RFC3550}}
  * kind - the kind of media being transported (meaning derived from MediaStreamTrack {{MediaCapture}})
  * connection id - the identifier correlating to the connection object ({{objconnection}})
  * codecs - (optional, the default send / receive codecs capabilities apply if not specified)

If FEC is supported by the browser and the developer's JavaScript has enabled the feature, the browser must supply the following additional information on FEC tracks:

  * the redundancy SSRC - the SSRC that is redundant to the current track

(NOTE: FEC is a complex subject with alternative methods to perform the same levels of redundancy. This draft takes no position on the best techniques and practices and only suggests one method for describing FEC within the steam description. As other description formats are able to support various FEC scenarios, the descriptive information for various FEC scenarios could easily be added into this draft depending on what the IETF/W3C wish to support by default.)

The browser must support a developer's JavaScript specifying capabilities per track on both the send and receive ends of the track. As each SSRC can be demuxed from other SSRCs at the RTP layer, the capabilities can be applied per track.

While these stream (and track) descriptions are intended to be sent to remote peers, the browser must not mandate the timing, format, or the signaling of these stream descriptions.


Example pseudo-code of how streams could be sent and signaled:

    // alice sends information about stream to bob
    var aliceStreamDesc = JSON.stringify(
      alice_session.getDescription(aliceMediaStreamToBob));
    
    alice_session.sendStream(aliceMediaStreamToBob);
    
    mysignal("to:bob", JSON.stringify(aliceStreamDesc));
    
    // bob receives stream
    var bobMediaStreamFromAlice = new MediaStream();
    bob_session.setDescription(
      bobMediaStreamFromAlice,
      JSON.parse(aliceStreamDesc)
      );
    bob_connection.receiveStream(bobMediaStreamFromAlice);


#### Media Stream Description Expressed as a Structure ####

This is an example-only media stream description expressed as a JavaScript structure. This example is not a signaling on-the-wire format mandate.

    var myStreamDescription = [
        { // audio
            track: "<track-id>",
            ssrc: 5,
            redundancySsrc: 10,
            connection-id: "<value>",
            capabilities: { /* optional capabilities */ }
        },
        { // video
            kind: "video",
            ssrc: 10,
            connection-id: "<value>",
            capabilities: { /* optional capabilities */ }
         },
         { // dtmf
            kind: "dtmf",
            ssrc: 5,
            connectin-id: "<value>",
            capabilities: { /* optional capabilities */ }
         }
    ];


### Sending Media Streams (over RTP) ###

The browser must be able to support sending streams via connections {{objconnection}} over the socket {{objsocket}}. The developer's JavaScript must be able to indicate to the browser the stream and track mappings (in the form of a stream description).

The browser must respect the media stream track mapping to the socket {{objsocket}}, SSRC, and optional capabilities. The "kind" property for sending is merely information but is not explicitly required, but if set, must match the media stream track's "kind".

The developer's JavaScript must be able to control the exact timing when a stream starts to send and when a stream stops sending. At any time, the developer's JavaScript must be able to change the track mapping and capabilities without the browser imposing any signaling requirements to a remote peer, thus allowing for various alternative signaling scenarios.

The browser must only utilize the codecs within the capabilities provided for sending, and alternate codecs at will. A browser should decide which codec is best suited for the network conditions amongst all the codecs within the capabilities. If a developer's JavaScript wishes to limit to a particular codec, the capabilities should only contain the codec desired.

If FEC is supported, the browser must coordinate the sending of the FEC RTP redundancy tracks.


### Receiving Streams (over RTP) ###

The browser must be able to support receiving streams via connections {{objconnection}} over the socket {{objsocket}}. The developer's JavaScript must be able to specify the streams expectations for any streams about to be received (or currently being received).

The developer's JavaScript is not required to signal the stream mapping should the developer not care about stream and stream track reassembly, or if the developer will manually assemble the streams tracks into streams based on their own logic. If media tracks arrive without any stream definition, the browser must render the RTP streams as media stream tracks unattached to a particular stream. Should the developer's JavaScript eventually specify a stream definition, the browser must assemble the individual tracks into streams according to the stream definitions. The developer's JavaScript must be able to assemble individual stream tracks into streams manually should the developer not wish to provide a stream definition.

The browser must be able to render the same media stream track as part of multiple media stream objects, to allow a stream definition to contain the same track in multiple streams.

Each media stream track is demuxed first by the connection ({{objsocket}}), then by the SSRC, then by "kind" based upon the codec "kind" of the payload. While technically possible to demux upon the same "kind", the browser must only support demuxing the same media "kind" with the same SSRC if the payload id used only matches one set of capabilities for a particular track on the receiving stream. A developer's JavaScript should not mux the same "kind" of media on the same SSRC. A developer's JavaScript should not mux different "kinds" of media on the same SSRC.

A developer's JavaScript must provide a stream definition to receive any streams that utilize the FEC feature when transmitting over RTP.


### Data Stream Description ###  {#objdatastreamdescription}

A data stream is an object that is a specialized type of stream (see {{datastream}}), just as a media stream is an object for media related information. The data stream description contains the following properties:

  * stream id - the identifier for the data stream (from MediaStream.id {{MediaCapture}})
  * connection id - the connection to carry the data stream ({{objsocket}})

The browser must not define or specify how or when the signaling for the data stream description is sent.


### Sending Data Streams (over DataSocekt) ###

The browser must be able to send a data stream over the connection. For the simple case, the developer's JavaScript should be able to send the stream without any additional information being specified to allow for simple signaling scenarios where the data stream mapping is unimportant.


### Receiving Data Streams (over DataSocket) ###

In the receive direction, the browser must be able to open a data stream based upon the data stream description information from the remote peer. The browser should support a method to fetch the next available data stream without needing any specifier to handle simple signaling scenarios where specific stream mappings are unimportant.


RTC Feature Discovery {#objfeaturediscovery}
---------------------

The browser must have a method to allow a developer's JavaScript to determine what RTP features are available from the browser's RTC engine without performing browser and browser version detection. While some features can be probed by way of checking if an object exists to support the feature, other features cannot. For example; the developer's JavaScript should be able to probe the browser to answer "Does the RTC engine support FEC?"

At minimum, each feature only needs to be a well defined standardized string that determines the exact expectations, feature sets and behaviors that are supported by the browser's RTC engine. Should the browsers define custom extension features, the browser must namespace each feature in such a way to avoid non-standard features from conflicting with a potential future standardized definition.

RTC features can be vast and complex. Having a feature discovery is especially important to allow for a variety of implementations without setting the requirements bar too high on browser implementations. Developers also need a method to ensure (through JavaScript) they can detect which options are available to remain compatible. Performing unreliable browser probing techniques is not sufficient. Many RTC features are hidden behind complex rules in the engine and cannot be probed easily, for example, FEC.


Example: DTMFMediaStreamTrack Specialization    {#dtmf}
---------------------------------------------

One of the use case requirements in Web Real-Time Communication Use-cases and Requirements is DTMF navigation of IVR systems {{I-D.ietf-rtcweb-use-cases-and-requirements}}. Typically DTMF is performed by transmitting RTP data as described in RFC 4733 {{RFC4733}}.

One possible method to implement a DTMF data stream is by specialization of the media stream track. The stream description ({{objstreamdescriptions}}) does allow for mapping a track to an RTP mapping and DTMF does not require any special mapping, except ensuring the DTMF track shares the same SSRC as an audio track. The browser should create reasonable rules to auto-pair a DTMF stream track to an audio track for a stream description.

The browser engine must interpret signals given from the DTMF stream to override the audio stream to send DTMF packets as described in RFC 4733. The browser must define a special codec for DTMF with a kind defined as "dtmf".

Should the decision to interpret incoming DTMF tones be mandated by the RTCWEB WG then the browser should interpret received DTMF tones received as per RFC 4733 and fire events that a developer's JavaScript can intercept. For a use case example, as browser engines are being embedded in other technologies, such as Node.js, having the browser be capable of interpreting DTMF signals may allow for JavaScript server based IVR style-system when bundled with a relevant signal protocol.


Example: DataStream Specialization      {#datastream}
----------------------------------

A data stream object can be a specialized type of stream, just as media stream is used for sending media data. Data streams allow data to be transmitted over a data compatible socket ({{objsocket}}), such as DTLS/SCTP {{RFC6347}}/{{RFC4960}}. A data stream is a bi-direction stream allowing for each peer to send and receive data over the same data stream.

To open a data stream, one peer must initiate the stream through a browser supplied send method, and another peer must be able to receive an event in JavaScript about the new incoming data stream. The developer's JavaScript must be able to start receive data stream events for any sent data. Both peers via JavaScript must be able to send data to the other and receive data stream events.


Signaling Models
================

SDP with Offer / Answer
-----------------------

While it could be said that the current WebRTC 1.0 API {{WebRTC10}} is best suited for SDP / SIP, that might not be the case. Many SIP networks have a large investment in their SIP infrastructures and technology, and updating all of their technology is not simple. If what the browser produces for SDP is not 100% compatible, they must either convert to SDPs from the browser via JavaScript to SDPs that are compatible, or introduce other gateways that do the conversion. By removing SDP from the browser, this allows the SIP vendors to generate SDP specifically tailored to their SIP network, and more easily control the exact conditions in the browser they need to be compatible.

In other words, one could argue that SDP / SIP developers wouldn't need to do extra work to be compatible with the browsers, but for many, that simply might not be true. Having SDP generated in the browser allows a SIP network provider less control over what SDP is produced than having JavaScript produce the SDP as a dynamically updated downloaded module where all browsers produce the identical SDP. To explore this reasoning further please see the related rationale draft {{I-D.raymond-rtcweb-webrtc-js-obj-api-rationale}}.

This section will outline how JavaScript can generate an SDP from basic descriptions, just as an SDP parser can reconstruct the information each object's description requires. Yet these SDPs that are produced are custom tailored with exact specifications yielding greater compatibility than relying upon each browser to produce identical SDPs.


### Constructing an Offer ###

The first step required is creating a connection object. The connection object can optionally be constructed with various socket configurations for transport, such as separating audio and video RTP sockets, or muxing the data on the same socket as RTP. The developer can obtain the send capabilities and populate the "m=" line(s) for the media desired, as well as all the codec information. The ice userFrag and password information is available from the connection object. If the SDP must generate SSRC mappings, the "getDescription" method can be used to obtain the mapping information in order to generate the various SDP SSRC entries accordingly. Finally, the ICE candidates will be notified using an "onconnectioncandidate" event and the offer can either be generated before all candidates arrive, or after once all candidates are ready.

The peer can be put into an "offer" state.


### Parsing an Offer ###

When a remote peer receives an offer, the peer must construct an answer, but it must parse the offer before this. The first step is to parse out the SDP information as per the expectations of how it was generated. Since the offer was constructed with JavaScript and not by the browser, the exact parameters and fields will be exactly how the JavaScript decided with no surprises (outside of fixable JavaScript bugs). The remote peer can construct a connection object in the same manner as the offer peer. The SDP is extracted for its ICE credentials, where "setDescription" is called for the connection with the relevant information from the offer about the ICE userFrag, password and the ICE candidates can be set using "setRemoteConnectionCandidate". The codecs are pulled from the SDP and "setCapabilities" is called to indicate the "receive" capabilities. If SSRC mapping was supported, the SSRC maps can be pulled out of the SDP and set using "setDescription" with a "receive" stream object provided.


### Generating an Answer ###

To construct answer, the ICE credentials are pulled from the connection object using the "getDescription". The candidates will arrive on the "onconnectioncandidate" and can be placed in the SDP. The capabilities for the "send" can be obtained by calling "getCapabilities" for the "send" direction (and only mutually agreed codecs can be filtered in if desired). The "m=" line can be generated for the specific medias with the related "a=" codec mappings. If the SDP must generate SSRC mappings, the "getDescription" method can be used to obtain the mapping information in order to generate the various SDP SSRC entries in the answer.

The remote peer can be put into the "negotiated" state.


### Parsing an Answer ###

When a peer in the offer state receives an answer, the peer must parse the answer. The first step is to parse out the SDP information as per the expectations of how it was generated. Since the offer was constructed with JavaScript and not by the browser, the exact parameters and fields will be exactly how the JavaScript decided with no surprises (outside of fixable JavaScript bugs). The peer has a connection object already. The SDP is extracted for its ICE credentials, where "setDescription" is called for the connection with the relevant information from the answer about the ICE userFrag, password and the ICE candidates can be set using "setRemoteConnectionCandidate". The codecs are pulled from the SDP and "setCapabilities" is called to indicate the "receive" capabilities. If SSRC mapping was supported, the SSRC maps can be pulled out of the SDP and set using "setDescription" with a "receive" stream object provided.

The peer can be put into the "negotiated" state.


### Use of Provisional Answers ###

In the object model, any connection can be forked ({{interactionforking}}) at any time by reusing the same socket object. If a provisional answer is needed as another answer has arrived from an offer, the developer only need only create a new connection object. The developer will then have a connection which can be independently managed with a unique set of ICE candidates, capabilities, and streams. Each connection is its own object and thus allows for independent negotiation with each forked offer which receives an answer.


### Rollback ###

As all the descriptions and capabilities for each object can be maintained easily in an offer / answer state machine, the state machine can obtain all this information prior to constructing a new offer with new information. If the offer is rejected, the state can be rolled back to the previous state immediately, or, alternatively a new state can be applied only when an offer has been accepted. The browser does not need to know the state machine or the previous states, as all of this information can easily be set from JavaScript.


### Configurable SDP Parameters ###

Many SIP providers have custom SDP options and extensions. By allowing the SDP to be generated by JavaScript, the SDP extensions can be placed inside the SDP, without petitioning the browser vendor to make a change.


### SDP Offer / Answer State Machine in JS ###

There is nothing special about an offer / answer state machine. The state machine can just as easily be defined in JavaScript as it can by the browser's internal language. The difference is that fixes in the state machine with JavaScript can be applied dynamically whereas fixes to mistakes in the browser's state machine are impossible until a browser update is released. At best a mistake in the browser can be "worked around". JavaScript can be made to follow the same rules, requirements and expectations as native code, except it becomes optional for JavaScript developers, rather than mandating the state machine be followed for all signaling protocols (even those that do not want offer / answer).


One-side Capabilities
---------------------

Another model for negotiation is "one-sided" capabilities negotiation. In this model, each peer derives a set of "receive" capabilities, which are set to the remote peer. Each peer is allowed to update their respective "receive" capabilities at any time, which must be adhered by the remote party. This is useful to be able to send streams at will without requiring constant back and forth "offer", "answer' and "negotiated" states.

To perform this model, each side obtains its "receive" capabilities using "getCapabilities" on the "receive" direction. The capabilities information is then mutually exchanged as needed. The ICE candidate information is only exchanged upon start and the session lasts for the lifetime of the connection. Imposing a state machine like offer / answer hampers the simplicity of this signaling model. Whereas, SDP requires that the offer and answer contain the "send" capabilities and not the "receive" capabilities. To make this work with offer / answer and SDP requires a fair amount of SDP jerry-rigging (see {{WebRTCJSObjExamples}} for an example).

Example pseudo-code of how capabilities could be re-signaled:

    // alice changes capabilities locally
    var myCapabilities = internalChangeCapabilities();
    alice_session.setCapabilities(myCapabilities, "receive");
    
    // alice signals changed capabilities to bob
    var sigCapabilities = JSON.stringify(
      alice_session.getCapabilities("receive"));
    mysignal(sigCapabilities);
    
    // bob changes sending capabilities
    bob_session.setCapabilities(JSON.parse(sigCapabilities), "send");


XMPP
----

XMPP using Jingle {{Jingle}} as a signaling mechanism. XMPP requires no SDP at all and uses XML as the mechanism for expressing capabilities. By allowing XMPP to directly obtain structured data in JavaScript as well as direct control over the objects without imposing signaling mandates, XMPP implementations are free of the burden imposed with parsing and generating an unfamiliar SDP format, or adhering to the SIP definition of offer / answer, rather than adhering to the Jingle state machine.


Interactions With Forking {#interactionforking}
-------------------------

Some signaling scenarios allow forking. With forking, an offer for a connection is made, but multiple peers might answer to the request to connect. For example, Alice might call Bob but Bob has two devices so both of Bob's devices might answer. The answers can arrive at any time after the offer to connect request has been made. For example, SIP {{RFC3261}} defines "Parallel Search", as well as "Sequential Search". The browser must support forking by allowing a connection object to become forked at any time. This allows each connection to share common ICE userFrag and password credentials with the capabilities defaulted identically, but each fork is allowed to negotiate its own ICE candidates, capabilities, connection and stream descriptions. By doing so, the JavaScript developer can fully support signaling protocols where the forking is allowed, such as SIP.


### Sequential Forking ###        {#sequentialforking}

As the offer / answer state machine is not imposed on a WebRTC JavaScript object model, there is no need to provide provisional answers in the state machine. A provisional answer is an answer that is temporarily accepted until the final answer is determined. Each answer has it's own fork, and the various forks can be handled independently. Once the final sequential answer arrives, all the other connections can be disconnected.


### Parallel Forking ###       {#parallelforking}

As the offer / answer state machine is not imposed on a WebRTC JavaScript object model, and forking is supported, parallel forking is also supported. Each answer has it's own fork, and the various forks can be managed independently. There is no need to disconnect any fork, and each connections' capabilities, descriptions, streams and lifetime is entirely in the control of the JavaScript application.


Session Rehydration
-------------------

The JSEP draft {{I-D.ietf-rtcweb-jsep}} described session rehydration as follows:
"In the event that the local application state is reinitialized, either due to a user reload of the page, or a decision within the application to reload itself (perhaps to update to a new version), it is possible to keep an existing session alive, via a process called "rehydration".  The explicit goal of rehydration is to carry out this session resumption with no interaction with the remote side other than normal call signaling messages."

In order for rehydration to work the current state of the connections must persist somewhere outside of the browser page's context. To support reydration, a developer's JavaScript must be capable of re-creating the stream and connection objects as per the previous persisted state, including but not limited to, all candidate negotiations, all connection establishments, all active streams and the relationships between streams, connections and sockets.

Further, the browser must maintain a reasonable lifetime for which JavaScript is allowed to reconnect to these objects with appropriate security precautions to ensure the objects cannot be hijacked by an unauthorized page containing malicious JavaScript.

The exact mechanism and API for these near-death object resurrections falls within the scope of the W3C and does not impact the overall concepts and design described in this draft, as long as the browser allows the RTC objects to be resurrected exactly in their previous states prior to a page reload, and JavaScript security tenants are adhered.


Comparison to Other Approaches
==============================

JSEP
----

The main differences in the WebRTC Object vs JSEP:

1. JSEP requires an all-encompassing complete description of media, transports, capabilities and streams.
2. Enforces an offer / answer state machine upon all signaling.
3. Enforces SDP offer / answer negotiation.
4. Requires other signaling protocols to transcode from an unfamiliar SDP format.
5. Offers little granular control over the contents of the SDP.
6. Allows simple WebRTC scenarios to be easily implemented.
7. Has a free format where behavior expectations for future additions are unknown, or what aspects of the SDP are allowed to be manipulated or not.
8. Allows SIP implementations to interface with the browser.

Where as WebRTC JavaScript Object Model:

1. Modularizes the information needed into independent components.
2. Enforces no signaling mandates, other than candidate connectivity checking.
3. Treats capabilities as independent and updateable as signaling protocols might require.
4. Allows JavaScript developers to utilize well known JavaScript structures to obtain information.
5. Exposes all the content in a granular fashion.
6. Allows simple WebRTC scenarios to be easily implemented.
7. Has strict rules as to what information is allowed for each object, and methods / events have testable compliance behaviors that can be established.
8. Allows SIP implementations to interface with the browser with more predictable behaviors.


Overall, the WebRTC JavaScript Object Model is more flexible but maintains the same ease of use for basic use cases.


PlanA or PlanB
--------------

Plan A {{I-D.roach-rtcweb-plan-a}}, Plan B {{I-D.uberti-rtcweb-plan}}, and NoPlan {{I-D.ivov-rtcweb-noplan}} are all methods for dealing with large number of media streams and how they must be expressly described in SDP, or negotiated along side SDP. Each have their own merits and pitfalls. However, since this is SDP / SIP specific the RTCWEB WG does not need to be involved in these discussions. All three of these models can be expressed in an object model, and should any additional description information be needed, the object model can be extended to accommodate accordingly.


NoPlan
------

NoPlan is a hybrid approach where some information is carried over SDP, the SDP offer / answer state machine still applies but many of the other description information exists outside the SDP. While a step in the right direction, this creates a lot of ambiguity for responsibilities as well as what should be adhered and what should not. More importantly, the ambiguity exists for extensions. Are they part of SDP, or not? Had NoPlan been taken to the full extent to replace SDP and offer / answer, this solution would be best. In many ways, this is what the object model proposed in this draft is doing.



Requirements Compliance Statement
=================================

This draft is compliant to the following relevant requirements specified in Web Real-Time Communication Use-cases and Requirements {{I-D.ietf-rtcweb-use-cases-and-requirements}}:

  * F2, F3, F4, F10, F11, F12, F16, F19, F20, F22, F25, F26, F27, F29, F31, F32, F33, F35, F36, F37, F38, F39

Whereas the following requirements are not applicable to this draft (either exist outside the object model or are an internal feature of an internal RTC engine):

  * F1, F5, F7, F8, F9, F13, F14, F15, F17, F18, F21, F23, F24, F28, F30, F34


Appropriate design consideration was given to the relevant sections in the "Web Real-Time Communication (WebRTC) - Media Transport and Use of RTP {{I-D.ietf-rtcweb-rtp-usage}}" draft.


Security Considerations
=======================

As stated best in JSEP draft {{I-D.ietf-rtcweb-jsep}} in the security considerations section "The intent of the WebRTC protocol suite is to provide an environment that is securable by default: all media is encrypted, keys are exchanged in a secure fashion, and the Javascript API includes functions that can be used to verify the identity of communication partners."


DTLS
----

### DTLS Fingerprint Responsibility ### {#fingerprintresponsibility}

When DTLS {{RFC6347}} is used for a connection ({{objconnection}}), the DTLS fingerprint(s) must be signaled by the developer in such a way that the hash cannot be modified during transit between connecting peers during a signaling exchange, and / or alternatively, the ownership of the fingerprint(s) must be cryptographically proven to correlate to the identity of the connected peer. Failure to perform a secure exchange of fingerprint(s) or a correlation of the fingerprint(s) to the remote peer's identity may result in a compromised DTLS channel. The responsibility for ensuring a non-compromised DTLS channel fingerprint must be left to the JavaScript developer and signaling path to handle.

The browser must expose the fingerprints involved from the Connection {{objconnection}} object and from each socket that uses DTLS {{objsocket}}.


### IdP ### {#idp}

The identity provider model (IdP) should not be mandated. This allows the IdP draft to become perfected until it is ready for usage and all scenarios and usages of IdP have been thoroughly thought through. Regardless, IdP should not be mandated. Not all WebRTC servers have an identity model for their users.

The IdP model is far from completion at the time of this draft. The current model requires round trips to perform validation before a stream can be rendered, which is not desirable. In simple use cases, the IdP model is not required as a WebSocket with TLS can be used to exchange fingerprints for a private service, which provides the same level of trust from a provider as IdP since the fingerprints are exchanged in a private and secure manner via a trusted provider. The complexity of any identity model must be factored, especially in consideration basic and simple WebRTC use cases.

Should IdP becomes perfected, the connection description {{objconnectiondescription}} can be extended to include the identity information as part of connection negotiation.


### DTLS / SRTP vs SDES / SRTP ### {#dtlssrtp}

Auto-keying for SRTP must be done through a DTLS handshakes extensions as opposed to requiring SDES {{RFC4568}}. DTLS must have a foolproof man-in-the-middle attack prevention scheme (i.e. fingerprint validation, see {{idp}}). Without the fingerprint validation, little security is provided should the stream be intercepted.

If substantial push-back exists over mandating the usage of DTLS for keying over SDES, then the keying for SDES can be easily be included as part of the capabilities structure {{objcapabilities}}. The final decision to mandate, recommend or allow for media key negotiation over DTLS must be made by the RTCWEB WG before any RTC JavaScript interface can be completed.


IANA Considerations
===================

This document requires no actions from IANA.


--- back

