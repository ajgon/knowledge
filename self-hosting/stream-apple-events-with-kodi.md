# Stream Apple Events with Kodi

* Go to [apple event page](https://www.apple.com/apple-events/livestream/)
* In source code look for a tag `<meta property="url-json" content="...">`.
* Visit the URL in `content` attribute.
* It should contain a JSON with `videoSrc.hls` attribute which is URL to the stream.
  * If it's not there (only `{"before": false}`), you need to wait for it to appear.
* On kodi machine create a `livetv.strm` file with the `videoSrc.hls` URL inside.
* Open this file in kodi.
