---
title: WebRTC JavaScript Object RTC
# abbrev: WebRTCJSObj
docname: draft-raymond-rtcweb-webrtc-js-obj-rtc-00
date: 2013-07-05
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

For example, some of the complex information required to exchange for RTP is the media stream track to SSRC associations. This information must be signaled to correctly reassemble RTP media into proper streams on the remote side. However, the only reason this mapping needs to be in signaling is because of the limitations of current RTC engines / protocols, which do not understand the concept of streams, or other features like FEC. Should a better RTP engine / protocol be defined in the future, it will be possible to replace the RTP component with "next gen RTP". Such an object that does understand things like streams and FEC and the component and information would not necessarily need mappings and might become a simplified alternative to the RTP model.

This WebRTC JavaScript object model still allows all the key information required for various signaling models, and the information can be exchanged easily as the information is modular, limited, and accessible. For the average JavaScript developer, the objects model is simplistic but still allows those developers who require greater control and flexible access to the constructs they require from an RTC engine inside the browser without needing to perform extensive parsing, mangling and regeneration of a master know-everything session format (such as SDP).

This model further reduces the requirements imposed on traditional / legacy devices by not requiring these devices to understand a vastly more complex session description format or by requiring an intermediary to interpret this complex master session description and state machine into something more palatable for the device.

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

By not imposing a master transport and media description format, a JavaScript developer will not be exposed to a complex unfamiliar session description format with little high level control over what's happening at protocol layers. The current approach requires developers to perform extensive parsing, interpret or transform this master session description when more control is required. Decoupling the master know-everything session format and the state machine required to exchange it into logical well-defined components with non-specified requirements on how the information must be exchanged allows for greater understanding of the information and flexibility in signaling. The net result should be less brittle implementations that are not dependent upon an ever evolving, adaptable and expanding master know-everything format.

Contrary to the assertions stated in JSEP {{I-D.ietf-rtcweb-jsep}} "1.2 Other Approaches Considered", the WebRTC JavaScript object model proposed does not result in overly complex JavaScript code to perform simple and basic WebRTC implementations. The simple scenarios for RTC control from JavaScript are no more complex than the simple JSEP examples and do not require dependency on additional libraries to create a reasonable reference implementation. Whereas the complexity required to implement other more advanced scenarios that do not follow the specific offer / answer state machine model is greatly reduced relative to JSEP's requirements. The manual manipulation and transformation of a complex session description format generated by the browser is all but eliminated. Only those who must implement a particular type of session format will be required to parse and generate that format without imposing that format upon others.

The WebRTC JS Object Model does not implement or add any features other than those already existing in WebRTC 1.0, but because of it's granular object based approach, it allows for a variety of alternative signaling scenarios that are difficult to achieve with the current WebRTC API {{WebRTC10}}. In fact, the requirements for a browser are eased by virtue of not having to handle an all-encompassing media description format with an offer / state machine. Both models need to perform all the same logic routines internally, including all the RTP data muxing rules, but the WebRTC object model removes many of the unneeded restrictions and burdens placed on a JavaScript developer on how a signaling protocol must work, and decouples the data needed to transform the object information into alternative formats.


Terminology
===========

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 {{RFC2119}}.



Object, Data and Control Model
==============================


MediaSocket, DataSocket, MediaDataMuxSocket {#objsocket}
-------------------------------------------

Each one of these types of transports is capable of carrying the appropriate type of streams implied by their name. The MediaSocket and MediaDataMuxSocket can carry media streams whereas the DataSocket and MediaDataMuxSocket can be used to carry data streams.

Under most usual conditions, a developer would want a "multipurpose" MediaDataMuxSocket, which is why a connection {{objconnection}} object by default creates this type of socket. However, there are rarer conditions where a developer may wish to split out the ports rather than mux them, thus they can create individualized sockets only when needed and provide them to the connection object. For example, a connected device may require the data and media sockets separate because it cannot support demuxing. Or the developer desires to split the audio and video to prevent the audio port buffering from being overflowed from video buffering.

The developer must be able to provide each of these sockets a list of stun and turn servers. This allows the socket to discover other network-reflective network mappings and open up relay network sockets to allow the socket to be "remoted" when direct connections to the socket are not possible.

Should the socket supports DTLS {{RFC6347}} (or DTLS/SCTP {{RFC4960}}), the developer's JavaScript must be able to be provided an alternative private / public certificate pair, rather than utilizing the browser's pre-generated certificate(s) (for further discussion on this issue see the Security Section ({{jscertificates}})). The media sockets may be provided a restrictive list of media "kinds" they are allowed to transport, for example "audio", and / or "video". This allows the media socket to restrict the kinds of media stream tracks allowed to be transported on a particular socket. For example, a developer might wish audio to send over one socket and video to be sent over a different socket.

The JavaScript developer must be able to control the mapping of what kind of streams or tracks go over which socket(s).


Connection        {#objconnection}
----------

A connection is an object that represents the logical connection between two Internet peers. Each peer constructs a connection object that gathers candidates (i.e. possible methods to connect over the Internet) and allows an external signaling mechanism to exchange these candidates between the peers. The peers test the candidates for connectivity and establish a connection once a candidate pairing discovers connectivity (see {{I-D.rescorla-mmusic-ice-trickle}}).

Once a connection is established on all sockets, an event must be fired to the developer's JavaScript / application to indicate connectivity. The connection remains established until disconnected by programmatic intent or by timeout failure, upon which time a disconnect event must be fired to indicate the disconnection state with the appropriate listed reason. Should the connection fail to connect, a disconnection event must fire with the appropriate listed reason.

Once a connection is established between peers, a connection can be used to transmit media or data utilizing the associated media or data sockets as appropriate. Appropriate constraints ({{objconstraints}}) for each connection must be exchanged before media streams may flow between peers.

A connection must provide by default a multipurpose socket ({{objsocket}}) (e.g. MediaDataMuxSocket) capable of carrying a variety of stream types if no socket is specified.


### Relationship to Socket ###

Each connection {#objconnection} object must have one or more sockets ({#objsocket}), and multiple connections can share the same socket objects. By default, a connection creates a multipurpose socket (e.g. MediaDataMuxSocket) if none is specified. In a forked condition ({{interactionforking}}), the connections will share the same socket objects. A socket represents a single logical connection binding on a network interface, thus a unique set of negotiations occurs for each ICE ({{objice}}) session on each socket despite the negotiation being performed on the same connection (which is demuxed by way of the "socket id" on the candidate description ({{objcandidatedescription}})).


### Connection Description ### {#objconnectiondescription}

While a connection might have a variety of other properties, every connection object has a description structure that must have the following properties:

  * cname - an optional cname override to use for all RTP streams within the connection (if none is provided a default will be selected).
  * contextId - a cryptographic random identifier for the connection auto-generated by the browser, which has a dual purpose of serving as the ICE userFrag.
  * secret - a cryptographic random string that can be used within the context of the connection as needed, which has a dual purpose of serving as the ICE password. If the secret is used in any other signaling keying, it must be appropriately salted using standard cryptographic salting principles.
  * fingerprints - an ordered list of all the hash fingerprints of any public keys used in the sockets (see {{fingerprintresponsibility}}).

A context identifier is used not only for the ICE username, it must be a common id that remains stable across forked connections. This allows the developer to correlate the forked connections as appropriate by a JavaScript developer.

The information contained in a connection's description must be exchanged with a remote peer's connection object before any candidate connectivity checks may be performed by the browser. The exact method and timing of the exchange must be left to the discretion of the JavaScript developer.


#### Connection Description Negotiation ####

The local peer has a connection description. The remote peer has a connection description. Each peer must exchange the information in the connection description with the other through signaling. The contextId becomes the useFrag, and the secret becomes the password for ICE signaling (see {{RFC5245}}). The CNAME is used as part of the RTCP reporting (see {{RFC3550}}) so that each peer can establish stream to peer correlation.

Each connection has an array of fingerprints for each socket added to the connection that supports DTLS {{RFC6347}}. These fingerprints must be signaled in order to preserve the fingerprint to DTLS mapping of each socket. The connection object is responsible for ensuring any remotely connected peer's fingerprint during DTLS exchange matches the fingerprint provided in the connection description of the remote peer. If the fingerprint does not match, the connection must be closed.

Example pseudo-code of how connection negotiated could be accomplished:

    // alice makes offer
    var alice_connection = new ORTC.Connection();
    var aliceDesc = alice_connection.getDescription();
    
    mysignal("to:bob", JSON.stringify(aliceDesc));
    
    var bob_connection = new ORTC.Connection();
    bob_connection.setDescription(JSON.parse(aliceDesc), "remote");


### ICE ###      {#objice}

ICE {{RFC5245}} is a standard used for discovering peer connectivity. A browser must support both ICE, and the ICE modification extension trickle ICE {{I-D.rescorla-mmusic-ice-trickle}} connectivity discovery mechanisms. The browser may support additional peer connectivity methods other than ICE.

When the browser detects a candidate is available, the candidate must be sent as an event to the developer's JavaScript. If all possible candidates have been discovered, the browser must send a discovery completion event to the developer's JavaScript.

Should the browser fire non-ICE candidate events to the developer's JavaScript, the developer's JavaScript should be able to deliver the candidate information to the remote peer 'as is' if the developer's signaling protocol allows for alternative non-ICE candidates. The browser must define a clear method to distinguish standard ICE discovered candidates from non-ICE extension candidates for signaling protocols unable to handle non-ICE connection candidates.


#### ICE Candidate Trickling ####

The browser must support trickle ICE {{I-D.rescorla-mmusic-ice-trickle}} discovery mechanism. This mechanism allows the browser to provide candidate information as they are discovered, thus allowing the developer's JavaScript to exchange this information to a remote peer via signaling. As candidates are exchanged, the browser must perform connectivity checks as defined by trickle-ICE even while other candidates are still being discovered and exchanged via signaling (to allow for faster connection times).

Before any connectivity checks may be performed by a browser, a connection's description information ({{objconnectiondescription}}) must be received from a remote peer's connection.


#### ICE Candidate Description ### {#objcandidatedescription}

Each ICE candidate has description information containing the following:

  * socket id - the socket identifier where the candidate was discovered, matching the identifier specified in the MediaSocket, DataSocekt or MediaDataMuxSocket ({{objsocket}}) provided to the connection ({{objconnection}}) object
  * foundation (as as defined in RFC5245 {{RFC5245}})
  * component id (as as defined in RFC5245)
  * transport (as as defined in RFC5245)
  * priority (as as defined in RFC5245)
  * connection address (as as defined in RFC5245)
  * connection port (as as defined in RFC5245)
  * type (defined as "srflx", "prflx", or "relay" whose meaning is derived from RFC5245)
  * related address (as as defined in RFC5245)
  * related port (as as defined in RFC5245)

Other non-ICE candidate types may contain other types of description information. The browser should implement only well described candidate types, or namespace the candidate type to not become confused with potential standardized candidate types. By using standardized descriptions, the browser will help facilitate translation to alternative signaling protocols. The developer's JavaScript should be able to transport the candidate description information 'as is' even if the description information is not understood so long as the developer's signaling protocol allows for alternative candidate types.

The candidate descriptions may be relayed to the remote peer independent of other description information or bundled together with other descriptions as determined by the needs of the JavaScript developer. The browser must not impose restrictions on external signaling protocols related to the exchange of candidates, or require the bundling of other types of descriptions, such as constraint descriptions ({{objconstraints}}) or stream descriptions ({{objstreamdescriptions}}).


### Constraints ### {#objconstraints}

Constraints consist of a list of media codecs and cryptographic encryption algorithms allowed to be used as part of the transmission of media. Constraints can consist of additional rules or limitations, for example; video size or bandwidth limitations. A developer's JavaScript must be able to obtain the types of constraints supported by the browser ({{objfeaturediscovery}}), and the default set of codecs and algorithms available.

Send constraints indicate media constraints that apply to media flowing from the local to the remote peer. Receive constraints apply to the media flowing to the local peer from a remote peer. The constraints may be identical in both directions, or the constraints may be a negotiated subset, or the constraints may be unique per direction (i.e. send versus receive). A developer's JavaScript must must be able to set the constraints on either direction, at will, and obtain the possible constraints for a connection.


#### Constraints Negotiation ####

The media constraints must be exchanged via some external signaling method by the developer's JavaScript. The local send constraints must be (at minimal) a subset of the received constraints of the remote peer, and the received constraints must be (at minimal) a subset of the send constraints of the remote peer. Should the browser be unable to fulfill the provided constraints for any reason (for example, none of the codecs were compatible), an error most be propagated to the developer's JavaScript, which should handle this condition rather than silently fail. The RTCWEB Working Group must define a mandatory common set of codecs and encryption algorithms to help alleviate the possibility of codec and algorithm failure conditions.

The browser must not dictate the timing or format of how constraints are exchanged, nor the rules of how each peer must provide the information, nor how the information is signaled.

The browser should only support codecs and algorithms that are well documented standards. The browser must namespace non-standard codecs and encryption algorithms using a method that will not cause confusion with codecs and algorithms that might become standardized in the future. The properties for each standardized codec and encryption algorithm must be defined clearly. By defining each codec and encryption algorithms' properties, alternative signaling protocols may translate the standardized codecs and algorithms into their respective signaling protocols. If a browser does not have specific knowledge of a codec, or an encryption algorithm supplied by a developer's JavaScript, it must ignore the non-understood codecs, and / or algorithms. A JavaScript developer may encode and signal non-standard codecs and algorithms so long as none of the browser provided properties (for the codecs and encryption algorithms) are lost during signaling exchange.

Other non-codec and non-encryption constraints may be set for a connection. For example; a developer may wish to set the maximum video dimensions for a connection. The browser should only implement standardized constraints and namespace non-standard constraints to ensure that no conflict arises should a constraint become standardized.

These "other" types of constraints are split into two categories: required (i.e. mandatory) or optional. The browser should not generate "other" mandatory constraints by default but must allow the developer's JavaScript to dictate mandatory constraints. If the mandatory constraint cannot be fulfilled or is not understood by the browser, the browser must propagate an error to the developer's JavaScript. A developer's JavaScript should properly handle a failure condition and must act appropriately (i.e. fail accordingly) when receiving a mandatory constraint that it is unable to fulfill, correctly signal (if applicable), or correctly understand. Mandatory constraints should be used sparingly.

Optional "other" constraints must be considered optional and any constraint property of this nature may be ignored. However, if the browser or the developer's JavaScript understands the constraint property, the property must be respected.

A developer's JavaScript may choose what constraints to signal to a remote peer, or not. The developer must only set constraints that are either understood, or were generated by the browser and signaled 'as is' to the remote party. A developer should attempt to signal all constraints generated by the browser to the remote party, but may choose to filter only the understood constraints.

The browser should only implement standardized constraints and must namespace non-standardized constraints appropriately to avoid conflict with constraints that might be standardized in the future.

Example pseudo-code of how constraints could be negotiated:

    // alice signals constraints to bob
    var aliceConstraints = JSON.stringify(
      alice_connection.getConstraints("receive"));
    mysignal("to:bob", aliceConstraints);
    
    // bob changes sending constraints
    bob_connection.setConstraints(JSON.parse(aliceConstraints), "send");
    
    // bob signals constraints to alice
    var bobConstraints = JSON.stringify(
      bob_connection.getConstraints("receive"));
    mysignal("to:alice", bobConstraints);
    
    // alice changes sending constraints
    alice_connection.setConstraints(JSON.parse(bobConstraints), "send");


### Media Stream (and Media Stream Track) Descriptions ### {#objstreamdescriptions}

RTP is the current standard for performing real-time media transmission. While RTP is effective at media transmission, the RTP transport lacks sufficient details to convert a WebRTC stream object into an RTP transmission and back into an WebRTC stream on the remote peer (where stream objects consist potentially of multiple audio, video (or other) media tracks).

Further, the exact mapping of which sockets and codecs are used is unspecified. RTP does have a SSRC (Synchronization source identifier) for every media stream sent over it which can be used to coordinate the RTP streams. A signaling layer must describe this coordination to be able to synchronize the streams from one peer to another.

Other features, like FEC (Forward Error Correction), are not described in WebRTC stream objects because they are a feature of a transport. Each stream could allow a media track to be sent redundantly on the wire so if part of the stream was damaged during the transmission it could be reconstructed from the redundant stream. Another case is simulcasting where the same media track is encoded multiple ways, for example video layering effects or various encoding sizes (thumbnails vs full streams).

RTP lacks sufficient information to be able to describe FEC or simulcasting as part of its transmission, thus a higher layer must signal this information to a remote peer if these features are to be supported.

The browser must provide a method to create a stream description for the features it supports, to allow for stream and track synchronization (and correlation), and other transport features like FEC or simulcast. The exact mapping structures for the stream must be clearly defined in exacting detail, describing the expected behaviors and use cases a browser must support.

For each track in a stream the browser must supply the following information:

  * track-id - the identifier of the track (i.e. from MediaStreamTrack.id {{MediaCapture}})
  * socket id - the identifier correlating to the media socket ({{objsocket}})
  * ssrc - the SSRC of the track as defined by RFC3550 {{RFC3550}}
  * kind - the kind of media being transported (meaning derived from MediaStreamTrack {{MediaCapture}})
  * constraints - (optional, the default send / receive connection ({{objconnection}}) constraints apply if not specified)

If FEC is supported by the browser and the developer's JavaScript has enabled the feature, the browser must supply the following additional information on FEC tracks:

  * the redundancy SSRC - the SSRC that is redundant to the current track

(NOTE: FEC is a complex subject with alternative methods to perform the same levels of redundancy. This draft takes no position on the best techniques and practices and only suggests one method for describing FEC within the steam description. As other description formats are able to support various FEC scenarios, the descriptive information for various FEC scenarios could easily be added into this draft depending on what the IETF/W3C wish to support by default.)

The browser must support a developer's JavaScript specifying constraints per track on both the send and receive ends of the track. As each SSRC can be demuxed from other SSRCs at the RTP layer, the constraints can be applied per track.

While these stream (and track) descriptions are intended to be sent to remote peers, the browser must not mandate the timing, format, or the signaling of these stream descriptions.


Example pseudo-code of how streams could be sent and signaled:

    // alice sends information about stream to bob
    var aliceStreamDesc = JSON.stringify(
      alice_connection.getDescription(aliceMediaStreamToBob));
    
    alice_connection.sendStream(aliceMediaStreamToBob);
    
    mysignal("to:bob", JSON.stringify(aliceStreamDesc));
    
    // bob receives stream
    var bobMediaStreamFromAlice = new MediaStream();
    bob_connection.setDescription(
      bobMediaStreamFromAlice,
      JSON.parse(aliceStreamDesc)
      );
    bob_connection.receiveStream(bobMediaStreamFromAlice);



#### Future Media Stream Descriptions #### {#objfuturestreamdescriptions}

The complexity in describing streams and tracks is mostly born because RTP lacks sufficient definition to reassemble RTP media tracks into streams on a remote peer. In the future, a new "next generation" RTP may become standard. In such a case, the "next generation" RTP layer may support the concept of tracks, streams, or things like FEC, and simulcasting.

If a "next generation" RTP transport becomes defined, a stream description might be different or not required at all, or perhaps with other properties yet to be defined. Each new supported transport type must define exactly the stream definition expected to be produced by the browser, if any at all.


### Sending Media Streams (over RTP) ###

The browser must be able to support sending streams via a connection {{objconnection}} over the socket(s) {{objsocket}}. The developer's JavaScript must be able to indicate to the browser the stream and track mappings (in the form of a stream description) if the transport requires this information (assuming it is required, see {{objfuturestreamdescriptions}}).

The browser must respect the media stream track mapping to the socket {{objsocket}}, SSRC, and optional constraints. The "kind" property for sending is merely information but is not explicitly required, but if set, must match the media stream track's "kind". The developer's JavaScript must be able to specify that individual media stream track may be omitted from sending, while other media stream tracks are sent.

The developer's JavaScript must be able to control the exact timing when a stream starts to send and when a stream stops sending. At any time, the developer's JavaScript must be able to change the track mapping and constraints without the browser imposing any signaling requirements to a remote peer, thus allowing for various alternative signaling scenarios.

The browser must only utilize the codecs within the constraints provided for sending, and alternate codecs at will. A browser should decide which codec is best suited for the network conditions amongst all the codecs within the send constraints. If a developer's JavaScript wishes to limit to a particular codec, the send constraints should only contain the codec desired.

If FEC is supported, the browser must coordinate the sending of the FEC RTP redundancy tracks.


### Receiving Streams (over RTP) ###

The browser must be able to support receiving streams via a connection {{objconnection}} over the socket(s) {{objsocket}}. The developer's JavaScript must be able to specify the streams expectations for any streams about to be received (or currently being received).

The developer's JavaScript is not required to signal the stream mapping should the developer not care about stream and stream track reassembly, or if the developer will manually assemble the streams tracks into streams based on their own logic. If media tracks arrive without any stream definition, the browser must render the RTP streams as media stream tracks unattached to a particular stream. Should the developer's JavaScript eventually specify a stream definition, the browser must assemble the individual tracks into streams according to the stream definitions. The developer's JavaScript must be able to assemble individual stream tracks into streams manually should the developer not wish to provide a stream definition.

The browser must be able to render the same media stream track as part of multiple media stream objects, to allow a stream definition to contain the same track in multiple streams.

Each media stream track is demuxed first by the socket ({{objsocket}}), then by the SSRC, then by "kind" based upon the codec "kind" of the payload. While technically possible to demux upon the same "kind", the browser must only support demuxing the same media "kind" with the same SSRC if the payload id used only matches one set of constraints for a particular track on the receiving stream. A developer's JavaScript should not mux the same "kind" of media on the same SSRC. A developer's JavaScript should not mux different "kinds" of media on the same SSRC.

A developer's JavaScript must provide a stream definition to receive any streams that utilize the FEC feature when transmitting over RTP.


#### Loosely Matching Media Stream Descriptions #### {#loosedescriptions}

The browser must support loose definitions for streams where a signaling was not used to send stream descriptions between peers. This allows the browser to support simpler signaling protocols which are either incapable or unwilling to signal stream descriptions.

The browser should be able to receive a stream without specifying any details. In this case, the browser will attach all incoming media stream tracks of different media kinds into the stream specified. In such a case, the browser will apply loose rules to automatically determine which track belongs to which stream, factoring in such things as the sockets, kinds of media on each socket, and the constraints provided (and perhaps timing). This allows for very simple signaling use cases where the developer doesn't need to signal stream details needlessly for the most common use cases. Alternatively, the developer's JavaScript should be able to specify partial or complete details such as socket id, to indicate a particular socket, or a "kind" to filter on particular kinds of media, or even on an SSRC that was generated and presumed based on an algorithm of the developer's choosing. This allows the developer's JavaScript complete control over the level of precision in the stream matching they can expect.

When filters are used for a stream description to define specific track definitions, the browser should match upon the most specific rule if items like the SSRC are specified in the rules. As the browser API defined by the W3C, the W3C must define specific rules to follow when receiving streams that make most sense for their anticipated use cases.

The intention of allowing for loose rules is to allow for a larger variety of signaling scenarios where simple "loose" rules could be used to match the media stream tracks rather than always mandating signaling be exchanged. Loose rules may work well for many and most common use cases, but still allows the developer to signal specifics when they do not suffice.


Example pseudo-code of how streams could be sent with loose matching media stream descriptions:

    // alice sends information about stream to bob
    alice_connection.sendStream(aliceMediaStreamToBob);
    
    mysignal("to:bob", "+stream");
    
    // bob receives stream (auto-loose matching will work in simple case)
    var bobMediaStreamFromAlice = new MediaStream();
    bob_connection.receiveStream(bobMediaStreamFromAlice);


### Data Stream Description ###  {#objdatastreamdescription}

A data stream is an object that is a specialized type of stream (see {{datastream}}), just as a media stream is an object for media related information. The data stream description contains the following properties:

  * stream id - the identifier for the data stream (from MediaStream.id {{MediaCapture}})
  * socket id - the socket to carry the data stream ({{objsocket}})

The browser must not define or specify how or when the signaling for the data stream description is sent.


### Sending Data Streams (over DataSocekt) ###

The browser must be able to send a data stream over the connection as long as a data stream compatible socket has been added, otherwise an error must be propagated to the developer's JavaScript. For the simple case, the developer's JavaScript should be able to send the stream without any additional information being specified to allow for simple signaling scenarios where the data stream mapping is unimportant.


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

The first step required is creating a connection object. The connection object can optionally be constructed with various socket configurations for transport, such as separating audio and video RTP sockets, or muxing the data on the same socket as RTP. The developer can obtain the send constraints and populate the "m=" line(s) for the media desired, as well as all the codec and cryptography information. The ice userFrag and password information is available from the connection object. If the SDP must generate SSRC mappings, the "getDescription" method can be used to obtain the mapping information in order to generate the various SDP SSRC entries accordingly. Finally, the ICE candidates will be notified using an "onconnectioncandidate" event and the offer can either be generated before all candidates arrive, or after once all candidates are ready.

The peer can be put into an "offer" state.


### Parsing an Offer ###

When a remote peer receives an offer, the peer must construct an answer, but it must parse the offer before this. The first step is to parse out the SDP information as per the expectations of how it was generated. Since the offer was constructed with JavaScript and not by the browser, the exact parameters and fields will be exactly how the JavaScript decided with no surprises (outside of fixable JavaScript bugs). The remote peer can construct a connection object in the same manner as the offer peer. The SDP is extracted for its ICE credentials, where "setDescription" is called for the connection with the relevant information from the offer about the ICE userFrag, password and the ICE candidates can be set using "setRemoteConnectionCandidate". The codecs and encryption algorithms are pulled from the SDP and "setConstraints" is called to indicate the "receive" constraints. If SSRC mapping was supported, the SSRC maps can be pulled out of the SDP and set using "setDescription" with a "receive" stream object provided.


### Generating an Answer ###

To construct answer, the ICE credentials are pulled from the connection object using the "getDescription". The candidates will arrive on the "onconnectioncandidate" and can be placed in the SDP. The constraints for the "send" can be obtained by calling "getConstraints" for the "send" direction (and only mutually agreed codecs can be filtered in if desired). The "m=" line can be generated for the specific medias with the related "a=" codec mappings. If the SDP must generate SSRC mappings, the "getDescription" method can be used to obtain the mapping information in order to generate the various SDP SSRC entries in the answer.

The remote peer can be put into the "negotiated" state.


### Parsing an Answer ###

When a peer in the offer state receives an answer, the peer must parse the answer. The first step is to parse out the SDP information as per the expectations of how it was generated. Since the offer was constructed with JavaScript and not by the browser, the exact parameters and fields will be exactly how the JavaScript decided with no surprises (outside of fixable JavaScript bugs). The peer has a connection object already. The SDP is extracted for its ICE credentials, where "setDescription" is called for the connection with the relevant information from the answer about the ICE userFrag, password and the ICE candidates can be set using "setRemoteConnectionCandidate". The codecs and encryption algorithms are pulled from the SDP and "setConstraints" is called to indicate the "receive" constraints. If SSRC mapping was supported, the SSRC maps can be pulled out of the SDP and set using "setDescription" with a "receive" stream object provided.

The peer can be put into the "negotiated" state.


### Use of Provisional Answers ###

In the object model, any connection can be forked ({{interactionforking}}) at any time. If a provisional answer is needed as another answer has arrived from an offer, the developer only needs to call the "fork" method on the connection. The developer will then have a connection which can be independently managed with a unique set of ICE candidates, constraints, encryption algorithms, and streams. Each forked connection is its own object and thus allows for independent negotiation with each forked peer.


### Rollback ###

As all the descriptions and constraints for each object can be maintained easily in an offer / answer state machine, the state machine can obtain all this information prior to constructing a new offer with new information. If the offer is rejected, the state can be rolled back to the previous state immediately, or, alternatively a new state can be applied only when an offer has been accepted. The browser does not need to know the state machine or the previous states, as all of this information can easily be set from JavaScript.


### Configurable SDP Parameters ###

Many SIP providers have custom SDP options and extensions. By allowing the SDP to be generated by JavaScript, the SDP extensions can be placed inside the SDP, without petitioning the browser vendor to make a change.


### SDP Offer / Answer State Machine in JS ###

There is nothing special about an offer / answer state machine. The state machine can just as easily be defined in JavaScript as it can by the browser's internal language. The difference is that fixes in the state machine with JavaScript can be applied dynamically whereas fixes to mistakes in the browser's state machine are impossible until a browser update is released. At best a mistake in the browser can be "worked around". JavaScript can be made to follow the same rules, requirements and expectations as native code, except it becomes optional for JavaScript developers, rather than mandating the state machine be followed for all signaling protocols (even those that do not want offer / answer).


One-side Constraints
--------------------

Another model for negotiation is "one-sided" constraints negotiation. In this model, each peer derives a set of "receive" constraints, which are set to the remote peer. Each peer is allowed to update their respective "receive" constraints at any time, which must be adhered by the remote party. This is useful to be able to send streams at will without requiring constant back and forth "offer", "answer' and "negotiated" states.

To perform this model, each side obtains its "receive" constraints using "getConstraints" on the "receive" direction. The constraints information is then mutually exchanged as needed. The ICE candidate information is only exchanged upon start and the session lasts for the lifetime of the connection. Imposing a state machine like offer / answer hampers the simplicity of this signaling model. Whereas, SDP requires that the offer and answer contain the "send" constraints and not the "receive" constraints. To make this work with offer / answer and SDP requires a fair amount of SDP jerry-rigging (see {{WebRTCJSObjExamples}} for an example).

Example pseudo-code of how constraints could be re-signaled:

    // alice changes constraints locally
    var myConstraints = internalChangeConstraints();
    alice_connection.setConstraints(myConstraints, "receive");
    
    // alice signals changed constraints to bob
    var sigConstraints = JSON.stringify(
      alice_connection.getConstraints("receive"));
    mysignal(sigConstraints);
    
    // bob changes sending constraints
    bob_connection.setConstraints(JSON.parse(sigConstraints), "send");


XMPP
----

XMPP using Jingle {{Jingle}} as a signaling mechanism. XMPP requires no SDP at all and uses XML as the mechanism for expressing constraints. By allowing XMPP to directly obtain structured data in JavaScript as well as direct control over the objects without imposing signaling mandates, XMPP implementations are free of the burden imposed with parsing and generating an unfamiliar SDP format, or adhering to the SIP definition of offer / answer, rather than adhering to the Jingle state machine.


Interactions With Forking {#interactionforking}
-------------------------

Some signaling scenarios allow forking. With forking, an offer for a connection is made, but multiple peers might answer to the request to connect. For example, Alice might call Bob but Bob has two devices so both of Bob's devices might answer. The answers can arrive at any time after the offer to connect request has been made. For example, SIP {{RFC3261}} defines "Parallel Search", as well as "Sequential Search". The browser must support forking by allowing a connection object to become forked at any time. This allows each connection to share common ICE userFrag and password credentials with the constraints defaulted identically, but each fork is allowed to negotiate its own ICE candidates, constraints, connection and stream descriptions. By doing so, the JavaScript developer can fully support signaling protocols where the forking is allowed, such as SIP.


### Sequential Forking ###        {#sequentialforking}

As the offer / answer state machine is not imposed on a WebRTC JavaScript object model, there is no need to provide provisional answers in the state machine. A provisional answer is an answer that is temporarily accepted until the final answer is determined. Each answer has it's own fork, and the various forks can be handled independently. Once the final sequential answer arrives, all the other connections can be disconnected.


### Parallel Forking ###       {#parallelforking}

As the offer / answer state machine is not imposed on a WebRTC JavaScript object model, and forking is supported, parallel forking is also supported. Each answer has it's own fork, and the various forks can be managed independently. There is no need to disconnect any fork, and each connections' constraints, descriptions, streams and lifetime is entirely in the control of the JavaScript application.


### Forking Example Code ###

Example pseudo-code of how forking can be accomplished (most of which would happen in parallel but illustrated sequentially):

    //-------------------------------------------------------------------
    // alice makes offer
    var aliceDesc = alice_connection.getDescription();
    
    mysignal("to:bob", JSON.stringify(aliceDesc));
    
    //-------------------------------------------------------------------
    // bob device-1 replies
    var bob1_connection = new ORTC.Connection();
    bob1_connection.setDescription(JSON.parse(aliceDesc), "remote");
    
    var bob1Desc = bob1_connection.getDescription();
    mysignal("to:alice", JSON.stringify(bob1Desc));
    
    //-------------------------------------------------------------------
    // alice sees bob1 reply
    alice_connection.setDescription(JSON.parse(bob1Desc), "remote");
    
    //-------------------------------------------------------------------
    // tightly couple trickle ICE (usually sent over signaling channel)
    alice_connection.onconnectionandidate = function(candidate) {
        bob1_connection.addConnectionCandidate(
          JSON.parse(JSON.stringify(candidate)));
    });
    bob1_connection.onconnectioncandidate = function(candidate) {
        alice_connection.addConnectionCandidate(
          JSON.parse(JSON.stringify(candidate)));
    });
    
    //-------------------------------------------------------------------
    // bob device-2 replies
    var bob2_connection = new ORTC.Connection();
    bob2_connection.setDescription(JSON.parse(aliceDesc), "remote");
    
    var bob2Desc = bob2_connection.getDescription();
    mysignal("to:alice", JSON.stringify(bob2Desc));
    
    //-------------------------------------------------------------------
    // alice sees bob2 reply and forks the connection object
    var alice2_connection = alice1_connection.fork();
    alice2_connection.setDescription(JSON.parse(bob1Desc), "remote");
    
    //-------------------------------------------------------------------
    // tightly couple trickle ICE (usually sent over signaling channel)
    alice2_connection.onconnectionandidate = function(candidate) {
        bob2_connection.addConnectionCandidate(
          JSON.parse(JSON.stringify(candidate)));
    });
    bob2_connection.onconnectioncandidate = function(candidate) {
        alice2_connection.addConnectionCandidate(
          JSON.parse(JSON.stringify(candidate)));
    });

Session Rehydration
-------------------

The JSEP draft {{I-D.ietf-rtcweb-jsep}} described session rehydration as follows:
"In the event that the local application state is reinitialized, either due to a user reload of the page, or a decision within the application to reload itself (perhaps to update to a new version), it is possible to keep an existing session alive, via a process called "rehydration".  The explicit goal of rehydration is to carry out this session resumption with no interaction with the remote side other than normal call signaling messages."

In order for rehydration to work the current state of the connections must persist somewhere outside of the browser page's context. To support reydration, a developer's JavaScript must be capable of re-creating the stream and connection objects as per the previous persisted state, including but not limited to, all candidate negotiations, all connection establishments, all active streams and the relationships between streams, connections and sockets.

Further, the browser must maintain a reasonable lifetime for which JavaScript is allowed to reconnect to these objects with appropriate security precautions to ensure the objects cannot be hijacked by an unauthorized page containing malicious JavaScript.

The exact mechanism and API for these near-death object resurrections falls within the scope of the W3C and does not impact the overall concepts and design described in this draft, as long as the browser allows the RTC objects to be resurrected exactly in their previous states prior to a page reload, and JavaScript security tenants are adhered.


Interface
=========

This section details the basic operations that must be present to implement the RTC JavaScript Object Model functionality.  The actual API exposed in the W3C API may have somewhat different syntax, but should map easily to these concepts.


Object RTC Global Methods
-------------------------

### getFeatures ###

The browser must supply a method to obtain well defined feature sets that are supported by the RTC engine (see {{objfeaturediscovery}}).


### getConstraints ###

The browser must supply a method to obtain the default system constraints (see {{objconstraints}}).


MediaSocket {#mediasocket}
-----------

The media socket object is responsible for sending RTP packets over the wire and providing raw connectivity to the network, including setting up connections with related stun and turn servers.

The media socket can be constructed with a few options specified by the developer's JavaScript:

  * flag to mux RTP and RTCP on a single port (default), or to use two independent ports, one for RTP and the other for RTCP (ports should be sequential, with the first port being even)
  * list of optional stun servers to perform reflexive IP discovery (default, browser specific)
  * list of optional turn servers to act as relays (default, browser specific)
  * restrictive list of media "kinds" (see MediaStreamTrack {{MediaCapture}}) to restrict the socket to only carry certain kinds of media (default, no restriction)
  * alternative public / private key pair (default, internal browser generated public / private key)

The browser must not allow access via JavaScript to any public / private keys internally generated by the browser, but must allow a developer's JavaScript to provide an alternative public / private key pair.

The fingerprint property representing the hash of the public key on a socket object supporting DTLS {{RFC6347}} must be retrievable (for further discussion {{dtlssrtp}}).


DataSocket {#datasocket}
----------

The data socket object is responsible for sending data streams over the wire and providing raw connectivity to the network, and this includes setting up connections with related stun and turn servers.

The data socket can be constructed with a few options specified by the developer's JavaScript:

  * list of optional stun servers to perform reflexive IP discovery (default, browser specific)
  * list of optional turn servers to act as relays (default, browser specific)
  * alternative public / private key pair (default, internal browser generated public / private key)

The browser must not allow access via JavaScript to any public / private keys internally generated by the browser, but must allow a developer's JavaScript to provide an alternative public / private key pair.

The fingerprint property representing the hash of the public key on a socket object supporting DTLS {{RFC6347}} must be retrievable.


MediaDataMuxSocket  {#mediadatamuxsocket}
------------------

The media / data mux socket object is a multiplexed socket object allowing either RTP or DTLS/SCTP {{RFC6347}}/{{RFC4960}} information to be carried on the same port. The object carries the same options and properties as both the media socket object ({{mediasocket}}) and the data socket object ({{datasocket}}).


Connection
----------

The connection object represents the interface to a connection between the local browser and a remote peer (or server).

### Construction ###

The connection object allows for the following optional information to be specified on construction:

  * list of sockets (if none provided, default MediaDataMuxSocket ({{mediadatamuxsocket}}) with id "default" is created)
  * activate state (default is "true", i.e. active, "false" therefor would mean that streams are prevented from sending or receiving)
  * send constraints - list of codec, algorithms and "other" constraint restrictions placed upon send streams by default (defaults to system constraints)
  * receive constraints - list of codec, algorithms and "other" constraint restrictions placed upon send streams by default (defaults to system constraints)

The constructor is also responsible for setting up the following information:

  * context id - all connections that are forked from the same root share the same context id (where as each connection still has a unique id)
  * secret - a randomly generated secret for the connection, which is shared by all forked connections
  * cname - optionally passed into as an option (default by the system according Guidelines for Choosing RTP Control Protocol (RTCP) Canonical Names (CNAMEs) ({{I-D.ietf-avtcore-6222bis}}))

The context id and the secret serve a dual role as being used for the ICE {{RFC5245}} userFrag and password during ICE connectivity checks, and may be used for other purposes. The secret must not be compromised by a developer's JavaScript but the value must be available to JavaScript to facilitate alternative signaling protocols.

### onconnectioncandidate ###

The "onconnectioncandidate" event must fire to allow a developer's JavaScript to receive a discovered connection candidate. The developer's JavaScript is responsible for signaling the candidate to a remote peer.

### onconnectioncandidatesdone ###

The "onconnectioncandidatesdone" event must fire to allow a developer's JavaScript to be notified when all candidate discoveries have completed. This is required to allow for signaling protocols that cannot or do not wish to trickle ICE candidates as they are discovered. The developer's JavaScript is responsible for signaling the candidates to a remote peer.

### onactiveconnectioncandidate ###

The "onactiveconnectioncandidate" event must fire to allow a developer's JavaScript to be notified which active candidate local / remote pairing the connection is using. This is required to allow for signaling protocols that may be required to report the active connection candidates.

### onconnected ###

The "onconnected" event must fire to allow a developer's JavaScript to be notified when a connection is connected. This is required to allow signaling protocols to start utilizing the established connection.

### ondisconnected ###

The "ondisconnected" event must fire to allow a developer's JavaScript to be notified when a connection is disconnected. This is required to allow signaling protocols to react to a disconnection in whatever manner is appropriate given the reason why the connection failed as part of the event. If a single socket should fail then the entire connection will disconnect. If that behavior is undesirable, the developer may create multiple connections rather than a single connection so that a failure on one socket does not affect another socket.

### onstreamconnected ###

The "onstreamconnected" event must fire to allow a developer's JavaScript to be notified when a stream has connected. This is required to allow signaling protocols to have knowledge when media or data can flow between peers.

### onstreamdisconnected ###

The "onstreamdisconnected" event must fire to allow a developer's JavaScript to be notified when a stream has disconnected. This is required to allow signaling protocols to have knowledge when media or data flowing between peers has stopped.

### onsteamreport ###

The "onsteamreport" event may fire to allow a developer's JavaScript to be notified of stream's conditions or statistics. The signaling protocol may wish to react based upon stream conditions, for example, to choose a downgraded stream when bandwidth is limited.

### ontrackconnected ###

The "ontrackconnected" event must fire to allow a developer's JavaScript to be notified when a media stream track has connected. This is required to allow signaling protocols to have knowledge when individual media stream tracks have connected.

### ontrackdisconnected ###

The "ontrackdisconnected" event must fire to allow a developer's JavaScript to be notified when a media stream track has disconnected. This is required to allow signaling protocols to have knowledge when individual media stream tracks have disconnected.

### ontrackcontributors ###

The "ontrackcontributors" event must fire to allow a developer's JavaScript to be notified when the contributing sources for a media stream track have changed. This is required to allow signaling protocols to have knowledge of the contributors active in a given media stream, for example, in a conferencing scenario. Since the contributing sources can change per packet, the browser may impose reasonable rules on how frequently this event will fire.

### ontrackreport ###

The "ontrackreport" event may fire to allow a developer's JavaScript to be notified of media stream track's conditions or statistics. The signaling protocol may wish to react based upon media stream track conditions, for example, to choose a downgraded stream when bandwidth is limited.

### activate ###

This method allows the developer's JavaScript to temporarily "pause" all media flowing in and out of the RTC engine. This may be required in some signaling scenarios where the setup process is more involved and the setup requires better synchronization.

### disconnect ###

This method allows the developer's JavaScript to disconnect the connection. This is required by signaling protocols to tear down connections.

### setRemoteConnectionCandidate ###

This method allows the developer's JavaScript to notify the connection about a remote peer's discovered connection candidate as part of the connection setup process.

### getDescription ###

This method allows the developer's JavaScript to obtain description information on connection ({{objconnectiondescription}}), media streams ({{objstreamdescriptions}}), and other streams like data streams ({{objdatastreamdescription}}). These descriptions must be obtainable in order to correctly signal information about connections and streams.

### setDescription ###

This method allows the developer's JavaScript to set the description information on a connection ({{objconnectiondescription}}), media streams ({{objstreamdescriptions}}), and other streams like data streams ({{objdatastreamdescription}}). These descriptions must be changeable in order to allow for a variety of signaling scenarios, or when the description for the remote peer must be set on the local peer in order to correctly establish communications.

### getConstraints ###

This method allows the developer's JavaScript to obtain the current restrictions on "send" or "receive" connections, streams, or individual tracks mapped into an RTP stream. This method is needed for the signaling protocol to obtain the current restriction placed on media streams for codecs, encryption algorithms, or other extension constraints.

### setConstraints ###

This method allows the developer's JavaScript to set the current restrictions on "send" or "receive" connections, streams, or individual tracks mapped into an RTP stream. This method is needed to set the current restrictions placed on media streams for codecs, encryption algorithms, or other extension constraints by signaling protocols.

### sendStream ###

This method allows the developer's JavaScript to send a stream to the remote peer, or remove a stream from sending with "false" specified. This is required to allow the signaling protocol to control when a stream is to be sent to the remote peer, and when a stream is to stop sending to the remote peer.

### receiveStream ###

This method allows the developer's JavaScript to receive a stream from the remote peer, or to stop receiving a stream with "false" specified. The receive stream must allow signaling protocols to use either a loose description ({{loosedescriptions}}), or with a specific stream description obtained from the remote party calling "getDescription(stream)" and signaling this information. By allowing for either loose, or specific descriptions, the browser allows a variety of signaling protocols to be implementable.

### fork ###

This method allows the developer's JavaScript to offer a single connection's description to multiple peers, allowing for multiple connection responses in cases of signaling protocols that allow sequential forking ({{sequentialforking}}) or parallel forking ({{parallelforking}}). Only the first connection ({{objconnection}}) object created must receive all the discovered connection candidates, but each forked connection can be given a unique set of remote connection candidates from each potentially responding peer. The streams attached and the setup for each forked connection must be unique. The constraints are copied from the original forked connection but can set unique per fork. Each forked connection object must be considered a unique object, except that it shares the same originating connection context, and sockets.


### Stream Relationship Management ###

The connection object has relationships to streams, e.g. media ({{MediaCapture}}) or data streams ({{datastream}}). A stream can either be sent, or received on a single connection but should not be both, even if the stream is bi-directional (e.g. a data stream). A data stream ({{datastream}}) only needs one peer to send and the other will receive the "onstreamconnected" event. The recommended method for performing loopback media (where media is sent back where it is received from) is to create a new stream with the old tracks in those rare use cases, as this allows signaling protocol implementation to be absolutely clear as to which direction constraints are being applied when applied at a media stream level.


Example: DTMFMediaStreamTrack Specialization
--------------------------------------------

The DTMF media stream track is an example of a specialized type of media stream track {{MediaCapture}} that can be implemented to facilitate IVR interactions via DTMF {{RFC4733}}. This example is illustrates the flexibility of adding new specialized object types without affecting, or expanding the behaviors (or methods) of other existing objects. Only if a JavaScript developer wishes to use these extension objects will they receive the additional behaviors. The exact features and support requirements for DTMF should be decided by the RTCWEB WG. This merely demonstrates what could be implemented, not what must be implemented.

### Construction ###

The DTMF media stream track should be constructed as a loose object that can be attached to an existing media stream object. By default, the browser must mux the DTMF track with the same SSRC as the first audio track in the stream when sending a stream as per the rules defined in RFC4733 {{RFC4733}}. As DTMF represents a different "kind" of media and has a unique codec attribute, the DTMF track can be easily demuxed into a unique DTMF media stream track by the receiving peer.

### startTone ###

This is an example method allowing the developer's JavaScript to start playing a DTMF tone over the media stream track.

### stopTone ###

This is an example method allowing the developer's JavaScript to stop playing a DTMF tone over the media stream track.

### playTones ###

This is an example method allowing the developer's JavaScript to easily pre-play an array of DTMF tones using equal or specific timings.

### flush ###

This is an example method allowing the developer's JavaScript to easily stop all pre-playing DTMF tones.

### ontonestart ###

This is an example event the developer's JavaScript could receive to be notified that a DTMF tone has started playing on a DTMF media stream track (if support for incoming DTMF events is decided to be optional / mandatory in the browser).

### ontonestop ###

This is an example event the developer's JavaScript could receive to be notified that a DTMF tone has stopped playing on a DTMF media stream track (if support for incoming DTMF events is desired to be optional / mandatory in the browser).


Example: DataStream Specialization
----------------------------------

The data stream is an example of a specialized type of media stream {{MediaCapture}} that can be implemented to facilitate arbitrary data transmission between peers. This example illustrates the flexibility of adding new specialized object types without affecting, or expanding the behaviors (or methods) of other existing objects. Only if a JavaScript developer wishes to use these extension objects will they receive the additional behaviors. Data streams are useful in a variety of signaling scenarios and informational where information can be transmitted directly peer-to-peer, e.g. file transfer.

### Construction ###

In the example, the data stream has an option to specify the reliability of the stream so the stream can be sent in an "envelop" or "fifo" fashion. This is required to allow for alternative signaling methods that wish to send fast updates to a remote peer but may be unreliably sent, or reliably sent in a first-in-first-out fashion. As this is an example only, the actual implementation can be expanded to contain whatever features the RTCWEB WG and W3C ultimately decide upon for data streams.

### onreceive ###

This is an example event the developer's JavaScript could receive to be notified of incoming data.

### send ###

This is an example method the developer's JavaScript could call to send data over the data stream.

### close ###

This is an example method the developer's JavaScript could call to close the data stream.


Comparison to Other Approaches
==============================

JSEP
----

The main differences in the WebRTC Object vs JSEP:

1. JSEP requires an all-encompassing complete description of media, transports, constraints and streams.
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
3. Treats constraints as independent and updateable as signaling protocols might require.
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

The identity provider model has not be mandated. The model is far away from completion, and the round trips to perform validation before a stream can be rendered is not desirable. IdP requires an identity model that might not exist conceptually for many web services. Further, in many simple cases this model is not required as a WebSocket with TLS can be used to exchange fingerprints for a private service providing the same level of trust from the provider since the fingerprints are exchanged in a private manner. IdP also suggest domain federation is possible but federation is only possible if two clients run the same JavaScript signaling model on both ends and signaling is federated across domains. The IdP federation identity model is complex for web services that wish to perform simple WebRTC use cases.


### JavaScript Provided DTLS Certificates ### {#jscertificates}

Controversy exists in allowing JavaScript to provide an alternate certificate to the browser's private certificate. Most of the controversy is over how to display the correct level of security, since allowing JavaScript to provide a certificate may weaken security. There are two forms of this weakness: malicious or accidental.

Preventing weakness due to a provider intentionally using malicious JavaScript is virtually impossible. The browser could have 100% on-the-wire security and still be compromised. The browser is in complete control over where to send streams (so long as ICE permission is granted). In other words, malicious JavaScript code could duplicate user generated streams to whatever destination they choose, which causes on-the-wire security to be essentially meaningless.

If the goal is preventing accidental errors in security, a custom certificate is not the only weak area in code to cause accidental compromises. For example, directing the stream to the wrong location by mistaken negotiation is possible, as are mistakes in server security infrastructure, along with the potential for basic mistakes like JavaScript choosing to contact the wrong party in the first place. Prevention against accidents is a lofty goal, but likely unobtainable without many additional usage restrictions.

The RTCWEB WG must come to a decision to allow this feature or not.


### DTLS / SRTP vs SDES / SRTP ### {#dtlssrtp}

Auto-keying for RTP could be done through a DTLS handshakes extensions as opposed to using SDES. If DTLS becomes the mandated method to perform media key negotiation then the implementation bar is raised for what a remote device must support to communicate, and many legacy devices and networks do not support this feature. DTLS without an absolutely foolproof man-in-the-middle attack prevention scheme (i.e. fingerprint validation, see {{idp}}) provides little security if the stream can be intercepted, and may in fact be weaker than allowing the browsers to exchange SDES credentials privately during their negotiation (e.g. keying via an HTTPS proxy). The final decision to mandate, recommend or allow for media key negotiation over DTLS must be made by the RTCWEB WG before any RTC JavaScript interface can be completed.


--- back
