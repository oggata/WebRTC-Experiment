## [MediaStreamRecorder.js](https://github.com/streamproc/MediaStreamRecorder) - [Demos](https://www.webrtc-experiment.com/msr/) -   [![npm](https://img.shields.io/npm/v/msr.svg)](https://npmjs.org/package/msr) [![downloads](https://img.shields.io/npm/dm/msr.svg)](https://npmjs.org/package/msr)

A cross-browser implementation to record audio/video streams:

1. MediaStreamRecorder can record both audio and video in single WebM file on Firefox.
2. MediaStreamRecorder can record audio as WAV and video as either WebM or animated gif on Chrome.

MediaStreamRecorder is useful in scenarios where you're planning to submit/upload recorded blobs in realtime to the server! You can get blobs after specific time-intervals.

## [Demos](https://www.webrtc-experiment.com/msr/) using [MediaStreamRecorder.js](https://github.com/streamproc/MediaStreamRecorder) library

| Experiment Name        | Demo           | Source Code |
| ------------- |-------------|-------------|
| **Audio Recording** | [Demo](https://www.webrtc-experiment.com/msr/audio-recorder.html) | [Source](https://github.com/streamproc/MediaStreamRecorder/tree/master/demos/audio-recorder.html) |
| **Video Recording** | [Demo](https://www.webrtc-experiment.com/msr/video-recorder.html) | [Source](https://github.com/streamproc/MediaStreamRecorder/tree/master/demos/video-recorder.html) |
| **Gif Recording** | [Demo](https://www.webrtc-experiment.com/msr/gif-recorder.html) | [Source](https://github.com/streamproc/MediaStreamRecorder/tree/master/demos/gif-recorder.html) |
| **MultiStreamRecorder Demo** | [Demo](https://www.webrtc-experiment.com/msr/MultiStreamRecorder.html) | [Source](https://github.com/streamproc/MediaStreamRecorder/tree/master/demos/MultiStreamRecorder.html) |

----

There is a similar project: **RecordRTC**! [Demo](https://www.webrtc-experiment.com/RecordRTC/) - [Documentation](https://github.com/muaz-khan/WebRTC-Experiment/tree/master/RecordRTC)

## How to link scripts?

You can [install scripts using NPM](https://www.npmjs.org/package/msr):

```javascript
npm install msr
```

Then link single/standalone "MediaStreamRecorder.js" file:

```html
<script src="./node_modules/msr/MediaStreamRecorder.js"> </script>
```

## Otherwise, you can "directly" link standalone file from CDN:

```html
<script src="https://cdn.webrtc-experiment.com/MediaStreamRecorder.js"> </script>
```

## Otherwise, you can link specific files:

* https://github.com/streamproc/MediaStreamRecorder/blob/master/How-to-Link-Specific-Files.md

## Record audio+video in Firefox in single WebM

```html
<script src="https://cdn.webrtc-experiment.com/MediaStreamRecorder.js"> </script>
<script>
var mediaConstraints = {
    audio: !!navigator.mozGetUserMedia, // don't forget audio!
    video: true                         // don't forget video!
};

navigator.getUserMedia(mediaConstraints, onMediaSuccess, onMediaError);

function onMediaSuccess(stream) {
    var mediaRecorder = new MediaStreamRecorder(stream);
    mediaRecorder.mimeType = 'video/webm';
    mediaRecorder.ondataavailable = function (blob) {
        // POST/PUT "Blob" using FormData/XHR2
        var blobURL = URL.createObjectURL(blob);
        document.write('<a href="' + blobURL + '">' + blobURL + '</a>');
    };
    mediaRecorder.start(3000);
}

function onMediaError(e) {
    console.error('media error', e);
}
</script>
```

## Record audio+video in Chrome

`MultiStreamRecorder.js` records both audio/video and returns both blobs in single `ondataavailable` event.

```html
<script src="https://cdn.webrtc-experiment.com/MediaStreamRecorder.js"> </script>
<script>
var mediaConstraints = {
    audio: true,
    video: true
};

navigator.getUserMedia(mediaConstraints, onMediaSuccess, onMediaError);

function onMediaSuccess(stream) {
    var multiStreamRecorder = new MultiStreamRecorder(stream);
    multiStreamRecorder.ondataavailable = function (blobs) {
        // blobs.audio
        // blobs.video
    };
    multiStreamRecorder.start(3 * 1000);
}

function onMediaError(e) {
    console.error('media error', e);
}
</script>
```

## Record only audio in Chrome/Firefox

```html
<script src="https://cdn.webrtc-experiment.com/MediaStreamRecorder.js"> </script>
```

```javascript
var mediaConstraints = {
    audio: true
};

navigator.getUserMedia(mediaConstraints, onMediaSuccess, onMediaError);

function onMediaSuccess(stream) {
    var mediaRecorder = new MediaStreamRecorder(stream);
    mediaRecorder.mimeType = 'audio/ogg';
    mediaRecorder.ondataavailable = function (blob) {
        // POST/PUT "Blob" using FormData/XHR2
        var blobURL = URL.createObjectURL(blob);
        document.write('<a href="' + blobURL + '">' + blobURL + '</a>');
    };
    mediaRecorder.start(3000);
}

function onMediaError(e) {
    console.error('media error', e);
}
```

## Record only-video in chrome

```html
<script src="https://cdn.webrtc-experiment.com/MediaStreamRecorder.js"> </script>
<script>
var mediaConstraints = {
    video: true
};

navigator.getUserMedia(mediaConstraints, onMediaSuccess, onMediaError);

function onMediaSuccess(stream) {
    var mediaRecorder = new MediaStreamRecorder(stream);
    mediaRecorder.mimeType = 'video/webm';
	
    // for gif recording
    // mediaRecorder.mimeType = 'image/gif';
	
    mediaRecorder.width = 320;
    mediaRecorder.height = 240;
	
    mediaRecorder.ondataavailable = function (blob) {
        // POST/PUT "Blob" using FormData/XHR2
        var blobURL = URL.createObjectURL(blob);
        document.write('<a href="' + blobURL + '">' + blobURL + '</a>');
    };
    mediaRecorder.start(3000);
}

function onMediaError(e) {
    console.error('media error', e);
}
</script>
```

## How to manually stop recordings?

```javascript
mediaRecorder.stop();
```

## How to upload recorded files using PHP?

**PHP code:**

```php
<?php
foreach(array('video', 'audio') as $type) {
    if (isset($_FILES["${type}-blob"])) {
        
		$fileName = $_POST["${type}-filename"];
        $uploadDirectory = "uploads/$fileName";
        
        if (!move_uploaded_file($_FILES["${type}-blob"]["tmp_name"], $uploadDirectory)) {
            echo("problem moving uploaded file");
        }
		
		echo($uploadDirectory);
    }
}
?>
```

**JavaScript Code:**

```javascript
var fileType = 'video'; // or "audio"
var fileName = 'ABCDEF.webm';  // or "wav" or "ogg"

var formData = new FormData();
formData.append(fileType + '-filename', fileName);
formData.append(fileType + '-blob', blob);

xhr('save.php', formData, function (fileURL) {
    window.open(fileURL);
});

function xhr(url, data, callback) {
    var request = new XMLHttpRequest();
    request.onreadystatechange = function () {
        if (request.readyState == 4 && request.status == 200) {
            callback(location.href + request.responseText);
        }
    };
    request.open('POST', url);
    request.send(data);
}
```

## Browser Support

| Browser        | Support           |
| ------------- |-------------|
| Firefox | [Stable](http://www.mozilla.org/en-US/firefox/new/) / [Aurora](http://www.mozilla.org/en-US/firefox/aurora/) / [Nightly](http://nightly.mozilla.org/) |
| Google Chrome | [Stable](https://www.google.com/intl/en_uk/chrome/browser/) / [Canary](https://www.google.com/intl/en/chrome/browser/canary.html) / [Beta](https://www.google.com/intl/en/chrome/browser/beta.html) / [Dev](https://www.google.com/intl/en/chrome/browser/index.html?extra=devchannel#eula) |
| Opera | [Stable](http://www.opera.com/) / [NEXT](http://www.opera.com/computer/next)  |
| Android | [Chrome](https://play.google.com/store/apps/details?id=com.chrome.beta&hl=en) / [Firefox](https://play.google.com/store/apps/details?id=org.mozilla.firefox) / [Opera](https://play.google.com/store/apps/details?id=com.opera.browser) |

## Contributors

1. [Muaz Khan](https://github.com/muaz-khan)
2. [neizerth](https://github.com/neizerth)
3. [andersaloof](https://github.com/andersaloof)

## License

[MediaStreamRecorder.js](https://github.com/streamproc/MediaStreamRecorder) library is released under [MIT licence](https://www.webrtc-experiment.com/licence/).
