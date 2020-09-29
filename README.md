# tinyCam Monitor PRO web server API
This is official documentation of the REST API provided by <a href="https://tinycammonitor.com/">tinyCam Monitor PRO</a> web server - Android app for video surveillance.
tinyCam web server uses API which is partially compatible with <a href="https://www.axis.com/files/manuals/HTTP_API_VAPIX_2.pdf">Axis IP cameras</a>. You can use any IP camera viewer software to view tinyCam server remotely.

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

GET `/api/v1/logout?token=<string>`

Deletes authentication token.

Example:
```
https://192.168.0.3:8083/api/v1/logout?token=d6bdc406225bf6bb6ecf720e97e8927acfe52022
```

## Get camera list

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

GET `/api/v1/get_cam_event_list?token=<string>&cameraId=<number>&endtime=<number>&count=<number>&type=<string>&motion=<string>`
* `cameraId` (mandatory) - camera ID returned by `/api/v1/get_cam_list` request
* `endtime` (optional) - time (Unix timestamp in milliseconds) from which events should be returned
* `count` (optional) - max number of events to get
* `type` (optional) - `local` (default) or `cloud` events
* `motion` (optional) - can be `face`, `person`, `vehicle`, `pet`, `motion`, or `audio`. If empty, all events returned.

List is returned in descending order (new events first).

Example:
```
 https://10.0.1.90:8083/api/v1/get_cam_event_list?token=d6bdc406225bf6bb6ecf720e97e8927acfe52022&cameraId=182399567&endtime=1601381164000&count=15&type=local&motion=person
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

