# WebRTC Graduation Project
## Popular WebRTC JS APIs
* **getUserMedia():** Capture audio and video
* **MediaRecorder():** Record audio and video
* **RTCPeerConnection():** Stream audio and video between users
* **RTCDataChannel():** Stream data between users

**In order to abstract away the browser differences with the WebRTC shim, adapter.js has to be included in the project!!!**
## Setting constraints and Applying CSS filters to Video element
```
video{
  -webkit-filter: blur(4px) invert(1) opacity(0.5);
}
```
```
video {
filter: hue-rotate(180deg) saturate(200%);
-moz-filter: hue-rotate(180deg) saturate(200%);
-webkit-filter: hue-rotate(180deg) saturate(200%);
}
```
[This demo](https://webrtc.github.io/samples/src/content/peerconnection/constraints/) shows ways to use constraints and statistics in WebRTC applications.

## Signalling
Signalling is a process of coordinating a communication. In order to setup a call, below information needs to be exchanged: 

* Session control messages used to open or close communication,
* Error messages, 
* Media metadata, such as codecs and codec settings, bandwidth and media types, 
* Key data, used to establish secure connections, 
* Network data, such as host's IP address and port as seen by the outside world. 

To avoid redundancy and maximize compatibility with established technologies, signalling protocols and standards **are not** specified by the WebRTC standards. 

## RTCPeerConnection
RTCPeerConnection is the API used to create connection between peers and communicate audio and video. 

Imagine Alice trying to call Eve, below is the full mechanism:
* Alice creates an **RTCPeerConnection** object,
* Alice creates an offer (an SDP session description) with the RTCPeerConnection **createOffer()** method,
* Alice calls **setLocalDescription()** with his offer,
* Alice stringfies the offer and uses a signaling mechanism to send it to Eve,
* Eve calls **setRemoteDescription()** with Alice's offer, so that her RTCPeerConnection knows about Alice's setup,
* Eve calls **createAnswer()**, and the success callback for this is passed a local session description: Eve's answer,
* Eve sets her answer as the local description by calling **setLocalDescription()**,
* Eve then uses the signaling mechanism to send her stringified answer back to Alice,
* Alice sets Eve's answer as the remote session description using **setRemoteDescription()**.

They also need to exchange some network information. The expression "finding candidates" refers to the finding the network ports and interfaces using ICE framework.
* Alice creates an RTCPeerConnection object with an **onicecandidate** handler,
* The handler is called when network candidates become available,
* In the handler, Alice sends stringified candidate data to Eve, via their signaling channel,
* When Eve gets a candidate message from Alice, she calls **addIceCandidate()**, to add the candidate to the remote peer description.

[Simple P2P example](https://www.w3.org/TR/webrtc/#simple-peer-to-peer-example) by W3C.

JSEP draft can be accessed [here](https://tools.ietf.org/html/draft-ietf-rtcweb-jsep-03#section-1.1).

After signalling, we should use ICE to cope with NATs and Firewalls WebRTC apps use the ICE framework to overcome the complexities of real-world networking. To do this, your application must pass ICE server URLs to RTCPeerConnection.  
ICE first tries to make a connection using the host address obtained from a device's operating system and network card; if that fails(which it does for devices behind the NATs) ICE obtains an external address using a STUN server,  and if it fails, the traffic is routed via a TURN relay server. A TURN server is a STUN server with added relay functionality built-in. URLs for STUN and TURN servers are optionally specified by an app in the iceServers configuration object that is the first argument to the RTCPeerConnection constructor. Once RTCPeerConnection has this information, the magic happens automatically, it uses this configuration to work out the best path between peers, working with STUN and TURN servers as necessary. 

### STUN Server
NATs provide a device with an IP address for use within a private local network, but this address can't be used externally. Without a public address, there's no way for WebRTC peers to communicate. To get around this problem WebRTC uses STUN. STUN servers live on the public internet and have one simple task: check the IP:port address of an incoming request (from an application running behind a NAT) and send that address back as a response. In other words, the application uses a STUN server to discover its IP:port from a public perspective. This process enables a WebRTC peer to get a publicly accessible address for itself, and then pass that onto another peer via a signaling mechanism, in order to set up a direct link.
### TURN Server
RTCPeerConnection tries to set up direct communication between peers over UDP, if it fails, it switches to TCP, if it also fails, TURN servers can be used as a fallback , relaying data between endpoints. Just to clarify, it is used to stream video, voice, and data between peers, not signalling data.

<img width="480" alt="screen shot 2017-03-23 at 18 31 08" src="https://cloud.githubusercontent.com/assets/18366839/24255557/866d0f66-0fee-11e7-8382-0cdf5395d510.png">

They have public addresses, so they can be contacted by peers, even if they are behind firewalls or NATs. Unlike STUN servers, they consume a lot of bandwidth, this is why they have to be beefier. 

