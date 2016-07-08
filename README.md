Caracal
=======

Caracal is a light multimedia server designed as a service.

It provides a simple HTTP REST API and a HTML5 user interface.

### Installation
```npm install caracal```

_GraphicsMagick or ImageMagick is required. FFmpeg (not libav) is necessary for video resizing and converting._

### [Docker](https://registry.hub.docker.com/u/yellowiscool/caracal/)

```sh
docker pull yellowiscool/caracal

docker volume create --name caracal-data

docker run -d --restart=always \
  --name caracal \
  -v caracal-data:/usr/src/app/data \
  -e DELETIONS_KEY=secretkey \
  yellowiscool/caracal
```


### Features

 * File uploads
 * File suppressions
 * Picture and video thumbnails
 * Picture resizing
 * Video resizing and converting
  * Convert videos to H264 or Webm 
 * Can fetch distant HTTP files
  * Multimedia reverse proxy with a cache
  * Transparent resizing
  * Transparent video converting
  * Basic HTTP authentification support
 * Light HTML5 user interface
  * Drag and drop support
  * Progression bar for slow connexions
  * Thumbnails and pagination

### Notes about the scalability

Caracal is designed to run as a single node on a single server. It does not support horizontal scaling. It is suitable for prototyping or simple non-critical multimedia applications, but it cannot be used to compete with Youtube or Imgur (yet). In case of a high load, a processing queue is used.

## Configuration

Caracal can be configured using environment variables.

|Environment Variable|Description|Default value|
|--------------------|-----------|-------------|
|DELETIONS_KEY|The required password/key to delete files.|*needs to be configured*|
|ALLOWED_DOMAINS|Optionnal JSON array of allowed domains, for [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)|`*` (all domains are allowed)|
|HTTP_PORT|Listening HTTP port|8075|
|DATAPATH|Location of the storage folder|`./data`|
|CACHE|[HTTP Cache-Control header](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9)|`max-age=29030400, public`|
|CONCURRENCY|Number of allowed concurrent image processing tasks|8|
|VIDEO_CONCURRENCY|Number of allowed concurrent video processing tasks|2|
|ALLOWED_SIZES|Array of allowed resizing sizes. If a requested size is not found, the closest size from this array is used instead. Set to `*` to allow every size, but it's not recommended since users can saturate the server by resizing the same image many times. The default configuration allows many sizes, you might want to use fewer sizes.|`[32, 64, 128, 256, 1024, 2048, 4096, 8192, 16384,	50, 100, 200, 400, 500, 600, 800, 1000, 1050, 1200, 1600, 120, 160, 240, 320, 480, 576, 640, 768, 854, 960, 1050, 1080, 1152, 1280, 1440, 1536,  1716, 1920, 2160, 2560, 3200,	3840, 3996, 4320, 4800, 5120, 6400, 6144, 7680, 12288]`|
|ALLOWED_VIDEO_SIZES|Array of allowed resizing sizes, for videos. Similar to ALLOWED_SIZES.|`[144, 240, 360, 480, 720, 1080, 3840]`|
|AUTHS|JSON association table (object) used for HTTP [basic access authentications](https://en.wikipedia.org/wiki/Basic_access_authentication#Client_side), when fetching distant files. keys are the host-names, values are the passwords.|`{}`|
|UA|The HTTP client user agent, to fetch distant files|~Firefox28-Win64|


## API

#### GET /

HTML5 user interface.

#### GET /files

JSON list of files stored in the server
```json
[
 {
    "size": 168454,
    "hash": "e8f8f15bfefafae3a19d845b9d5c42dc2014206f",
    "extension": "jpeg",
    "type": "image/jpeg",
    "name": "Veymont-aiguille_mg-k.jpg",
    "url": "http://upload.wikimedia.org/wikipedia/commons/d/d3/Veymont-aiguille_mg-k.jpg",
    "mtime": "2014-04-09T11:36:05.145Z",
    "_id": "sEHgLEvyYAkV94J8"
  },...
]
```

#### POST /upload

Save on the server the provided file
```json
{
  "name": "208T16-05E.jpg",
  "status": "ok",
  "hash": "f2a4b8f39e757e59e89c03f1ec36ada979f75203",
  "extension": "jpeg"
}
```

#### GET /{URL}

If the URL is not in the cache, fetch, save and return the content of the url.

Example : ```GET /http://perdu.com```

#### GET /fetch/{URL}

If the URL is not in the cache fetch and save the content of the url. Then return information about the URL.

Response example :
```json
{
  "status": "ok",
  "hash": "70aa99ede90f16ffbb7cbb66c8bde1a4e8d37383",
  "extension": "jpeg"
}
```

#### GET /remove/{hash}.{extension}

Remove the related files to the hash and the extension from the server.

Example : ```GET /remove/70aa99ede90f16ffbb7cbb66c8bde1a4e8d37383.jpeg```

#### GET /thumbnail/{hash}.{extension}

Create and return a 128x128 thumbnail of the given file. If the format is not supported by GraphicsMagic, an error is returned.

Example : ```GET /thumbnail/70aa99ede90f16ffbb7cbb66c8bde1a4e8d37383.jpeg``

#### GET /resize/{max_width}/{max_height}/{hash}.{extension}

Resize the given image. The image aspect ratio is conserved.

The image format must be compatible with GraphicsMagick.

Example : ```GET /resize/1280/720/70aa99ede90f16ffbb7cbb66c8bde1a4e8d37383.jpeg```

#### GET /resize/deform/{max_width}/{max_height}/{hash}.{extension}

Same as resize but the ratio is not conserved.

#### GET /convert/{format}/{height}/{hash}.{extension}

Resize the given video. The video aspect ratio is conserved.

The input video format must be compatible with your FFmpeg installation.

Supported output formats : mp4 or webm
Supported output sizes : 240, 480, 720, 1080

Example : ```GET /convert/mp4/720/e7c7c984066753d5cc52e97f26f4f7892df67bacb.wmv```

#### GET /thumbnail/{URL}

Create and return a 128x128 thumbnail of the distant image.

Example : ```GET /thumbnail/http://upload.wikimedia.org/wikipedia/commons/d/d3/Veymont-aiguille_mg-k.jpg```

#### GET /resize/{max_width}/{max_height}/{URL}

Resize the distant image. The image aspect ratio is conserved.

Example : ```GET /resize/1280/720/http://upload.wikimedia.org/wikipedia/commons/d/d3/Veymont-aiguille_mg-k.jpg```

#### GET /resize/deform/{max_width}/{max_height}/{URL}

Same as resize but the ratio is not conserved.

#### GET /convert/{format}/{height}/{URL}

Resize the distant video. The video ratio is conserved.

The input video format must be compatible with your FFmpeg installation.

Supported output formats : mp4 or webm
Supported output sizes : 240, 480, 720, 1080

Example : ```GET /convert/webm/480/http://example.net/video.flv```

#### GET /{hash}.{extension}

Return the file :-)

## Behind

 * [NodeJs](http://nodejs.org/)
 * [GraphicsMagick](http://www.graphicsmagick.org/) with [gm](http://aheckmann.github.io/gm/)
 * [FFmpeg](https://www.ffmpeg.org/) with [node-fluent-ffmpeg](https://github.com/fluent-ffmpeg/node-fluent-ffmpeg)
 * [Express](http://expressjs.com/)
 * [Bootstrap](http://getbootstrap.com)
 * [jQuery](http://jquery.com)
 * [jQuery File Upload](http://blueimp.github.io/jQuery-File-Upload/)
 * [NeDB](https://github.com/louischatriot/nedb)

## Configuration

You can change the http listening port, the user-agent and the list of basic http authentification username/password couples in the config.json file.

## Screenshot (work in progress)

![Old screenshot](http://i.imgur.com/vydii2e.png)

### Acknowledgements

This library is developed in context of the [BRIDGE](http://www.bridgeproject.eu/en) project.

### Licence

The source code of this library is licenced under the MIT License.
