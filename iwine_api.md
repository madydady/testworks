# Overview

The iWine is a "smart" decanter for wine. The decanter provides some usefull information about the wine stored in it. It can identify the type of wine, measure the volume of poured wine, alcohol percentage, sweetness and the wine's current temperature. With the iWine decanter you can also heat or cold wine remotly as well as shake it to fill with air. 

# iWine API

iWine works as an HTTP server and provides an API with the following methods:

* (GET) volume - returns the volume of the decanter; 
* (GET) alcohol - returns the alcohol percentage for wine in the decanter;
* (GET) sugar - returns the sweetness of wine in the decanter;
* (GET) winetype - returns the type of wine in the decanter;
* (GET\POST) tempareture - returns the current wine temperature or sets the temperature to the desired degree;
* (POST) shake - makes the decanter to shake wine and fill it with air.

**Note** There are also two callback requests to inform, when the wine reaches desired temperature and is filled with air after shaking. These requests are performed by iWine server as a response to *temperature* and *shake* requests. Processing of these callback requests shuold be implemented at client's side.

## Host 

Connected to your network iWine server works at http://192.168.1.1:8080.
Basic URL for all API methods is http://192.168.1.1:8080/service/. The endpoints for each method are described below.

## Authentication

No authentication required.

## Request parameters

Most API methods work as GET-requests without parameters. Only 'temperature' and 'shake' methods have POST-requests with webhooks for callback requests passed in the request body.  

## Response body

Server returns response data in JSON format. For example:

	HTTP/1.1 200 OK
	Content-Type: application/json; charset=utf-8
    
	{
		"type": "dry",
		"accuracy": 0.8
	}

# Methods

## Volume of wine

### Request

A GET-request to the 'volume' endpoint returns the current volume of wine stored in the decanter. 

URL: http://192.168.1.1:8080/service/volume

Request has no parameters.

### Responses

| Code | Description |
| ---- | ----------- |
| 200 OK | Response object as a JSON (see below) | 
| 405 Method Not Allowed | Error in request name. Correct the URL and try again |
| 413 Payload Too Large | The decanter is overload. Pour out some wine |

Response object is specified below

	{
		"total": 0.75,
		"used": 0.5,
		"free": 0.25
	}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| total | number | yes | Total available volume of the decanter (litres) |
| used | number | yes | Volume of wine (litres) currently poured in the decanter |
| free | number | yes | Free space (litres) available in the decanter |

## Alcohol in wine

### Request

A GET-request to the 'alcohol' endpoint returns the alcohol percentage of wine stored in the decanter.  

URL: http://192.168.1.1:8080/service/alcohol

Request has no parameters.

### Responses

| Code | Description |
| ---- | ----------- |
| 200 OK | Response object as a JSON (see below) |
| 204 No Content | Alcohol-free liquid in decanter | 
| 405 Method Not Allowed | Error in request name. Correct the URL and try again |

Response object is specified below

	{"alcohol": 11 }

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| alcohol | number | yes | The alcohol percentage (number of percents) of wine |

## Sweetness of wine

### Request

A GET-request to the 'sugar' endpoint returns the sweetness (gram of sugar per cubic decimeter of wine) and a sort of wine (sweet, dry, etc.) in the decanter.

URL: http://192.168.1.1:8080/service/sugar

Request has no parameters.

### Responses

| Code | Description |
| ---- | ----------- |
| 200 OK | Response object as a JSON (see below) |
| 204 No Content | Sugar-free liquid in decanter | 
| 405 Method Not Allowed | Error in request name. Correct the URL and try again |

Response object is specified below

	{
		"sugar":0.25
		"sort":"dry"
	}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| sugar | number | yes | The sweetness of wine (gram of sugar per cubic decimeter) |
| sort | string | yes | Sort of wine. One of the following: "dry", "semidry", "sweet", "semisweet" | 

## Wine temperature 

### GET-request

GET-request to the 'temperature' endpoint returns current temperature of wine in the decanter. 

URL: http://192.168.1.1:8080/service/temperature

Request has no parameters.

### Responses

| Code | Description |
| ---- | ----------- |
| 200 OK | Response object as a JSON (see below) | 
| 405 Method Not Allowed | Error in request name. Correct the URL and try again |
| 503 Service Unavailable | Temperature can't be measured. Check the thermal measuring system | 

Response object is specified below

	{"temp": 10 }

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| temp | number | yes | The current temperature of wine (in celcius) |

### POST-request

POST-request to the 'tempareture' endpoint with a degree number and a callback function passed in request body is used to set the wine temperature to the specified degree (in the range -5...+20 in celsius). Approximate time needed to reach the required temperature returns in the result body. To get informed when the temperature reaches the required value the callback request is used. Handling of such a request should be implemented at client's side. 

URL: http://192.168.1.1:8080/service/temperature

#### Request body

Request body contains a JSON with required tempareture and URL to callback function (webhook) with optional authorisation parameters, if server needs it.

	{
		"degree": 10,

		"callback": {
			"webhook": "URL to callback function",
			"auth": true,
			"login": "user",
			"password": "pass"
		{
	}

#### Parameters

Request parameters are specified below

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| degree | number | yes | Degree number (in celsius) that the wine temperature should be brought to |
| callback | object | yes | Parameters for callback request |
| &nbsp; webhook | string | yes | URL to the callback function |
| &nbsp; auth | boolean | yes | true if request needs authorisation at client's side, false in another case |
| &nbsp; login | string | no | login to authorise at clienr's side if required |
| &nbsp; password | string | no | password to authorise at clienr's side if required |

### Responses

| Code | Description |
| ---- | ----------- |
| 200 OK | Response object as a JSON (see below) | 
| 400 Bad Request | Errors in request parameters. Check that "degree" is a number and "webhook" is a valid URL. Check "login" and "password", if "auth" is set to "true" | 
| 405 Method Not Allowed | Error in request name. Correct the URL and try again |
| 416 Range Not Satisfiable | The specified temperature is out of range -5...+20 in celsius and can't be reached |
| 503 Service Unavailable | Temperature can't be measured. Check the thermal measuring system | 

Response object is specified below

	{"apprTime": 5 }

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| apprTime | number | yes | Approximate time (in minutes) needed to set wine temperature |

Callback request will be executed by iWine server, when temperature reaches the desired level. 

Request body contatins a JSON with operation status.

	{
		"status": "OK",
		"message": "The wine temperature is now N degrees",
	}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| status | string | yes | "OK" if the wine temperature reached specified degree, "Fail" if not - details in *message* field |
| message | string | yes | Text "The wine temperature is now N degrees", where *N* is the *degree* parameter in initial request. Or error details, if the specified temperature can't be reached |

## Type of wine

### Request

A GET-request to the 'winetype' endpoint returns the type of wine (red, white, rose, etc.) in the decanter and accuracy coefficient. The type is predicted using machine learning arlgorithms and is defined with some accuracy coefficient.

URL: http://192.168.1.1:8080/service/winetype

### Responses

| Code | Description |
| ---- | ----------- |
| 200 OK | Response object as a JSON (see below) | 
| 405 Method Not Allowed | Error in request name. Correct the URL and try again |
| 409 Conflict | Type of wine is undefined | 
| 503 Service Unavailable | Service is unavailable. Contact the developer | 

Response object is specified below

	{
		"type": "dry",
		"accuracy": 0.8
	}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| type | string | yes | Type of wine. One of the following values: red, white, rose |
| accuracy | number | yes | Accuracy coefficient for predicted wine type, a number in range (0;1) |

## Shaking wine

When the iWine server gets shake request it begins to shake the decanter to fill the wine with air. The time required for this operation depends on current volume of wine in the decanter. Approximate time is calculated and the server returns it in the result body. To get informed when the process is over a callback request is made.

### Request

POST-request to the 'shake' endpoint with 'callback' parameter in the request body

URL: http://192.168.1.1:8080/service/shake

Request body contatins a JSON with URL to callback function (webhook) with optional authorisation parameters, if server needs it.

	{
		"callback": {
			"webhook": "URL to callback function",
			"auth": true,
			"login": "user",
			"password": "pass"
		{
	}

#### Parameters

Request parameters are specified below

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| callback | object | yes | Parameters for callback request |
| &nbsp;&nbsp; webhook | string | yes | URL to the callback function |
| &nbsp;&nbsp; auth | boolean | yes | "true" if request needs authorisation at client's side, "false" if not |
| &nbsp;&nbsp; login | string | no | Login to authorise at clienr's side if required |
| &nbsp;&nbsp; password | string | no | Password to authorise at clienr's side if required |

### Responses

| Code | Description |
| ---- | ----------- |
| 200 OK | Response object as a JSON (see below) | 
| 405 Method Not Allowed | Error in request name. Correct the URL and try again |
| 409 Conflict | Decanter is caped, the wine can't be filled with air. Take the cap off and try again | 
| 503 Service Unavailable | Shaking mechanism is out of order. Contact the developer | 

Response object is specified below

	{"apprTime": 5 }

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| apprTime | number | yes | Approximate time (in minutes) needed to fill wine with air |

Callback request will be made by iWine server when shaking is over and the wine is filled with air. 

Request body contatins a JSON with operation status.

	{
		"status": "OK",
		"message": "The wine is filled with air"
	}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| satus | string | yes | "OK" if the wine is successfully filled with air, "Fail" if not - details in *message* field |
| message | string | yes | Text of message with the results of operation, or error details |
