# Introduction

Since GitHub currently does not support any form of groups, folders, buckets, or whatever you might call a feature for organizing repositories, this file serves as a central point for three repositories I recently created.

These repositories are closely related because, together, they form a system for:
- Scanning Wi-Fi networks and capturing BSSID, SSID, capabilities, and especially signal strength at geolocations using an Android phone.
- Parsing the output JSON file from the Android app, calculating approximate access point locations, and pushing this data into a database.
- Presenting this data on a map within a web application.

## Wi-Fi Cartophrapher

📂 https://github.com/marianpekar/wifi-cartographer

An Android app written in [Kotlin](https://kotlinlang.org) for scanning Wi-Fi networks and capturing BSSID, SSID, capabilities, and signal strength at specific geolocations. The output is a JSON file, with the default name `wifi_map_data.json`, that contains an array of objects that follow this JSON schema.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "patternProperties": {
    "^[0-9a-fA-F]{2}(:[0-9a-fA-F]{2}){5}$": {
      "type": "object",
      "properties": {
        "capabilities": {
          "type": "string",
          "pattern": "\\[.*\\]"
        },
        "frequency": {
          "type": "integer",
        },
        "lastUpdateTime": {
          "type": "integer",
        },
        "levels": {
          "type": "object",
          "patternProperties": {
            "^\\d{2}\\.\\d+,-?\\d{2}\\.\\d+$": {
              "type": "integer",
              "minimum": -90,
              "maximum": 0
            }
          },
          "additionalProperties": false
        },
        "ssid": {
          "type": "string",
        }
      },
      "required": ["capabilities", "frequency", "lastUpdateTime", "levels", "ssid"],
      "additionalProperties": false
    }
  },
  "additionalProperties": false
}
```

## Wi-Fi Map Data Importer

📂 https://github.com/marianpekar/wifi-map-data-importer

A [.NET](https://dotnet.microsoft.com) 8 console application that parses the `wifi_map_data.json` output file from the *Wi-Fi Cartographer* Android app, calculates the approximate location of each AP, and stores a record for each network as a document in a [MongoDB](https://www.mongodb.com) collection where every document follows this JSON schema.


```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "_id": {
      "type": "object",
      "properties": {
        "$oid": {
          "type": "string",
          "pattern": "^[0-9a-fA-F]{24}$"
        }
      },
      "required": ["$oid"],
      "additionalProperties": false
    },
    "Bssid": {
      "type": "string",
      "pattern": "^[0-9a-fA-F]{2}(:[0-9a-fA-F]{2}){5}$"
    },
    "Ssid": {
      "type": "string",
    },
    "Frequency": {
      "type": "integer",
    },
    "Capabilities": {
      "type": "string",
    },
    "Longitude": {
      "type": "number",
      "minimum": -180,
      "maximum": 180
    },
    "Latitude": {
      "type": "number",
      "minimum": -90,
      "maximum": 90
    }
  },
  "required": ["_id", "Bssid", "Ssid", "Frequency", "Capabilities", "Longitude", "Latitude"],
  "additionalProperties": false
}
```

## Wi-Fi Map

📂 https://github.com/marianpekar/wifi-map

A web application written in [Node.js](https://nodejs.org) using the [Express](https://expressjs.com) framework, [Pug](https://pugjs.org/) templates, and [OpenStreetMap](https://www.openstreetmap.org/) to list networks stored in [MongoDB](https://www.mongodb.com) and mark the approximate location of each AP on a map.
