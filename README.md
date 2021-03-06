## [RTCMultiConnection-v3.0](https://github.com/muaz-khan/RTCMultiConnection/tree/master/RTCMultiConnection-v3.0) / [Many Live Demos](https://rtcmulticonnection.herokuapp.com/)  

[![npm](https://img.shields.io/npm/v/rtcmulticonnection-v3.svg)](https://npmjs.org/package/rtcmulticonnection-v3) [![downloads](https://img.shields.io/npm/dm/rtcmulticonnection-v3.svg)](https://npmjs.org/package/rtcmulticonnection-v3) [![Build Status: Linux](https://travis-ci.org/muaz-khan/RTCMultiConnection.png?branch=master)](https://travis-ci.org/muaz-khan/RTCMultiConnection)

Fetch latest code:

```
sudo npm install rtcmulticonnection-v3

# or MOST preferred one
mkdir RTCMultiConnection-v3.0 && cd RTCMultiConnection-v3.0
wget http://dl.webrtc-experiment.com/rtcmulticonnection-v3.tar.gz
tar -zxvf rtcmulticonnection-v3.tar.gz
ls -a
```

* [rtcmulticonnection-v3.tar.gz](http://dl.webrtc-experiment.com/rtcmulticonnection-v3.tar.gz)

To TEST:

```
npm start

# or
node server.js

# if fails,
lsof -n -i4TCP:9001 | grep LISTEN
kill process-ID

# or kill specific port
# it may require "sudo" privileges
fuser -vk 9001/tcp
```

Now open: `https://localhost:9001/`

## Keep running server in background

```
nohup nodejs server.js > /dev/null 2>&1 &
```

## Link Single File

```html
<script src="/RTCMultiConnection.js"></script>

<!-- or minified file -->
<script src="/RTCMultiConnection.min.js"></script>
```

If you're sharing files, you also need to link:

```html
<script src="/dev/FileBufferReader.js"></script>
```

You can link multiple files from "dev" directory.

## Set different socket URL

Either via `config.json` file:

```json
{
  "socketURL": "/",
  "socketMessageEvent": "RTCMultiConnection-Message"
}
```

or override in your HTML code:

```javascript
connection.socketURL = 'http://yourdomain.com:8080/';

// if your server is already having "message" event
// then you can use something else, unique.
connection.socketMessageEvent = 'unique-message';
```

**For testing purpose**, you can use this as well:

```json
{
  "socketURL": "https://rtcmulticonnection.herokuapp.com:443/",
  "socketMessageEvent": "RTCMultiConnection-Message"
}
```

or

```javascript
connection.socketURL = 'https://rtcmulticonnection.herokuapp.com:443/';
```

Here is a demo explaining how to use above `socketURL`:

* [https://jsfiddle.net/zd9Lsdfk/](https://jsfiddle.net/zd9Lsdfk/)

## Integrate in your own applications?

```javascript
// node.js code
require('./Signaling-Server.js')(httpServerHandlerOrPort);
```

## Migrating from older versions?

Use `streamEvents` instead of `connection.streams`:

```javascript
var stream = connection.streamEvents['streamid'];

# you can even use "getStreamById"
var stream = connection.attachStreams.getStreamById('streamid');

# to get remote stream by id
var allRemoteStreams = connection.getRemoteStreams('remote-user-id');
var stream = allRemoteStreams.getStreamByid('streamid');
```

Wanna check `isScreen` or `isVideo` or `isAudio`?

```javascript
connection.onstream = function(event) {
	if(event.stream.isScreen) {
		// screen stream
	}

	if(event.stream.isVideo) {
		// audio+video or video-only stream
	}

	if(event.stream.isAudio) {
		// audio-only stream
	}
};
```

Wanna mute/unmute?

```javascript
var stream = connection.streamEvents['streamid'].stream;
stream.mute('audio'); // audio or video or both
```

# API

### `applyConstraints`

This method allows you change video resolutions or audio sources without making a new getUserMedia request i.e. it modifies your existing MediaStream:

```javascript
var width = 1280;
var height = 720;

var supports = navigator.mediaDevices.getSupportedConstraints();

var constraints = {};
if (supports.width && supports.height) {
    constraints = {
        width: width,
        height: height
    };
}

connection.applyConstraints({
    video: constraints
});
```

`applyConstraints` access `mediaConstraints` object, defined here:

* [http://www.rtcmulticonnection.org/docs/mediaConstraints/](http://www.rtcmulticonnection.org/docs/mediaConstraints/)

### `replaceTrack`

This method allows you replace your front-camera video with back-camera video or replace video with screen or replace older low-quality video with new high quality video.

```javascript
// here is its simpler usage
connection.replaceTrack({
	screen: true,
	oneway: true
});
```

You can even pass `MediaStreamTrack` object:

```javascript
var videoTrack = yourVideoStream.getVideoTracks()[0];
connection.replaceTrack(videoTrack);
```

You can even pass `MediaStream` object:

```javascript
connection.replaceTrack(yourVideoStream);
```

You can even force to replace tracks only with a single user:

```javascript
var remoteUserId = 'single-remote-userid';

var videoTrack = yourVideoStream.getVideoTracks()[0];
connection.replaceTrack(videoTrack, remoteUserId);
```

### `onUserStatusChanged`

This even allows you show online/offline statuses of the user:

```javascript
connection.onUserStatusChanged = function(status) {
	document.getElementById(event.userid).src = status === 'online' ? 'online.gif' : 'offline.gif';
};
```

### `getSocket`

This method allows you get the `socket` object used for signaling (handshake/presence-detection/etc.):

```javascript
var socket = connection.getSocket();
socket.emit('custom-event', 'hi there');
socket.on('custom-event', function(message) {
	alert(message);
});
```

If socket isn't connected yet, then above method will auto-connect it. It is using `connectSocket` to connect socket. See below section.

### `connectSocket`

It is same like old RTCMultiConnection `connect` method:

* [http://www.rtcmulticonnection.org/docs/connect/](http://www.rtcmulticonnection.org/docs/connect/)

`connectSocket` method simply initializes socket.io server so that you can send custom-messages before creating/joining rooms:

```javascript
connection.connectSocket(function(socket) {
	socket.on('custom-message', function(message) {
		alert(message);

		// custom message
		if(message.joinMyRoom) {
			connection.join(message.roomid);
		}
	});

	socket.emit('custom-message', 'hi there');

	connection.open('room-id');
});
```

### `getUserMediaHandler`

This object allows you capture audio/video stream yourself. RTCMultiConnection will NEVER know about your stream until you add it yourself, manually:

```javascript
var options = {
	localMediaConstraints: {
		audio: true,
		video: true
	},
	onGettingLocalMedia: function(stream) {},
	onLocalMediaError: function(error) {}
};
connection.getUserMediaHandler(options);
```

Its defined here:

* [getUserMedia.js#L20](https://github.com/muaz-khan/RTCMultiConnection/blob/master/RTCMultiConnection-v3.0/dev/getUserMedia.js#L20)

## `becomePublicModerator`

By default: all moderators are private.

This method returns list of all moderators (room owners) who declared themselves as `public` via `becomePublicModerator` method:

```javascript
# to become a public moderator
connection.open('roomid', true); // 2nd argument is "TRUE"

# or call this method later (any time)
connection.becomePublicModerator();
```

### `getPublicModerators`

You can access list of all the public rooms using this method. This works similar to old RTCMultiConnection method `onNewSession`.

Here is how to get public moderators:

```javascript
connection.getPublicModerators(function(array) {
	array.forEach(function(moderator) {
		// moderator.extra
		connection.join(moderator.userid);
	});
});
```

You can even search for specific moderators. Moderators whose userid starts with specific string:

```javascript
var moderatorIdStartsWith = 'public-moderator-';
connection.getPublicModerators(moderatorIdStartsWith, function(array) {
	// only those moderators are returned here
	// that are having userid similar to this:
	// public-moderator-xyz
	// public-moderator-abc
	// public-moderator-muaz
	// public-moderator-conference
	array.forEach(function(moderator) {
		// moderator.extra
		connection.join(moderator.userid);
	});
});
```

## `setUserPreferences`

You can force `dontAttachStream` and `dontGetRemoteStream` for any or each user in any situation:

```javascript
connection.setUserPreferences = function(userPreferences) {
    if (connection.dontAttachStream) {
    	// current user's streams will NEVER be shared with any other user
        userPreferences.dontAttachLocalStream = true;
    }

    if (connection.dontGetRemoteStream) {
    	// current user will NEVER receive any stream from any other user
        userPreferences.dontGetRemoteStream = true;
    }

    return userPreferences;
};
```

Scenarios:

1. All users in the room are having cameras
2. All users in the room can see only **self video**
3. All users in the room can text-chat or share files; but can't share videos
4. As soon as teacher or moderator or presenter enters in the room; he can ask all the participants or specific participants to share their cameras with single or multiple users.

They can enable cameras as following:

```javascritp
connection.onmessage = function(event) {
	var message = event.data;
	if(message.shareYourCameraWithMe) {
		connection.dontAttachStream = false;
		connection.renegotiate(event.userid); // share only with single user
	}

	if(message.shareYourCameraWithAllUsers) {
		connection.dontAttachStream = false;
		connection.renegotiate(); // share with all users
	}
}
```

i.e. `setUserPreferences` allows you enable camera on demand.

## `checkPresence`

This method allows you check presence of the moderators/rooms:

```javascript
connection.checkPresence('roomid', function(isRoomEists, roomid) {
	if(isRoomEists) {
		connection.join(roomid);
	}
	else {
		connection.open(roomid);
	}
});
```

## `onReadyForOffer`

This even is fired as soon as callee says "I am ready for offer. I enabled camera. Please create offer and share.".

```javascript
connection.onReadyForOffer = function(remoteUserId, userPreferences) {
	// if OfferToReceiveAudio/OfferToReceiveVideo should be enabled for specific users
	userPreferences.localPeerSdpConstraints.OfferToReceiveAudio = true;
	userPreferences.localPeerSdpConstraints.OfferToReceiveVideo = true;

	userPreferences.dontAttachStream = false; // according to situation
	userPreferences.dontGetRemoteStream = false;  // according to situation

	// below line must be included. Above all lines are optional.
	connection.multiPeersHandler.createNewPeer(remoteUserId, userPreferences);
};
```

## `onNewParticipant`

This event is fired as soon as someone tries to join you. You can either reject his request or set preferences.

```javascript
connection.onNewParticipant = function(participantId, userPreferences) {
    // if OfferToReceiveAudio/OfferToReceiveVideo should be enabled for specific users
	userPreferences.localPeerSdpConstraints.OfferToReceiveAudio = true;
	userPreferences.localPeerSdpConstraints.OfferToReceiveVideo = true;

	userPreferences.dontAttachStream = false; // according to situation
	userPreferences.dontGetRemoteStream = false;  // according to situation

	// below line must be included. Above all lines are optional.
	// if below line is NOT included; "join-request" will be considered rejected.
    connection.acceptParticipationRequest(participantId, userPreferences);
};
```

Or:

```javascript
var alreadyAllowed = {};
connection.onNewParticipant = function(participantId, userPreferences) {
	if(alreadyAllowed[participantId]) {
		connection.addParticipationRequest(participantId, userPreferences);
		return;
	}

	var message = participantId + ' is trying to join your room. Confirm to accept his request.';
	if( window.confirm(messsage ) ) {
		connection.addParticipationRequest(participantId, userPreferences);
	}
};
```

## `disconnectWith`

Disconnect with single or multiple users. This method allows you keep connected to `socket` however either leave entire room or remove single or multiple users:

```javascript
connection.disconnectWith('remoteUserId');

// to leave entire room
connection.getAllParticipants().forEach(function(participantId) {
	connection.disconnectWith(participantId);
});
```

## `getAllParticipants`

Get list of all participants that are connected with current user.

```javascript
var numberOfUsersInTheRoom = connection.getAllParticipants().length;

var remoteUserId = 'xyz';
var isUserConnectedWithYou = connection.getAllParticipants().indexOf(remoteUserId) !== -1;

connection.getAllParticipants().forEach(function(remoteUserId) {
	var user = connection.peers[remoteUserId];
	console.log(user.extra);

	user.peer.connection.close();
	alert(user.peer.connection === webkitRTCPeerConnection);
});
```

## `maxParticipantsAllowed`

Set number of users who can join your room.

```javascript
// to allow another single person to join your room
// it will become one-to-one (i.e. you+anotherUser)
connection.maxParticipantsAllowed = 1;
```

## `setCustomSocketHandler`

This method allows you skip Socket.io and force Firebase or PubNub or WebSockets or PHP/ASPNET whatever.

```javascript
connection.setCustomSocketHandler(FirebaseConnection);
```

Please check [`FirebaseConnection`](https://github.com/muaz-khan/RTCMultiConnection/blob/master/RTCMultiConnection-v3.0/dev/FirebaseConnection.js) or [`PubNubConnection.js`](https://github.com/muaz-khan/RTCMultiConnection/blob/master/RTCMultiConnection-v3.0/dev/PubNubConnection.js) to understand how it works.

## `enableLogs`

By default, logs are enabled.

```javascript
connection.enableLogs = false; // to disable logs
```

## `updateExtraData`

You can force all the extra-data to be synced among all connected users.

```javascript
connection.extra.fullName = 'New Full Name';
connection.updateExtraData(); // now above value will be auto synced among all connected users
```

## `onExtraDataUpdated`

This event is fired as soon as extra-data from any user is updated:

```javascript
connection.onExtraDataUpdated = function(event) {
	console.log('extra data updated', event.userid, event.extra);

	// make sure that <video> header is having latest fullName
	document.getElementById('video-header').innerHTML = event.extra.fullName;
};
```

## `streamEvents`

It is similar to this:

* http://www.rtcmulticonnection.org/docs/streams/

## `socketURL`

If socket.io is listening on a separate port or external URL:

```javascript
connection.socketURL = 'https://domain:port/';
```

## `socketOptions`

Socket.io options:

```javascript
connection.socketOptions = {
	'force new connection': true, // For SocketIO version < 1.0
	'forceNew': true, // For SocketIO version >= 1.0
	'transport': 'polling' // fixing transport:unknown issues
};
```

## `DetectRTC`

Wanna detect current browser?

```javascript
if(connection.DetectRTC.browser.isChrome) {
	// it is Chrome
}

// you can even set backward compatibility hack
connection.UA = connection.DetectRTC.browser;
if(connection.UA.isChrome) { }
```

Wanna detect if user is having microphone or webcam?

```javascript
connection.DetectRTC.detectMediaAvailability(function(media){
	if(media.hasWebcam) { }
	if(media.hasMicrophone) { }
	if(media.hasSpeakers) { }
});
```

## `invokeSelectFileDialog`

Get files problematically instead of using `input[type=file]`:

```javascript
connection.invokeSelectFileDialog(function(file) {
	var file = this.files[0];
	if(file){
		connection.shareFile(file);
	}
});
```

## `processSdp`

Force bandwidth, bitrates, etc.

```javascript
var BandwidthHandler = connection.BandwidthHandler;
connection.bandwidth = {
	audio: 128,
	video: 256,
	screen: 300
};
connection.processSdp = function(sdp) {
    sdp = BandwidthHandler.setApplicationSpecificBandwidth(sdp, connection.bandwidth, !!connection.session.screen);
    sdp = BandwidthHandler.setVideoBitrates(sdp, {
        min: connection.bandwidth.video,
        max: connection.bandwidth.video
    });

    sdp = BandwidthHandler.setOpusAttributes(sdp);

    sdp = BandwidthHandler.setOpusAttributes(sdp, {
        'stereo': 1,
        //'sprop-stereo': 1,
        'maxaveragebitrate': connection.bandwidth.audio * 1000 * 8,
        'maxplaybackrate': connection.bandwidth.audio * 1000 * 8,
        //'cbr': 1,
        //'useinbandfec': 1,
        // 'usedtx': 1,
        'maxptime': 3
    });

    return sdp;
};
```

* http://www.rtcmulticonnection.org/docs/processSdp/

## `shiftModerationControl`

Moderator can shift moderation control to any other user:

```javascript
connection.shiftModerationControl('remoteUserId', connection.broadcasters, false);
```

`connection.broadcasters` is the array of users that builds mesh-networking model i.e. multi-user conference.

Moderator shares `connection.broadcasters` with each new participant; so that new participants can connect with other members of the room as well.

## `onShiftedModerationControl`

This event is fired, as soon as moderator of the room shifts moderation control toward you:

```javascript
connection.onShiftedModerationControl = function(sender, existingBroadcasters) {
	connection.acceptModerationControl(sender, existingBroadcasters);
};
```

## `autoCloseEntireSession`

* http://www.rtcmulticonnection.org/docs/autoCloseEntireSession/

## `filesContainer`

A DOM-element to show progress-bars and preview files.

```javascript
connection.filesContainer = document.getElementById('files-container');
```

## `videosContainer`

A DOM-element to append videos or audios or screens:

```javascript
connection.videosContainer = document.getElementById('videos-container');
```

## `addNewBroadcaster`

In a one-way session, you can make multiple broadcasters using this method:

```javascript
if(connection.isInitiator) {
	connection.addNewBroadcaster('remoteUserId');
}
```

Now this user will also share videos/screens.

## `removeFromBroadcastersList`

Remove user from `connection.broadcasters` list.

```javascript
connection.removeFromBroadcastersList('remote-userid');
```

## `onMediaError`

If screen or video capturing fails:

```javascript
connection.onMediaError = function(error) {
	alert( 'onMediaError:\n' + JSON.stringify(error) );
};
```

## `renegotiate`

Recreate peers. Capture new video using `connection.captureUserMedia` and call `connection.renegotiate()` and that new video will be shared with all connected users.

```javascript
connection.renegotiate('with-single-userid');

connection.renegotiate(); // with all users
```

## `addStream`

* http://www.rtcmulticonnection.org/docs/addStream/

You can even pass `streamCallback`:

```javascript
connection.addStream({
	screen: true,
	oneway: true,
	streamCallback: function(screenStream) {
		// this will be fired as soon as stream is captured
		screenStream.onended = function() {
			document.getElementById('share-screen').disabled = false;

			// or show button
			$('#share-screen').show();
		}
	}
});
```

## `mediaConstraints`

* http://www.rtcmulticonnection.org/docs/mediaConstraints/

## `sdpConstraints`

* http://www.rtcmulticonnection.org/docs/sdpConstraints/

## `extra`

* http://www.rtcmulticonnection.org/docs/extra/

## `userid`

* http://www.rtcmulticonnection.org/docs/userid/

## `session`

* http://www.rtcmulticonnection.org/docs/session/

## `enableFileSharing`

To enable file sharing. By default, it is `false`:

```javascript
connection.enableFileSharing = true;
```

## `changeUserId`

Change userid and update userid among all connected peers:

```javascript
connection.changeUserId('new-userid');
```

## `closeBeforeUnload`

It is `true` by default. If you are handling `window.onbeforeunload` yourself, then you can set it to `false`:

```javascript
connection.closeBeforeUnload = false;
window.onbeforeunlaod = function() {
	connection.close();
};
```

## `captureUserMedia`

* http://www.rtcmulticonnection.org/docs/captureUserMedia/

## `open`

Open room:

```javascript
var isPublicRoom = false;
connection.open('roomid', isPublicRoom);

// or
connection.open('roomid', function() {
	// on room created
});
```

## `join`

Join room:

```javascript
connection.join('roomid');

// or pass "options"
connection.join('roomid', {
	localPeerSdpConstraints: {
		OfferToReceiveAudio: true,
		OfferToReceiveVideo: true
	},
	remotePeerSdpConstraints: {
		OfferToReceiveAudio: true,
		OfferToReceiveVideo: true
	},
	isOneWay: false,
	isDataOnly: false
});
```

## `openOrJoin`

```javascript
connection.openOrJoin('roomid');

// or
connection.openOrJoin('roomid', function(isRoomExists, roomid) {
	if(isRoomExists) alert('opened the room');
	else alert('joined the room');
});
```

## `dontCaptureUserMedia`

By default, it is `false`. Which means that RTCMultiConnection will always capture video if `connection.session.video===true`.

If you are attaching external streams, you can ask RTCMultiConnection to DO NOT capture video:

```javascript
connection.dontCaptureUserMedia = true;
```

## `dontAttachStream`

By default, it is `false`. Which means that RTCMultiConnection will always attach local streams.

```javascript
connection.dontAttachStream = true;
```

## `dontGetRemoteStream`

By default, it is `false`. Which means that RTCMultiConnection will always get remote streams.

```javascript
connection.dontGetRemoteStream = true;
```

## Firebase?

If you are willing to use Firebase instead of Socket.io there, open [GruntFile.js](https://github.com/muaz-khan/RTCMultiConnection/blob/master/RTCMultiConnection-v3.0/Gruntfile.js) and replace `SocketConnection.js` with `FirebaseConnection.js`.

Then use `grunt` to recompile RTCMultiConnection.js.

**Otherwise** if you don't want to modify RTCMultiConnection:

```html
<script src="/dev/globals.js"></script>
<script src="/dev/FirebaseConnection.js"></script>
<script>
var connection = new RTCMultiConnection();

connection.firebase = 'your-firebase-account';

// below line replaces FirebaseConnection
connection.setCustomSocketHandler(FirebaseConnection);
</script>
```

Demo: [https://rtcmulticonnection.herokuapp.com/demos/Firebase-Demo.html](https://rtcmulticonnection.herokuapp.com/demos/Firebase-Demo.html)

## PubNub?

Follow above all "firebase" steps and use `PubNubConnection.js` instead.

Please don't forget to use your own PubNub keys.

Demo: [https://rtcmulticonnection.herokuapp.com/demos/PubNub-Demo.html](https://rtcmulticonnection.herokuapp.com/demos/PubNub-Demo.html)

## Configure v3.0

* [wiki/Configure-v3.0](https://github.com/muaz-khan/RTCMultiConnection/wiki/Configure-v3.0)


## Scalable Broadcasting

v3.0 now supports WebRTC scalable broadcasting. Two new API are introduced: `enableScalableBroadcast` and `singleBroadcastAttendees`.

```javascript
connection.enableScalableBroadcast = true; // by default, it is false.
connection.singleBroadcastAttendees = 3;   // how many users are handled by each broadcaster
```

Live Demos:

* [Files-Scalable-Broadcast.html](https://rtcmulticonnection.herokuapp.com/demos/Files-Scalable-Broadcast.html)
* [Video-Scalable-Broadcast.html](https://rtcmulticonnection.herokuapp.com/demos/Video-Scalable-Broadcast.html)

## Fix Echo

```javascript
connection.onstream = function(event) {
	if(event.mediaElement) {
		event.mediaElement.muted = true;
		delete event.mediaElement;
	}

	if(event.stream.mediaElement) {
		event.stream.mediaElement.muted = true;
		delete event.stream.mediaElement;
	}

	var video = document.createElement('video');
	if(event.type === 'local') {
		video.muted = true;
	}
	video.src = URL.createObjectURL(event.stream);
	connection.videosContainer.appendChild(video);
}
```

## License

[RTCMultiConnection](https://github.com/muaz-khan/RTCMultiConnection) is released under [MIT licence](https://github.com/muaz-khan/RTCMultiConnection/blob/master/LICENSE.md) . Copyright (c) [Muaz Khan](http://www.MuazKhan.com/).
