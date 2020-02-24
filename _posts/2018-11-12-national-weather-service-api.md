---
layout: post
title: National Weather Service API
tags: [API]
comments: true
---

Recently I developed a web app that retrieves the forecast based on your location.  It uses your latitude and longitude to retrieve weather data using the National Weather Service APIs that are available here [Weather API](https://www.weather.gov/documentation/services-web-api).

The National Weather Service APIs are free of charge, and the data that is retrieved is part of what we see when we track storms and other weather using NOAA.  The API enables you to retrieve the data and use it in your code rather than just relying on pages like [www.weather.gov](https://www.weather.gov/)

I wanted to write about this to show how this works, and how you can leverage this service in an application.  I’m going to walk through getting a local forecast, starting with current weather and then moving onto a weekly forecast.

## Start with the metadata endpoint

To start with, all the endpoints use the API base [api.weather.gov](https://api.weather.gov)

The basic endpoints are all extensions of that original API base to include latitude and longitude values.  You can get those in your code using location services (checkout my post on location services here

For this walkthrough, we’re going to get the local weather for Richmond, Va.  We’re going to use the following location:

* latitude = 37.540726
* longitude = -77.436050

If you want to know your current location you can google it, or checkout this easy [site](https://www.latlong.net/)

Lets get the metadata for that location using the metadata endpoint:

```json
https://api.weather.gov/points/{<latitude>,<longitude>}
```

So for the Richmond location, this would look like:

```json
https://api.weather.gov/points/37.540726,-77.436050
```

Using postman, this output looks like the following:

```json
{
    "@context": [
        "https://raw.githubusercontent.com/geojson/geojson-ld/master/contexts/geojson-base.jsonld",
        {
            "wx": "https://api.weather.gov/ontology#",
            "s": "https://schema.org/",
            "geo": "http://www.opengis.net/ont/geosparql#",
            "unit": "http://codes.wmo.int/common/unit/",
            "@vocab": "https://api.weather.gov/ontology#",
            "geometry": {
                "@id": "s:GeoCoordinates",
                "@type": "geo:wktLiteral"
            },
            "city": "s:addressLocality",
            "state": "s:addressRegion",
            "distance": {
                "@id": "s:Distance",
                "@type": "s:QuantitativeValue"
            },
            "bearing": {
                "@type": "s:QuantitativeValue"
            },
            "value": {
                "@id": "s:value"
            },
            "unitCode": {
                "@id": "s:unitCode",
                "@type": "@id"
            },
            "forecastOffice": {
                "@type": "@id"
            },
            "forecastGridData": {
                "@type": "@id"
            },
            "publicZone": {
                "@type": "@id"
            },
            "county": {
                "@type": "@id"
            }
        }
    ],
    "id": "https://api.weather.gov/points/37.5407,-77.4361",
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [
            -77.436099999999996,
            37.540700000000001
        ]
    },
    "properties": {
        "@id": "https://api.weather.gov/points/37.5407,-77.4361",
        "@type": "wx:Point",
        "cwa": "AKQ",
        "forecastOffice": "https://api.weather.gov/offices/AKQ",
        "gridX": 45,
        "gridY": 76,
        "forecast": "https://api.weather.gov/gridpoints/AKQ/45,76/forecast",
        "forecastHourly": "https://api.weather.gov/gridpoints/AKQ/45,76/forecast/hourly",
        "forecastGridData": "https://api.weather.gov/gridpoints/AKQ/45,76",
        "observationStations": "https://api.weather.gov/gridpoints/AKQ/45,76/stations",
        "relativeLocation": {
            "type": "Feature",
            "geometry": {
                "type": "Point",
                "coordinates": [
                    -77.476009000000005,
                    37.531399
                ]
            },
            "properties": {
                "city": "Richmond",
                "state": "VA",
                "distance": {
                    "value": 3667.7842790420277,
                    "unitCode": "unit:m"
                },
                "bearing": {
                    "value": 73,
                    "unitCode": "unit:degrees_true"
                }
            }
        },
        "forecastZone": "https://api.weather.gov/zones/forecast/VAZ515",
        "county": "https://api.weather.gov/zones/county/VAC760",
        "fireWeatherZone": "https://api.weather.gov/zones/fire/VAZ515",
        "timeZone": "America/New_York",
        "radarStation": "KAKQ"
    }
}
```

Calling the metadata endpoint, provides basic information about the location.  The response also includes additional preloaded URLs based on your location to include the following:

```json
{
  "forecast": "https://api.weather.gov/gridpoints/AKQ/45,76/forecast",
  "forecastHourly": "https://api.weather.gov/gridpoints/AKQ/45,76/forecast/hourly",
  "forecastGridData": "https://api.weather.gov/gridpoints/AKQ/45,76",
  "observationStations": "https://api.weather.gov/gridpoints/AKQ/45,76/stations"
}
```

Calling those endpoints would give you the associated forecast, hourly forecast, grid data, and observations stations.

## Forecast Endpoint

Using the metadata endpoint that we looked at before, one of the preloaded URLs was the following:

```json
https://api.weather.gov/gridpoints/AKQ/45,76/forecast
```

This endpoint provides the forecast for the next few days for the associated region.  Calling the one for Richmond with Postman would result in the following:

```json
{
    "@context": [
        "https://raw.githubusercontent.com/geojson/geojson-ld/master/contexts/geojson-base.jsonld",
        {
            "wx": "https://api.weather.gov/ontology#",
            "geo": "http://www.opengis.net/ont/geosparql#",
            "unit": "http://codes.wmo.int/common/unit/",
            "@vocab": "https://api.weather.gov/ontology#"
        }
    ],
    "type": "Feature",
    "geometry": {
        "type": "GeometryCollection",
        "geometries": [
            {
                "type": "Point",
                "coordinates": [
                    -77.422099799999998,
                    37.540535499999997
                ]
            },
            {
                "type": "Polygon",
                "coordinates": [
                    [
                        [
                            -77.434209100000004,
                            37.553019900000002
                        ],
                        [
                            -77.437838900000003,
                            37.530928600000003
                        ],
                        [
                            -77.409993299999996,
                            37.528049600000003
                        ],
                        [
                            -77.406357799999995,
                            37.550140600000006
                        ],
                        [
                            -77.434209100000004,
                            37.553019900000002
                        ]
                    ]
                ]
            }
        ]
    },
    "properties": {
        "updated": "2018-12-06T07:58:18+00:00",
        "units": "us",
        "forecastGenerator": "BaselineForecastGenerator",
        "generatedAt": "2018-12-06T10:35:46+00:00",
        "updateTime": "2018-12-06T07:58:18+00:00",
        "validTimes": "2018-12-06T01:00:00+00:00/P7DT11H",
        "elevation": {
            "value": 29.8704,
            "unitCode": "unit:m"
        },
        "periods": [
            {
                "number": 1,
                "name": "Overnight",
                "startTime": "2018-12-06T05:00:00-05:00",
                "endTime": "2018-12-06T06:00:00-05:00",
                "isDaytime": false,
                "temperature": 27,
                "temperatureUnit": "F",
                "temperatureTrend": null,
                "windSpeed": "3 mph",
                "windDirection": "W",
                "icon": "https://api.weather.gov/icons/land/night/few?size=medium",
                "shortForecast": "Mostly Clear",
                "detailedForecast": "Mostly clear, with a low around 27. West wind around 3 mph."
            },
            {
                "number": 2,
                "name": "Thursday",
                "startTime": "2018-12-06T06:00:00-05:00",
                "endTime": "2018-12-06T18:00:00-05:00",
                "isDaytime": true,
                "temperature": 46,
                "temperatureUnit": "F",
                "temperatureTrend": null,
                "windSpeed": "3 to 7 mph",
                "windDirection": "SW",
                "icon": "https://api.weather.gov/icons/land/day/few?size=medium",
                "shortForecast": "Sunny",
                "detailedForecast": "Sunny, with a high near 46. Southwest wind 3 to 7 mph."
            },
            {
                "number": 3,
                "name": "Thursday Night",
                "startTime": "2018-12-06T18:00:00-05:00",
                "endTime": "2018-12-07T06:00:00-05:00",
                "isDaytime": false,
                "temperature": 31,
                "temperatureUnit": "F",
                "temperatureTrend": null,
                "windSpeed": "7 mph",
                "windDirection": "SW",
                "icon": "https://api.weather.gov/icons/land/night/bkn?size=medium",
                "shortForecast": "Mostly Cloudy",
                "detailedForecast": "Mostly cloudy, with a low around 31. Southwest wind around 7 mph."
            },
            {
                "number": 4,
                "name": "Friday",
                "startTime": "2018-12-07T06:00:00-05:00",
                "endTime": "2018-12-07T18:00:00-05:00",
                "isDaytime": true,
                "temperature": 47,
                "temperatureUnit": "F",
                "temperatureTrend": null,
                "windSpeed": "7 mph",
                "windDirection": "NW",
                "icon": "https://api.weather.gov/icons/land/day/few?size=medium",
                "shortForecast": "Sunny",
                "detailedForecast": "Sunny, with a high near 47. Northwest wind around 7 mph."
            },
            {
                "number": 5,
                "name": "Friday Night",
                "startTime": "2018-12-07T18:00:00-05:00",
                "endTime": "2018-12-08T06:00:00-05:00",
                "isDaytime": false,
                "temperature": 26,
                "temperatureUnit": "F",
                "temperatureTrend": null,
                "windSpeed": "5 mph",
                "windDirection": "N",
                "icon": "https://api.weather.gov/icons/land/night/few?size=medium",
                "shortForecast": "Mostly Clear",
                "detailedForecast": "Mostly clear, with a low around 26. North wind around 5 mph."
            },
            {
                "number": 6,
                "name": "Saturday",
                "startTime": "2018-12-08T06:00:00-05:00",
                "endTime": "2018-12-08T18:00:00-05:00",
                "isDaytime": true,
                "temperature": 41,
                "temperatureUnit": "F",
                "temperatureTrend": null,
                "windSpeed": "5 mph",
                "windDirection": "N",
                "icon": "https://api.weather.gov/icons/land/day/sct?size=medium",
                "shortForecast": "Mostly Sunny",
                "detailedForecast": "Mostly sunny, with a high near 41."
            },
            {
                "number": 7,
                "name": "Saturday Night",
                "startTime": "2018-12-08T18:00:00-05:00",
                "endTime": "2018-12-09T06:00:00-05:00",
                "isDaytime": false,
                "temperature": 29,
                "temperatureUnit": "F",
                "temperatureTrend": null,
                "windSpeed": "5 mph",
                "windDirection": "N",
                "icon": "https://api.weather.gov/icons/land/night/bkn?size=medium",
                "shortForecast": "Mostly Cloudy",
                "detailedForecast": "Mostly cloudy, with a low around 29."
            },
            {
                "number": 8,
                "name": "Sunday",
                "startTime": "2018-12-09T06:00:00-05:00",
                "endTime": "2018-12-09T18:00:00-05:00",
                "isDaytime": true,
                "temperature": 35,
                "temperatureUnit": "F",
                "temperatureTrend": null,
                "windSpeed": "5 to 9 mph",
                "windDirection": "NE",
                "icon": "https://api.weather.gov/icons/land/day/snow,30/snow,60?size=medium",
                "shortForecast": "Snow Likely",
                "detailedForecast": "Snow likely after 7am. Cloudy, with a high near 35. Chance of precipitation is 60%."
            },
            {
                "number": 9,
                "name": "Sunday Night",
                "startTime": "2018-12-09T18:00:00-05:00",
                "endTime": "2018-12-10T06:00:00-05:00",
                "isDaytime": false,
                "temperature": 32,
                "temperatureUnit": "F",
                "temperatureTrend": null,
                "windSpeed": "12 mph",
                "windDirection": "NE",
                "icon": "https://api.weather.gov/icons/land/night/snow,70?size=medium",
                "shortForecast": "Snow Likely",
                "detailedForecast": "Snow likely before 7pm, then rain and snow likely. Cloudy, with a low around 32. Chance of precipitation is 70%."
            },
            {
                "number": 10,
                "name": "Monday",
                "startTime": "2018-12-10T06:00:00-05:00",
                "endTime": "2018-12-10T18:00:00-05:00",
                "isDaytime": true,
                "temperature": 38,
                "temperatureUnit": "F",
                "temperatureTrend": null,
                "windSpeed": "8 to 12 mph",
                "windDirection": "N",
                "icon": "https://api.weather.gov/icons/land/day/snow,70/snow,50?size=medium",
                "shortForecast": "Snow Likely",
                "detailedForecast": "Snow likely before 1pm, then a chance of rain and snow. Mostly cloudy, with a high near 38. Chance of precipitation is 70%."
            },
            {
                "number": 11,
                "name": "Monday Night",
                "startTime": "2018-12-10T18:00:00-05:00",
                "endTime": "2018-12-11T06:00:00-05:00",
                "isDaytime": false,
                "temperature": 27,
                "temperatureUnit": "F",
                "temperatureTrend": null,
                "windSpeed": "8 mph",
                "windDirection": "N",
                "icon": "https://api.weather.gov/icons/land/night/snow,50/snow,30?size=medium",
                "shortForecast": "Chance Rain And Snow",
                "detailedForecast": "A chance of rain and snow. Mostly cloudy, with a low around 27. Chance of precipitation is 50%."
            },
            {
                "number": 12,
                "name": "Tuesday",
                "startTime": "2018-12-11T06:00:00-05:00",
                "endTime": "2018-12-11T18:00:00-05:00",
                "isDaytime": true,
                "temperature": 43,
                "temperatureUnit": "F",
                "temperatureTrend": null,
                "windSpeed": "8 mph",
                "windDirection": "NW",
                "icon": "https://api.weather.gov/icons/land/day/snow,30/sct?size=medium",
                "shortForecast": "Chance Snow then Mostly Sunny",
                "detailedForecast": "A chance of snow before 7am. Mostly sunny, with a high near 43. Chance of precipitation is 30%."
            },
            {
                "number": 13,
                "name": "Tuesday Night",
                "startTime": "2018-12-11T18:00:00-05:00",
                "endTime": "2018-12-12T06:00:00-05:00",
                "isDaytime": false,
                "temperature": 26,
                "temperatureUnit": "F",
                "temperatureTrend": null,
                "windSpeed": "6 mph",
                "windDirection": "W",
                "icon": "https://api.weather.gov/icons/land/night/few?size=medium",
                "shortForecast": "Mostly Clear",
                "detailedForecast": "Mostly clear, with a low around 26."
            },
            {
                "number": 14,
                "name": "Wednesday",
                "startTime": "2018-12-12T06:00:00-05:00",
                "endTime": "2018-12-12T18:00:00-05:00",
                "isDaytime": true,
                "temperature": 45,
                "temperatureUnit": "F",
                "temperatureTrend": null,
                "windSpeed": "5 mph",
                "windDirection": "W",
                "icon": "https://api.weather.gov/icons/land/day/few?size=medium",
                "shortForecast": "Sunny",
                "detailedForecast": "Sunny, with a high near 45."
            }
        ]
    }
}
```

As you can see from this output, you can map the result and see the detailed forecast for the next several days based on time.  Like for today you can map the forecast of:

```json
{
  "number": 1,
  "name": "Overnight",
  "startTime": "2018-12-06T05:00:00-05:00",
  "endTime": "2018-12-06T06:00:00-05:00",
  "isDaytime": false,
  "temperature": 27,
  "temperatureUnit": "F",
  "temperatureTrend": null,
  "windSpeed": "3 mph",
  "windDirection": "W",
  "icon": "https://api.weather.gov/icons/land/night/few?size=medium",
  "shortForecast": "Mostly Clear",
  "detailedForecast": "Mostly clear, with a low around 27. West wind around 3 mph."
}
```

## Radar stations

Several of the endpoints also require the use of a radar station ID.  This is to map readings from the closest station to your location.  The weather API makes this very easy for you and pre-populates an endpoint in the “metadata” response we looked at earlier.  From the Richmond, Va example the observationsStations pre-populated endpoint was the following:

```json
https://api.weather.gov/gridpoints/AKQ/45,76/stations
```
If you call this endpoint, you end up with a long list of stations and their relative location.  To make this post a little shorter and to spare you this long list, I just copied and pasted one of them here:

```json
{
  "id": "https://api.weather.gov/stations/KRIC",
  "type": "Feature",
  "geometry": {
      "type": "Point",
      "coordinates": [
          -77.323329999999999,
          37.511110000000002
      ]
  }
}
```

The above is for the station with the ID of “KRIC” which is located at -77.3233 and 37.5111 respectively.   You can look through the list of radar stations and find the radar station with the nearest coordinates to your coordinates, and then use that to populate a forecast and hourly forecast link for more data.

## Observations endpoint with radar station id

In the above example, we noted the KRIC location as an example station that was returned from the radar stations endpoint.  If we wanted to use the KRIC location to get the latest observations (current temperature, wind, etc.) then you could use either the current observations endpoint:

```json
https://api.weather.gov/stations/<station id>/observations
```

or the latest observations with:

```json
https://api.weather.gov/stations/<station id>/observations/latest
```

Using the `observations` endpoint for KRIC the resulting output would look like the following:

```json
{
    "@context": [
        "https://raw.githubusercontent.com/geojson/geojson-ld/master/contexts/geojson-base.jsonld",
        {
            "wx": "https://api.weather.gov/ontology#",
            "s": "https://schema.org/",
            "geo": "http://www.opengis.net/ont/geosparql#",
            "unit": "http://codes.wmo.int/common/unit/",
            "@vocab": "https://api.weather.gov/ontology#",
            "geometry": {
                "@id": "s:GeoCoordinates",
                "@type": "geo:wktLiteral"
            },
            "city": "s:addressLocality",
            "state": "s:addressRegion",
            "distance": {
                "@id": "s:Distance",
                "@type": "s:QuantitativeValue"
            },
            "bearing": {
                "@type": "s:QuantitativeValue"
            },
            "value": {
                "@id": "s:value"
            },
            "unitCode": {
                "@id": "s:unitCode",
                "@type": "@id"
            },
            "forecastOffice": {
                "@type": "@id"
            },
            "forecastGridData": {
                "@type": "@id"
            },
            "publicZone": {
                "@type": "@id"
            },
            "county": {
                "@type": "@id"
            }
        }
    ],
    "id": "https://api.weather.gov/stations/KRIC/observations/2018-12-06T09:54:00+00:00",
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [
            -77.329999999999998,
            37.5
        ]
    },
    "properties": {
        "@id": "https://api.weather.gov/stations/KRIC/observations/2018-12-06T09:54:00+00:00",
        "@type": "wx:ObservationStation",
        "elevation": {
            "value": 54,
            "unitCode": "unit:m"
        },
        "station": "https://api.weather.gov/stations/KRIC",
        "timestamp": "2018-12-06T09:54:00+00:00",
        "rawMessage": "KRIC 060954Z 00000KT 10SM CLR M02/M05 A3030 RMK AO2 SLP266 I1000 T10221050",
        "textDescription": "Clear",
        "icon": "https://api.weather.gov/icons/land/night/skc?size=medium",
        "presentWeather": [],
        "temperature": {
            "value": -2.1999999999999886,
            "unitCode": "unit:degC",
            "qualityControl": "qc:V"
        },
        "dewpoint": {
            "value": -5,
            "unitCode": "unit:degC",
            "qualityControl": "qc:V"
        },
        "windDirection": {
            "value": 0,
            "unitCode": "unit:degree_(angle)",
            "qualityControl": "qc:V"
        },
        "windSpeed": {
            "value": 0,
            "unitCode": "unit:m_s-1",
            "qualityControl": "qc:V"
        },
        "windGust": {
            "value": null,
            "unitCode": "unit:m_s-1",
            "qualityControl": "qc:Z"
        },
        "barometricPressure": {
            "value": 102610,
            "unitCode": "unit:Pa",
            "qualityControl": "qc:V"
        },
        "seaLevelPressure": {
            "value": 102660,
            "unitCode": "unit:Pa",
            "qualityControl": "qc:V"
        },
        "visibility": {
            "value": 16090,
            "unitCode": "unit:m",
            "qualityControl": "qc:C"
        },
        "maxTemperatureLast24Hours": {
            "value": null,
            "unitCode": "unit:degC",
            "qualityControl": null
        },
        "minTemperatureLast24Hours": {
            "value": null,
            "unitCode": "unit:degC",
            "qualityControl": null
        },
        "precipitationLastHour": {
            "value": null,
            "unitCode": "unit:m",
            "qualityControl": "qc:Z"
        },
        "precipitationLast3Hours": {
            "value": null,
            "unitCode": "unit:m",
            "qualityControl": "qc:Z"
        },
        "precipitationLast6Hours": {
            "value": null,
            "unitCode": "unit:m",
            "qualityControl": "qc:Z"
        },
        "relativeHumidity": {
            "value": 81.12233310208461,
            "unitCode": "unit:percent",
            "qualityControl": "qc:C"
        },
        "windChill": {
            "value": null,
            "unitCode": "unit:degC",
            "qualityControl": "qc:V"
        },
        "heatIndex": {
            "value": null,
            "unitCode": "unit:degC",
            "qualityControl": "qc:V"
        },
        "cloudLayers": [
            {
                "base": {
                    "value": null,
                    "unitCode": "unit:m"
                },
                "amount": "CLR"
            }
        ]
    }
}
```

As you can see, you can pull out the temperature or wind speed or a variety of other data just from this one endpoint.

## Closing comments

The weather API is a great public service that can be easily utilized in your applications.  When I developed my Weather App, I found it took some trial end error to map out what is returned.  My basic experience with the weather API was that the endpoints were stable, and fairly easy to understand.  My best recommendation would be just to play with the endpoints for some test locations and go from there.