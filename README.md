# tinyCam app web server API
This is official documentation of the REST API provided by <a href="https://tinycammonitor.com/">tinyCam Monitor PRO</a> web server - Android app for video surveillance.
tinyCam web server uses API which is partially compatible with <a href="https://www.axis.com/vapix-library">Axis IP cameras</a>. You can use any IP camera viewer software to view tinyCam server remotely.

## Authentication
tinyCam supports three types of authentication:
### 1. <a href="https://en.wikipedia.org/wiki/Basic_access_authentication">Basic authentication</a> (legacy)

Example:
```
https://admin:mypassword@192.168.0.3:8083/axis-cgi/mjpg/video.cgi
```


### 2. Providing credentials as parameters

GET parameters `user=<string>&pwd=<string>`

* `user` (optional) - username,
* `pwd` (optional) - password.

Use it when there is no way to use basic authentication (e.g. for home automation).

Example:
```
https://192.168.0.3:8083/axis-cgi/mjpg/video.cgi?user=admin&pwd=mypassword
```


### 3. <b>Token based authentication</b> (recommended to use)
Access: guest, admin

GET `/api/v1/login?user=<string>&pwd=<string>`

Example:
```
https://192.168.0.3:8083/api/v1/login?user=admin&pwd=mypassword
```

JSON payload:
```
{
	"data": {
		"token": "d6bdc406225bf6bb6ecf720e97e8927acfe52022",
		"access": "admin"
	}
}
```

For subsequent requests, add `token` with token value returned in the login response.

Example:
```
https://192.168.0.3:8083/axis-cgi/mjpg/video.cgi?token=d6bdc406225bf6bb6ecf720e97e8927acfe52022
```

## Logout
Access: guest, admin

GET `/api/v1/logout?token=<string>`

Deletes authentication token.

Example:
```
https://192.168.0.3:8083/api/v1/logout?token=d6bdc406225bf6bb6ecf720e97e8927acfe52022
```

## Get camera list
Access: guest, admin

GET `/api/v1/get_cam_list?token=<string>`

Example:
```
https://192.168.0.3:8083/api/v1/get_cam_list?token=d6bdc406225bf6bb6ecf720e97e8927acfe52022
```

JSON payload:
```
{
	"data": [{
		"name": "Front yard",
		"enabled": true,
		"id": 182399567,
		"ptzCapabilities": 230911,
		"audioListening": true,
		"cloudAccess": false
	}, {
		"name": "Doorbell",
		"enabled": true,
		"id": 1455317736,
		"ptzCapabilities": 32768,
		"audioListening": true,
		"cloudAccess": true
	}]
}
```


## Get camera event list
Access: guest, admin

GET `/api/v1/get_cam_event_list?token=<string>&cameraId=<number>&endtime=<number>&count=<number>&type=<string>&filter=<string>`
* `cameraId` (optional) - camera ID returned by `/api/v1/get_cam_list` request. If empty, events from all enabled cameras returned.
* `endtime` (optional) - time (Unix timestamp in milliseconds) from which events should be returned
* `count` (optional) - max number of events to get
* `type` (optional) - `local` (default) or `cloud` events
* `filter` (optional) - can be `pin`, `face`, `person`, `vehicle`, `pet`, `motion`, or `audio`. If empty, all events returned.

List is returned in descending order (new events first).

Example:
```
 https://192.168.0.3/api/v1/get_cam_event_list?token=d6bdc406225bf6bb6ecf720e97e8927acfe52022&cameraId=182399567&endtime=1601381164000&count=15&type=local&filter=person
```

JSON payload:
```
{
	"data": [{
		"time": 1601381164000,
		"duration": 120000,
		"motion": "person",
		"video": "\/api\/v1\/get_file?file=\/Front%20yard\/2020-09-29%2015.06.04%20120sec%20person.mp4",
		"image": "\/api\/v1\/get_file?file=\/Front%20yard\/2020-09-29%2015.06.04%20120sec%20person.mp4.jpg"
	}, {
		"time": 1601380363000,
		"duration": 58000,
		"motion": "person",
		"video": "\/api\/v1\/get_file?file=\/Front%20yard\/2020-09-29%2014.52.43%2058sec%20person.mp4",
		"image": "\/api\/v1\/get_file?file=\/Front%20yard\/2020-09-29%2014.52.43%2058sec%20person.mp4.jpg"
	}, {
		"time": 1601379016000,
		"duration": 54000,
		"motion": "person",
		"video": "\/api\/v1\/get_file?file=\/Front%20yard\/2020-09-29%2014.30.16%2054sec%20person.mp4",
		"image": "\/api\/v1\/get_file?file=\/Front%20yard\/2020-09-29%2014.30.16%2054sec%20person.mp4.jpg"
	}]
}
```

## Get file

GET `/api/v1/get_file?token=<string>&file=<string>`

* `file` (mandatory) - filename returned by `/api/v1/get_cam_event_list` to be requested (image or video)

Example:
```
https://192.168.0.3/api/v1/get_file?file=/Front%20yard/2020-09-29%2010.51.26%2043sec%20motion.mp4.jpg&token=d6bdc406225bf6bb6ecf720e97e8927acfe52022
```

## Get status
Access: admin

GET `/api/v1/get_status?token=<string>&cameraId=<number>`

* `cameraId` (optional) - camera ID returned by `/api/v1/get_cam_list` request. If empty, global status returned.

Example:
```
https://192.168.0.3/api/v1/get_status?token=d6bdc406225bf6bb6ecf720e97e8927acfe52022
https://192.168.0.3/api/v1/get_status?token=d6bdc406225bf6bb6ecf720e97e8927acfe52022&cameraId=182399567
```

Global JSON payload:
```
{
	"data": {
		"backgroundMode": true,
		"cpuUsagePercents": 0,
		"cpuFrequencyMhz": 2014,
		"temperature": -1,
		"memoryAvailable": 1089949696,
		"memoryUsed": 490082304,
		"threadsUsed": 73,
		"threadsRunnableUsed": 20,
		"processes": [{
			"name": "UI",
			"memoryUsed": 38047744
		}, {
			"name": "Watchdog",
			"memoryUsed": 29640704
		}, {
			"name": "WebServer",
			"memoryUsed": 231104512
		}, {
			"name": "Background",
			"memoryUsed": 193558528
		}],
		"batteryLevel": 100,
		"batteryStatus": "charging",
		"uptime": 23160200,
		"networkInBps": 76103,
		"networkOutBps": 0,
		"streamProfile": 0,
		"powerSafeMode": false,
		"notifications": true,
		"rootAvailable": false,
		"spaceUsed": 4094579744,
		"spaceAvailable": 1849597952,
		"liveConnections": 1,
		"motionCameras": ["Front yard"]
	}
}
```

Camera JSON payload:
```
{
	"data": {
		"motion": true
	}
}
```

## Get MJPEG and JPEG video streams
Access: guest, admin

GET `/axis-cgi/mjpg/video.cgi`

Used to request a Motion JPEG video stream with specified arguments.

GET `/axis-cgi/jpg/image.cgi`

Request for single JPEG.

```
http://<servername>/axis-cgi/mjpg/video.cgi[?<argument>=<value>[&<argument>=<value>...]]
http://<servername>/axis-cgi/jpg/image.cgi[?<argument>=<value>[&<argument>=<value>...]]
```

Parameters:
* `camera=<int>` (optional) - select video source. 1..n.
* `fps=<int>` (optional) - image frame rate. 0 - unlimited.
* `compression=<int>` (optional) - adjust image compression level. Higher values correspond to higher compression, that is lower quality and smaller image size.
* `resolution=<string>` (optional) - resolution [width]x[height] of the returned image, e.g. `640x480`.


Examples:
```
http://192.168.0.3:8083/axis-cgi/mjpg/video.cgi
http://192.168.0.3:8083/axis-cgi/jpg/image.cgi
http://192.168.0.3:8083/axis-cgi/mjpg/video.cgi?camera=2&fps=1&compression=80&resolution=320x240
http://192.168.0.3:8083/axis-cgi/jpg/image.cgi?camera=3&compression=50&resolution=480x640
http://pastebin.com/NCWWSQxa (matrix 2x2)
```

## Get Audio stream
Access: admin

GET `/axis-cgi/audio/receive.wav`

is used to request a WAV audio stream with specified arguments.
 `http://<servername>/axis-cgi/audio/receive.wav[?<argument>=<value>[&<argument>=<value>...]]`

Parameters:
* `camera=<int>` (optional) - select audio source. 1..n.
* `cameraId` (optional) - camera ID

Examples:
```
http://192.168.0.3:8083/axis-cgi/audio/receive.wav
http://192.168.0.3:8083/axis-cgi/audio/receive.wav?camera=2
```

## PTZ control
Access: admin

GET `/axis-cgi/com/ptz.cgi`

Parameters:
* `camera=<int>` (optional) - select video source. 1..n.
* `continuouspantiltmove=<int>,<int>` (optional) - continuous pan/tilt motion. Positive values mean right (pan) and up (tilt), negative values mean left (pan) and down (tilt). "0,0" means stop. Values as <pan speed>,<tilt speed>.
* `continuouszoommove=<int>` (optional) - continuous zoom motion. Positive values mean zoom in and negative values mean zoom out. "0" means stop.
* `continuousfocusmmove=<int>` (optional) - continuous focus motion. Positive values focus near and negative values mean focus far. "0" means stop.
* `continuousirismmove=<int>` (optional) - continuous iris motion. Positive values mean iris open and negative values mean iris close. "0" means stop.
* `move=home` (optional) - moves home.
* `gotoserverpresetno=<int>` (optional) - move to the position associated with the specified preset position number. 1..n.

Examples:
```
http://192.168.0.3:8083/axis-cgi/com/ptz.cgi?move=home
http://192.168.0.3:8083/axis-cgi/com/ptz.cgi?camera=2&gotoserverpresetno=3
http://192.168.0.3:8083/axis-cgi/com/ptz.cgi?continuouspantiltmove=0,0
http://192.168.0.3:8083/axis-cgi/com/ptz.cgi?continuouspantiltmove=100,-100&continuouszoommove=100
```

## Background mode
Access: admin.

The `root.BackgroundMode` parameter is used to switch on/off background mode in tinyCam Monitor.

GET `http://<servername>/param.cgi?action=update&root.BackgroundMode=<string>`

Parameters:
* `root.BackgroundMode=<string>` (mandatory) - can be `on` or `off`.


Examples:
```
http://192.168.0.3:8083/param.cgi?action=update&root.BackgroundMode=on
http://192.168.0.3:8083/param.cgi?action=update&root.BackgroundMode=off
```

## Stream profile
Access: admin

The `root.StreamProfile` parameter is used to change stream profile in tinyCam Monitor.

GET `http://<servername>/param.cgi?action=update&root.StreamProfile=<string>`

Parameters:
* `root.StreamProfile=<string>` (mandatory) - can be `main`, `sub` or `auto`.


Examples:
```
http://192.168.0.3:8083/param.cgi?action=update&root.StreamProfile=main
http://192.168.0.3:8083/param.cgi?action=update&root.StreamProfile=auto
```

## Notificaions
Access: admin

The `root.Notifications` parameter is used to switch on/off notifications in tinyCam Monitor.

GET `http://<servername>/param.cgi?action=update&root.Notifications=<string>[&tag=<string>]`

Parameters:
* `root.Notifications=<string>` (mandatory) - can be `on` or `off`.
* `tag=<string>` (optional) - tag name. If specified the app sends "Motion Detection On" or "Motion Detection Off" command to all cameras under the tag for changing on-camera motion detection.


Examples:
```
http://192.168.0.3:8083/param.cgi?action=update&root.Notifications=on
http://192.168.0.3:8083/param.cgi?action=update&root.Notifications=on&tag=Office
http://192.168.0.3:8083/param.cgi?action=update&root.Notifications=off
http://192.168.0.3:8083/param.cgi?action=update&root.Notifications=off&tag=Home
```

## Power safe mode
Access: admin only

The `root.PowerSafeMode` parameter is used to switch on/off power safe mode in tinyCam Monitor.

GET `http://<servername>/param.cgi?action=update&root.PowerSafeMode=<string>`

Parameters:
* `root.PowerSafeMode=<string>` (mandatory) - can be `on` or `off`.


Examples:
```
http://192.168.0.3:8083/param.cgi?action=update&root.PowerSafeMode=on
http://192.168.0.3:8083/param.cgi?action=update&root.PowerSafeMode=off
```

## Delete file
Access: admin 

GET `http://<servername>/param.cgi?action=delete&root.Filename=<string>`

`action=delete` with `root.Filename` parameter used to delete recorded MP4 or JPEG files in tinyCam Monitor.


Parameters:
* `root.Filename=<string>` (mandatory) - filename.


Examples:
```
http://192.168.0.3:8083/param.cgi?action=delete&root.Filename=/IPV68P2P/2016-10-21%2010.03.33.mp4
```

## Pin file
Access: admin

`action=pin` with `root.Filename` parameter used to pin recorded MP4 or JPEG files in tinyCam Monitor.

GET `http://<servername>/param.cgi?action=pin&root.Filename=<string>`

Parameters:
* `root.Filename=<string>` (mandatory) - filename.

Examples:
```
http://192.168.0.3:8083/param.cgi?action=pin&root.Filename=/IPV68P2P/2016-10-21%2010.03.33.mp4
```

## Unpin file
Access: admin

`action=unpin` with `root.Filename` parameter used to unpin recorded MP4 or JPEG files in tinyCam Monitor.
 `http://<servername>/param.cgi?action=unpin&root.Filename=<string>`

Parameters:
* `root.Filename=<string>` (mandatory) - filename.

Examples:
```
http://192.168.0.3:8083/param.cgi?action=unpin&root.Filename=/IPV68P2P/2016-10-21%2010.03.33_pin.mp4
```

## Reboot Android device
Access: admin. Root required.

GET `http://<servername>/axis-cgi/admin/restart.cgi`

Examples:
```
http://192.168.0.3:8083/axis-cgi/admin/restart.cgi
```
