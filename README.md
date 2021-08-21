twilio-webrtc.js
================

[![NPM](https://img.shields.io/npm/v/%40twilio/webrtc.svg)](https://www.npmjs.com/package/%40twilio/webrtc) [![CircleCI](https://circleci.com/gh/twilio/twilio-webrtc.js/tree/master.svg?style=svg&circle-token=80e91c8284c21ff16d3003702e17b903c0b32f1d)](https://circleci.com/gh/twilio/twilio-webrtc.js/tree/master)

twilio-webrtc.js contains the various WebRTC shims used by twilio-video.js.
It is not intended for general consumption.

## Installation

```
npm install --save @twilio/webrtc
```

## Exports

The following WebRTC API shims are available:

```javascript
const {
  getStats,
  getUserMedia,
  MediaStream,
  MediaStreamTrack,
  RTCIceCandidate,
  RTCPeerConnection
} = require('@twilio/webrtc');
```

### getStats

`getStats` resolves with normalized WebRTC statistics for the active ICE
candidate pair and each `MediaStreamTrack`, local or remote, of a particular
`RTCPeerConnection`.

```javascript
/**
 * Get the statistics for a given RTCPeerConnection.
 * @param {RTCPeerConnection} peerConnection
 * @returns {Promise<StandardizedStatsResponse>}
 */
function getStats(peerConnection) {}
```

__NOTE__: [StandardizedStatsResponse](https://github.com/twilio/twilio-webrtc.js/blob/master/lib/getstats.js#L299)
normalizes the different formats of the stats returned by `RTCPeerConnection#getStats` in different
browsers. It does not conform to the [W3C spec](https://www.w3.org/TR/webrtc-stats/).

### getUserMedia

`getUserMedia` accepts a `MediaStreamConstraints` object and resolves
with a `MediaStream`. By default, it requests both audio and video.

```javascript
/**
 * Request media from the user.
 * @param {MediaStreamConstraints} [constraints={audio: true, video: true}]
 * @returns {Promise<MediaStream>}
 */
function getUserMedia(constraints) {}
```

### RTCPeerConnection

`RTCPeerConnection` abstracts away some of the browser-specific implementations
of WebRTC, and implements some WebRTC features that are not present in some
browsers.

#### Chrome
* Provides a workaround for the case where, when the SSRC of a `MediaStreamTrack` changes, the
  browser treats this as a removal of the existing `MediaStreamTrack` and the addition of a new
  `MediaStreamTrack`.

#### Firefox
* For new offers, adds support for calling `setLocalDescription` and `setRemoteDescription` in
  `have-local-offer` and `have-remote-offer` signaling states respectively.
* Adds support for calling `createOffer` in signaling state `have-local-offer`.
* The above features are implemented using rollback to work around [this bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1072388).
* Provides a workaround for [this bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1240897), where the browser may
  change the previously negotiated DTLS role in an answer, which breaks Chrome.
* Provides a workaround for [this bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1363815),
  where the browser throws when `RTCPeerConnection.prototype.peerIdentity` is accessed.
* Works around Firefox [Bug 1480277](https://bugzilla.mozilla.org/show_bug.cgi?id=1480277).

#### Safari
* Adds rollback support, according to the workaround specified [here](https://bugs.chromium.org/p/webrtc/issues/detail?id=5738#c3).
* Provides a workaround for the case where, when the SSRC of a `MediaStreamTrack` changes, the
  browser treats this as a removal of the existing `MediaStreamTrack` and the addition of a new
  `MediaStreamTrack`.
* Provides a workaround for [this bug](https://github.com/webrtc/adapter/issues/714), where webrtc-adapter's shimmed
  `addTrack` method does not return the `RTCRtpSender` associated with the added track.

### Others

`MediaStream`, `MediaStreamTrack`, and `RTCIceCandidate` abstracts away their
browser-prefixed counterparts for earlier browser versions.
