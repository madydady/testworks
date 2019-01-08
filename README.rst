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

Basic URL for all API methods is 'http://iwine.com:8080/service/'. The endpoints for each method are described below.  

## Request parameters

Most API methods work as GET-requests without parameters. Only 'temperature' and 'shake' are POST-requests with callback methods URL's passed in the request body.  

## Response body

Every response body contatins a JSON with data returned by server. For example:

    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    
    {"volume":0.75}

# Methods

## Volume of wine

### Request
