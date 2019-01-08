Графин умеет сообщать на мобильное устройство следующую информацию о налитом в него вине:
объем;
содержание алкоголя;
содержание сахара;
температура;
тип вина — красное, белое, розовое, оранжевое (эта информация определяется с использованием алгоритмов машинного обучения и сопровождается некоторым коэффициентом достоверности).

С помощью мобильного устройства с графином можно производить следующие операции:
охладить или нагреть вино до определенной температуры;
взболтать вино для насыщения его воздухом.

# Overview

The iWine is a "smart" decanter for wine. The decanter provides some usefull information about the wine stored in it. It can recognise the type of wine, measure the volume of poured wine, alcohol percentage, sweetness and the wine's current temperature. With the iWine decanter you can also heat or cold wine remotly as well as shake it to fill with air. 

# iWine API

iWine works as an HTTP server and provides an API with the following methods:

* (GET) volume
* (GET) alcohol
* (GET) sugar
* (GET) tempareture
* (GET) winetype
* (GET) shake

## Host 

Connected to your network iWine server works at http://iwine.com:8080. 

## Authentication

No authentication required

## Basic URL

Basic URL for all API methods is 'http://iwine.com:8080/service/'. The endpoints are described below for each method.  

## Request parameters

API methods work as GET-requests without parameters. Only 'temperture' request has one optional parameter passed in the request line:

'http://iwine.com:8080/service/temperature/{?degree}'

## Response body

Every response body contatins a JSON with data returned by server. For example:

... code

    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    
    {"volume":0.75}

# Methods

## Volume of wine

### Request

A GET-request to the 'volume' endpoint returns the current volume of wine stored in the decanter. Request has no parameters. 

Example: http://iwine.com:8080/service/volume

### Response

#### 200 OK

Response object is specified below

... code

    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    
    {"volume":0.75}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| volume | number | yes | The volume of wine (litres) currently stored in decanter |

#### 404 Not available

404 response code is returned when server is not available. Check your internet connection.   



## Alcohol in wine

### Request

A GET-request to the 'alcohol' endpoint returns the alcohol percentage of wine stored in the decanter. Request has no parameters. 

Example: http://iwine.com:8080/service/alcohol

### Response

#### 200 OK

Response object is specified below

... code

    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    
    {"alcohol":11}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| alcohol | number | yes | The alcohol percentage (number of percents) of wine |

## Sweetness of wine

### Request

A GET-request to the 'sugar' endpoint returns the sweetness (how much shugar) and a sort of wine (sweet, dry, etc.) in the decanter. Request has no parameters.

Example: http://iwine.com:8080/service/sugar

### Response

#### 200 OK

Response object is specified below

... code

    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    
    {"sugar":0.25,
	 "sort":"dry"
	}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| sugar | number | yes | The sweetness of wine (gram of sugar per cubic decimeter) |
| sort | string | yes | Sort of wine. One of the following: "dry", "semidry", "sweet", "semisweet" | 


## Wine temperature 

GET-request to the 'tempareture' endpoint returns current temperature of wine in the decanter. POST-request with a degree number and a callback function passed in request body is used to set the wine temperature to the specified degree. Approximate time needed to reach the required temperature value returns in the result body. To get informed when the temperature reaches the required value the callback request is used. Handling of such a request should be implemented at client's side.  

### GET-request

GET-request to the 'tempareture' endpoint: 

Example: http://iwine.com:8080/service/temperature

### Response

#### 200 OK

Response object is specified below

... code

    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    
    {"sugar":0.25,
	 "sort":"dry"
	}

| Name | Type | Mandatory | Description |
| ---- | ---- | --------- | ----------- |
| sugar | number | yes | The sweetness of wine (gram of sugar per cubic decimeter) |
| sort | string | yes | Sort of wine. One of the following: "dry", "semidry", "sweet", "semisweet" | 

### POST-request

POST-request to the 'tempareture' endpoint with 'degree' and 'callback' parameters in the request body

Example: http://iwine.com:8080/service/temperature

POST-request body

Request body contatins a JSON with required tempareture and URL to callback function (with optional authorisation parameters).

... code

    HTTP/1.1 http://iwine.com:8080/service/temperature
    Content-Type: application/json; charset=utf-8
    
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

... code

    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    
    {"apprTime": 5}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| apprTime | number | yes | Approximate time needed to set wine temperature |


## Type of wine

### Request

A GET-request to the 'winetype' endpoint returns the type of wine (red, white, rose, etc.) in the decanter and accuracy coefficient. The type is predicted using machine learning arlgorithms and is defined with some accuracy coefficient.

Example: http://iwine.com:8080/service/winetype

### Response

#### 200 OK

Response object is specified below

... code

    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    
    {"type": "dry",
	 "accuracy": 0.8
	}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| type | string | yes | Type of wine. One of the following values: red, white, rose |
| accuracy | number | yes | Accuracy coefficient for predicted wine type, a number in range (0;1) |


## Shaking wine

When the iWine server gets shake request it begins to shake the decanter to fill the wine with air. The time required for this operation depends on current volume of wine in the decanter. Approximate time is calculated and the server returns it in the result body. To get informed when the process is over a callback request is made.
URL: http://iwine.com:8080/service/shake

### Request

POST-request to the 'shake' endpoint with 'callback' parameter in the request body

Example: http://iwine.com:8080/service/shake

Request body contatins a JSON with URL to callback function (with optional authorisation parameters).

... code

    HTTP/1.1 http://iwine.com:8080/service/shake
    Content-Type: application/json; charset=utf-8
    
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
| &nbsp; URL | string | yes | URL to the callback function |
| &nbsp; auth | boolean | yes | "true" if request needs authorisation at client's side, "false" in another case |
| &nbsp; login | string | no | Login to authorise at clienr's side if required |
| &nbsp; password | string | no | Password to authorise at clienr's side if required |

### Response

#### 200 OK

Response object is specified below

... code

    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    
    {"apprTime": 5}

| Name | Type | Required | Description |
| ---- | ---- | --------- | ----------- |
| apprTime | number | yes | Approximate time needed to fill wine with air |
