# WebRTC in a Blazor Application
Blazor Webassembly has no access to the Browser Api through C#. The WebRTC implementation built into modern browsers therefor has to be accessed through helper functions in Javascript.

## WebRTC
WebRTC is a highlevel api which allows managing a realtime connection between two clients in a peer 2 peer manner (a "direct" connection without a server, albeit a TURN server for forwarding is required in case NAT traversals prevent a direct connection). The browser apis implementation is specifically catering to video and audio transmitting, having built in support for the browsers MediaStream / MediaStreamTrack types.
## P2P Call Process
1. Request access to camera and audio:

    Any script wanting to utilise camera or microphone input has to request access from the browser and in extension the user. Some browsers do not expose the list of devices without this access.

    1. Request camera and microphone access
    1. Get the list of audio and video input devices
    1. The user chooses which inputs (if any) to activate
1. Configuring the RTC Peer connection
    1. Get a list of STUN / TURN configurations from the servers
    1. Attach Streams and configure the data channel
1. Establishing connection
    
    Set local/remote descriptions and Forward negotiation messages. [Useful Guide on the topic](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Perfect_negotiation).

    Negotiation messages are delivered between peers via SignalR hubs.
## Issues
* Whenever the browser Api is invoked from JS interop, promise rejections or other exceptions are not guaranteed to end up in the browser console (unlike .NET WASM exceptions). As a solution browser Api calls are wrapped in try catch blocks.
* There are instances where a browser api calls simply never return. As a solution calls to the browser api are raced against 1 second timeout.
* RTCPeerConnection doesn't begin negotiating if there is no payload (track or datachannel) attached. A solution is to attach a datachannel by default (wether its actually used or not)
* There is no way of marking streams attached to a RTCPeerConnection
    * Stream Ids, Track Ids, Track labels sometimes are preserved, sometimes are not depending on the clients browser.
    * Changes to the Sdp string will, depending on the clients browser work, cause irrecoverable errors or are simply reset.

    As a consequence it is a big challenge to determine wether an incoming video stream is a camera or a screen capture. The final "solution" for this implementation was to just not label incoming streams in the UI.