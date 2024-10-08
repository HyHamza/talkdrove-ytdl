# Hey there! talkdrove-ytdl is created by Hamza, all the docs are wrriten below




# talkdrove-ytdl

![Depfu](https://img.shields.io/depfu/HyHamza/talkdrove-ytdl)
[![codecov](https://codecov.io/gh/HyHamza/talkdrove-ytdl/branch/master/graph/badge.svg)](https://codecov.io/gh/HyHamza/talkdrove-ytdl)

Yet another YouTube downloading module. Written with only Javascript and a node-friendly streaming interface by Hamza.

# Support
You can contact us for support on our [TalkDrove](https://chat.talkdrove.cc.nf/talkdrove)

# WhatsApp channel:
Here is our WhatsApp channel link [WhatsApp](https://whatsapp.com/channel/0029VaNRcHSJP2199iMQ4W0l)

# Usage

```js
const fs = require('fs');
const talkdroveYtdl = require('talkdrove-ytdl');
// TypeScript: import talkdrove-ytdl from 'talkdrove-ytdl'; with --esModuleInterop
// TypeScript: import * as talkdrove-ytdl from 'talkdrove-ytdl'; with --allowSyntheticDefaultImports
// TypeScript: import talkdrove-ytdl = require('talkdrove-ytdl'); with neither of the above

talkdroveYtdl('https://www.youtube.com/watch?v=jBSyDNLtFWw')
  .pipe(fs.createWriteStream('talkdrove-video.mp4'));
```


# API
### talkdrove-ytdl(url, [options])

Attempts to download a video from the given url. Returns a [readable stream](https://nodejs.org/api/stream.html#stream_class_stream_readable). `options` can have the following, in addition to any [`getInfo()` option](#async-talkdrove-ytdlgetinfourl-options) and [`chooseFormat()` option](#talkdrove-ytdlchooseformatformats-options).

* `range` - A byte range in the form `{start: INT, end: INT}` that specifies part of the file to download, ie {start: 10355705, end: 12452856}. Not supported on segmented (DASH MPD, m3u8) formats.
  * This downloads a portion of the file, and not a separately spliced video.
* `begin` - What time in the video to begin. Supports formats `00:00:00.000`, `0ms, 0s, 0m, 0h`, or number of milliseconds. Example: `1:30`, `05:10.123`, `10m30s`.
  * For live videos, this also accepts a unix timestamp or Date object, and defaults to `Date.now()`.
  * This option is not very reliable for non-live videos, see [#129](https://github.com/HyHamza/talkdrove-ytdl/issues/129) and [#219](https://github.com/HyHamza/talkdrove-ytdl/issues/219).
* `liveBuffer` - How much time buffer to use for live videos in milliseconds. Default is `20000`.
* `highWaterMark` - How much of the video download to buffer into memory. See [node's docs](https://nodejs.org/api/stream.html#stream_constructor_new_stream_writable_options) for more. Defaults to 512KB.
* `dlChunkSize` - When the chosen format is video only or audio only, the download is separated into multiple chunks to avoid throttling. This option specifies the size of each chunk in bytes. Setting it to 0 disables chunking. Defaults to 10MB.
* `IPv6Block` - IPv6 block to rotate through, an alternative to using a proxy. [Read more](#how-does-using-an-ipv6-block-help). Defaults to `undefined`.

#### Event: info
* [`talkdrove-ytdl.videoInfo`](typings/index.d.ts#L235) - Info.
* [`talkdrove-ytdl.videoFormat`](typings/index.d.ts#L32) - Video Format.

Emitted when the video's `info` is fetched, along with the chosen format to download.

#### Event: progress
* `number` - Chunk length in bytes or segment number.
* `number` - Total bytes or segments downloaded.
* `number` - Total bytes or segments.

Emitted whenever a new chunk is received. Passes values describing the download progress.

#### miniget events

All [miniget events](https://github.com/HyHamza/node-miniget#event-redirect) are forwarded and can be listened to from the returned stream.

### Stream#destroy()

Call to abort and stop downloading a video.

### async talkdrove-ytdl.getBasicInfo(url, [options])

Use this if you only want to get metainfo from a video.

### async talkdrove-ytdl.getInfo(url, [options])

Gets metainfo from a video. Includes additional formats, and ready to download deciphered URL. This is what the `talkdrove-ytdl()` function uses internally.

`options` can have the following

* `requestOptions` - Anything to merge into the request options which [miniget](https://github.com/HyHamza/node-miniget) is called with, such as `headers`.
* `requestCallback` - Provide a callback function that receives miniget request stream objects used while fetching metainfo.
* `lang` - The 2 character symbol of a language. Default is `en`.

### talkdrove-ytdl.downloadFromInfo(info, options)

Once you have received metadata from a video with the `talkdrove-ytdl.getInfo` function, you may pass that information along with other options to this function.

### talkdrove-ytdl.chooseFormat(formats, options)

Can be used if you'd like to choose a format yourself. Throws an Error if it fails to find any matching format.

`options` can have the following

* `quality` - Video quality to download. Can be an [itag value](http://en.wikipedia.org/wiki/YouTube#Quality_and_formats), a list of itag values, or one of these strings: `highest`/`lowest`/`highestaudio`/`lowestaudio`/`highestvideo`/`lowestvideo`. `highestaudio`/`lowestaudio` try to minimize video bitrate for equally good audio formats while `highestvideo`/`lowestvideo` try to minimize audio respectively. Defaults to `highest`, which prefers formats with both video and audio.

  A typical video's formats will be sorted in the following way using `quality: 'highest'`
  ```
  itag container quality codecs                 bitrate  audio bitrate
  18   mp4       360p    avc1.42001E, mp4a.40.2 696.66KB 96KB
  137  mp4       1080p   avc1.640028            4.53MB
  248  webm      1080p   vp9                    2.52MB
  136  mp4       720p    avc1.4d4016            2.2MB
  247  webm      720p    vp9                    1.44MB
  135  mp4       480p    avc1.4d4014            1.1MB
  134  mp4       360p    avc1.4d401e            593.26KB
  140  mp4               mp4a.40.2                       128KB
  ```
  format 18 at 360p will be chosen first since it's the highest quality format with both video and audio. If you'd like a higher quality format with both video and audio, see the section on [handling separate streams](#handling-separate-streams).
* `filter` - Used to filter the list of formats to choose from. Can be `audioandvideo` or `videoandaudio` to filter formats that contain both video and audio, `video` to filter for formats that contain video, or `videoonly` for formats that contain video and no additional audio track. Can also be `audio` or `audioonly`. You can give a filtering function that gets called with each format available. This function is given the `format` object as its first argument, and should return true if the format is preferable.
  ```js
  // Example with custom function.
  talkdrove-ytdl(url, { filter: format => format.container === 'mp4' })
  ```
* `format` - Primarily used to download specific video or audio streams. This can be a specific `format` object returned from `getInfo`.
  * Supplying this option will ignore the `filter` and `quality` options since the format is explicitly provided.

```js
// Example of choosing a video format.
let info = await talkdrove-ytdl.getInfo(videoID);
let format = talkdrove-ytdl.chooseFormat(info.formats, { quality: '134' });
console.log('Format found!', format);
```

### talkdrove-ytdl.filterFormats(formats, filter)

If you'd like to work with only some formats, you can use the [`filter` option above](#talkdrove-ytdlurl-options).

```js
// Example of filtering the formats to audio only.
let info = await talkdrove-ytdl.getInfo(videoID);
let audioFormats = talkdrove-ytdl.filterFormats(info.formats, 'audioonly');
console.log('Formats with only audio: ' + audioFormats.length);
```

### talkdrove-ytdl.validateID(id)

Returns true if the given string satisfies YouTube's ID format.

### talkdrove-ytdl.validateURL(url)

Returns true if able to parse out a valid video ID.

### talkdrove-ytdl.getURLVideoID(url)

Returns a video ID from a YouTube URL.
Throws an Error if it fails to parse an ID.

### talkdrove-ytdl.getVideoID(str)

Same as the above `talkdrove-ytdl.getURLVideoID()`, but can be called with the video ID directly, in which case it returns it. This is what talkdrove-ytdl uses internally.
Throws an Error if it fails to parse an ID.

### talkdrove-ytdl.version

The version string taken directly from the package.json.

## Limitations

talkdrove-ytdl cannot download videos that fall into the following
* Regionally restricted (requires a [proxy](example/proxy.js))
* Private (if you have access, requires [cookies](example/cookies.js))
* Rentals (if you have access, requires [cookies](example/cookies.js))
* YouTube Premium content (if you have access, requires [cookies](example/cookies.js))
* Only [HLS Livestreams](https://en.wikipedia.org/wiki/HTTP_Live_Streaming) are currently supported. Other formats will get filtered out in talkdrove-ytdl.chooseFormats

Generated download links are valid for 6 hours, and may only be downloadable from the same IP address.

### Ratelimits
When doing too many requests, YouTube might block. This will result in your requests getting denied with HTTP-StatusCode 429. The following Steps might help you:
* Update talkdrove-ytdl to the latest version
* Use proxies (you can find an example [here](https://github.com/HyHamza/talkdrove-ytdl/blob/master/example/proxy.js))
* Extend on the Proxy Idea by rotating (IPv6-)Addresses
  * read [this](#How-does-using-an-IPv6-block-help?) for more information about this
* Use cookies (you can find an example [here](https://github.com/HyHamza/talkdrove-ytdl/blob/master/example/cookies.js))
  * for this to take effect you have to FIRST wait for the current ratelimit to expire
* Wait it out (it usually goes away within a few days)

#### How does using an IPv6 block help?

For request-intensive tasks it might be useful to spread your requests across multiple source IP-Addresses. Changing the source IP that you use is similar to using a proxy, except without bypassing restrictions such as a region lock. More IP-Addresses result in less requests per IP and therefor increase your ratelimit. Since IPv4 Addresses are a limited Resource we advise to use IPv6.

Using an IPv6 block is essentially having millions of IPv6 addresses at your request. In a /64 IPv6 block (which is usually the Block given to a single Household), there are 18,446,744,073,709,551,616 unique IPv6 addresses. This would allow you to make each request with a different IPv6 address.

Even though using an IP-Block does help against ratelimits it requires you to setup your host system to accept http traffic from every message in an IP-Block. We can not help you with the setup for any specific host / hosting provider but searching the internet most likely can.

## Handling Separate Streams

Typically 1080p or better videos do not have audio encoded with it. The audio must be downloaded separately and merged via an encoding library. `ffmpeg` is the most widely used tool, with many [Node.js modules available](https://www.npmjs.com/search?q=ffmpeg). Use the `format` objects returned from `talkdrove-ytdl.getInfo` to download specific streams to combine to fit your needs. Look at [example/ffmpeg.js](example/ffmpeg.js) for an example on doing this.

## What if it stops working?

YouTube updates their website all the time, it's not that rare for this to stop working. If it doesn't work for you and you're using the latest version, feel free to open up an issue. Make sure to check if there isn't one already with the same error.

Run the tests at `test/irl-test.js` to make sure this is really an issue with talkdrove-ytdl.

    npm run test:irl

These tests are not mocked, they try to start downloading a few videos. If these fail, then it's time to debug. If the error you're getting is signature deciphering, check `lib/sig.js`. Otherwise, the error is likely within `lib/info.js`.

# Install

```bash
npm install talkdrove-ytdl@latest
```

Or for Yarn users:
```bash
yarn add talkdrove-ytdl@latest
```


## Update Checks

The issue of using an outdated version of talkdrove-ytdl became so prevalent, that talkdrove-ytdl now checks for updates at run time, and every 12 hours. If it finds an update, it will print a warning to the console advising you to update. Due to the nature of this library, it is important to always use the latest version as YouTube continues to update.

If you'd like to disable this update check, you can do so by providing the `talkdrove-YTDL_NO_UPDATE` env variable.

```
env talkdrove-YTDL_NO_UPDATE=1 node myapp.js
```

# Related Projects

- [talkdrove-ytdl](https://github.com/HyHamza/talkdrove-ytdl) - A cli wrapper of this.
- [pully](https://github.com/JimmyBoh/pully) - Another cli wrapper of this aimed at high quality formats.
- [ytsr](https://github.com/TimeForANinja/node-ytsr) - YouTube video search results.
- [ytpl](https://github.com/TimeForANinja/node-ytpl) - YouTube playlist and channel resolver.


# Tests
Tests are written with [mocha](https://mochajs.org)

```bash
npm test
```
# Thanks for using talkdrove-ytdl

```Created by Hamza!
Powered by TalkDrove
```