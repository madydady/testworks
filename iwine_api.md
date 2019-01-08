# Overview

The iWine is a "smart" decanter for wine. The decanter provides some usefull information about the wine stored in it. It can identify the type of wine, measure the volume of poured wine, alcohol percentage, sweetness and the wine's current temperature. With the iWine decanter you can also heat or cold wine remotly as well as shake it to fill with air. 

# iWine API

iWine works as an HTTP server and provides an API with the following methods:

* (GET) volume - returns the volume of the decanter; 
* (GET) alcohol - returns the alcohol percentage of wine in the decanter;
* (GET) sugar - returns the sweetness of wine in the decanter;
* (GET) winetype - returns the type of wine in the decanter;
* (GET\POST) tempareture - returns the current wine temperature or sets the temperature to the desired degree;
* (POST) shake - makes the decanter to shake wine and fill it with air.

**Note** There are also two callback requests to inform, when the wine reaches desired temperature and is filled with air after shaking. These requests are performed by iWine server as a response to *temperature* and *shake* requests. Processing of these callback requests shuold be implemented at client's side.

## Host 

Connected to your network iWine server works at http://iwine.com:8080. 

## Authentication

No authentication required.

## Basic URL

Basic URL for all API methods is http://iwine.com:8080/service/. The endpoints for each method are described below.  

## Request parameters

Most API methods work as GET-requests without parameters. Only 'temperature' and 'shake' are POST-requests with callback methods URL passed in the request body.  

## Response body

Server returns response data in JSON format. For example:

    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    
    {"volume":0.75}

# Methods

## Volume of wine

### Request

A GET-request to the 'volume' endpoint returns the current volume of wine stored in the decanter. 

URL: http://iwine.com:8080/service/volume

Request has no parameters.

### Response

#### 200 OK

Response object is specified below

	{
    		"total": 0.75,
		"used": 0.5,
		"free": 0.25
	}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| total | number | yes | Total volume of the decanter (litres) |
| used | number | yes | Volume of wine (litres) currently stored in the decanter |
| free | number | yes | Free space (litres) available in the decanter |


#### 404 Not available

404 response code is returned when server is not available. Check your internet connection.   



## Alcohol in wine

### Request

A GET-request to the 'alcohol' endpoint returns the alcohol percentage of wine stored in the decanter.  

URL: http://iwine.com:8080/service/alcohol

Request has no parameters.

### Response

#### 200 OK

Response object is specified below

	{"alcohol": 11}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| alcohol | number | yes | The alcohol percentage (number of percents) of wine |

## Sweetness of wine

### Request

A GET-request to the 'sugar' endpoint returns the sweetness (how much shugar) and a sort of wine (sweet, dry, etc.) in the decanter.

URL: http://iwine.com:8080/service/sugar

Request has no parameters.

### Response

#### 200 OK

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

GET-request to the 'tempareture' endpoint returns current temperature of wine in the decanter. 

URL: http://iwine.com:8080/service/temperature

Request has no parameters.

### Response

#### 200 OK

Response object is specified below

	{"temp": 10 }

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| temp | number | yes | The current temperature of wine (in Celcius) |

### POST-request

POST-request to the 'tempareture' endpoint with a degree number and a callback function passed in request body is used to set the wine temperature to the specified degree. Approximate time needed to reach the required temperature returns in the result body. To get informed when the temperature reaches the required value the callback request is used. Handling of such a request should be implemented at client's side. 

URL: http://iwine.com:8080/service/temperature

#### Request body

Request body contatins a JSON with required tempareture and URL to callback function (with optional authorisation parameters).

	{
		"degree":20,

		"callback": {
			"URL": "<URL to callback function>",
			"auth": true,
			"login": "user",
			"password": "pass"
		{
	}

#### Parameters

Request parameters are specified below

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| degree | number | yes | Degree number (in Celcius) that the wine temperature should be brought to |
| callback | object | yes | Parameters for callback request |
| &nbsp; URL | string | yes | URL to the callback function |
| &nbsp; auth | boolean | yes | true if request needs authorisation at client's side, false in another case |
| &nbsp; login | string | no | login to authorise at clienr's side if required |
| &nbsp; password | string | no | password to authorise at clienr's side if required |

### Response

#### 200 OK

Response object is specified below

	{"apprTime": 5}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| apprTime | number | yes | Approximate time needed to set wine temperature |

Callback request will be made by iWine server when temperature reaches the desired level. 

Request body contatins a JSON with operation status.

	{
		"status": "OK",
		"message": "The wine temperature is now N degrees",
	}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| satus | string | yes | "OK" if the wine temperature reaches the desired, "Fail" if not - details in *message* field |
| temperature | string | yes | Text "The wine temperature is now N degrees", where *N* is the *degree* parameter in initial request. Or error details, if the desired temperature can't be reached |

## Type of wine

### Request

A GET-request to the 'winetype' endpoint returns the type of wine (red, white, rose, etc.) in the decanter and accuracy coefficient. The type is predicted using machine learning arlgorithms and is defined with some accuracy coefficient.

URL: http://iwine.com:8080/service/winetype

### Response

#### 200 OK

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

URL: http://iwine.com:8080/service/shake

Request body contatins a JSON with URL to callback function (with optional authorisation parameters).

	{
		"callback": {
			"URL": "<URL to callback function>",
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
| &nbsp;&nbsp; URL | string | yes | URL to the callback function |
| &nbsp;&nbsp; auth | boolean | yes | "true" if request needs authorisation at client's side, "false" in another case |
| &nbsp;&nbsp; login | string | no | Login to authorise at clienr's side if required |
| &nbsp;&nbsp; password | string | no | Password to authorise at clienr's side if required |

### Response

#### 200 OK

Response object is specified below

	{"apprTime": 5}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| apprTime | number | yes | Approximate time needed to fill wine with air |

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
