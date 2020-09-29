# tinyCam Monitor PRO web server API
This is official documentation of the REST API provided by <a href="https://tinycammonitor.com/">tinyCam Monitor PRO</a> web server - Android app for video surveillance.
tinyCam Monitor PRO web server uses API which is partially compatible with <a href="https://www.axis.com/files/manuals/HTTP_API_VAPIX_2.pdf">Axis IP cameras</a>. You can use any IP camera viewer software to view tinyCam server remotely.

## Authentication
tinyCam supports three types of authentication:
### 1. <a href="https://en.wikipedia.org/wiki/Basic_access_authentication">Basic authentication</a> (legacy)

Example:
```
https://admin:mypassword@192.168.0.3:8083/axis-cgi/mjpg/video.cgi
```


### 2. Providing credentials as parameters
Use it when there is no way to use basic authentication (e.g. for home automation).

GET parameters:
```
user=<string>
Username. Optional.
```

```
pwd=<string>
Password. Optional.
```

Example:
```
https://192.168.0.3:8083/axis-cgi/mjpg/video.cgi?user=admin&pwd=mypassword
```


### 3. <b>Token based authentication</b> (recommended to use)

`GET /api/v1/login?user=<string>&pwd=<string>`

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
