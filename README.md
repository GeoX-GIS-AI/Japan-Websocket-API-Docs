# Overview
This API allows you to search properties from our vast range of footprints data via websockets.

# Autorization
The API requires to be signed with Version 4 signing process of AWS. Here is how we can do it in different ways:

## Python
We can install and use the AWS Requests Auth library from [here](https://pypi.org/project/aws-requests-auth/)

# HTTP response codes
- 200 — Success.
- 400 — Bad Request There was a validation error with the input. Most probably you missed to provide address/prefecture or correaddressionId in the request body.
- 401 — You Signging process was failed. Probably due to incorrect AWS Access Key/Secret Key.
- 403 — Forbidden.
- 404 — Not Found One or more of the required resources was not found, please make sure you entered correct endpoint URL.
- 500 — Internal Server Error This is an issue with our servers processing your request. In most cases we are notified so that we can investigate the issue.

# Getting Started

## Access API with Python
1. We will need following libraries to be installed
   - requests
   - aws_requests_auth
   - websocket

### Python code for websockets connection
```python
import json
import threading
import time

import websocket
from aws_requests_auth.aws_auth import AWSRequestsAuth
from requests import PreparedRequest

AWS_ACCESS_KEY = "your_aws_access_key"
AWS_SECRET_ACCESS_KEY = "your_aws_secret_access_key"


def prepare_aws_auth_headers(request_url):
    _, host = request_url.split("://")
    region = 'ap-northeast-1'

    request = PreparedRequest()
    request.prepare_url(request_url, None)
    request.prepare_headers(dict())
    request.method = "GET"
    aws_details = {
        'aws_access_key': AWS_ACCESS_KEY,
        'aws_secret_access_key': AWS_SECRET_ACCESS_KEY,
        'aws_host': host,
        'aws_region': region,
        'aws_service': "execute-api"
    }
    auth = AWSRequestsAuth(**aws_details)
    return auth(request).headers


def on_message(ws, message):
    print("Received message: ", message)


def on_error(ws, error):
    print("Error: ", error)


def on_close(ws, _, __):
    print("WebSocket connection closed")


def on_open(ws):
    def run(*args):
        get_reqeust = {
            "action": "GET",
            "resource": "onmessage",
            "payload": {
                "address": "大分市中央町３丁目５−8",
                "prefecture": "大分県",
            }
        }
        ws.send(json.dumps(get_reqeust))
        print("GET request sent")
        time.sleep(5)

        post_reqeust = {
            "action": "POST",
            "resource": "onmessage",
            "payload": [
                {
                    "address": "大分市中央町２丁目２−20",
                    "prefecture": "大分県",
                    "correlationId": "1"
                },
                {
                    "address": "大分市中央町３丁目６−29 布屋ビル 別館 1F",
                    "prefecture": "大分県",
                    "correlationId": "2"
                }
            ]
        }
        ws.send(json.dumps(post_reqeust))

        print("POST request sent")
        time.sleep(5)

        # More requests can be sent here

        # Finally close the connection
        ws.close()

    thread = threading.Thread(target=run)
    thread.start()


def start_websocket():
    api_endpoint = "wss://api.geox-ai-japan-commercial-insights.com"
    headers = prepare_aws_auth_headers(api_endpoint)
    headers_list = [f"{header}: {headers[header]}" for header in headers]
    ws = websocket.WebSocketApp(api_endpoint,
                                on_open=on_open,
                                on_message=on_message,
                                on_error=on_error,
                                on_close=on_close,
                                header=headers_list)
    ws.run_forever()


if __name__ == '__main__':
    start_websocket()

```

# Request and Response Samples
Here are the sample request and response

## Request URL
```shell
wss://api.geox-ai-japan-commercial-insights.com
```

## Request Single Sample
```json
{
    "action": "GET",
    "resource": "onmessage",
    "payload": {
        "address": "大分市中央町３丁目５−8",
        "prefecture": "大分県",
    }
}
```
## Request Body Sample (Batch Location API)
```json
{
    "action": "POST",
    "resource": "onmessage",
    "payload": [
        {
            "address": "大分市中央町２丁目２−20",
            "prefecture": "大分県",
            "correlationId": "1"
        },
        {
            "address": "大分市中央町３丁目６−29 布屋ビル 別館 1F",
            "prefecture": "大分県",
            "correlationId": "2"
        }
    ]
}
```

## Response Sample

=> Single Loation API
```json
{
  "data": {
    "parcel": {
      "parcel_id": 44067460,
      "region": "Kyūshū",
      "sports_fields_area": 0.0,
      "solar_panel_ground_area": 0.0,
      "solar_panel_buildings_area": 0.0,
      "prefecture": "Ōita"
    },
    "buildings": [
      {
        "footprint_id": 44067460001,
        "addr": null,
        "dis_vegetation": 0.0,
        "region": "Kyūshū",
        "air_conditioner_area": 0.0,
        "building_volume": null,
        "floors_number": null,
        "flat_area": 67.46,
        "roof_condition": null,
        "lon": 131.60378955307428,
        "tree_height": null,
        "dis_fire_station": 1.16,
        "building_ground_height": null,
        "solar_panel_area": 0.0,
        "dis_road": 0.0,
        "tree_overhang": 0.0,
        "roof_material": "metal",
        "building_height": null,
        "gable_length": 0.0,
        "air_conditioner_count": 0,
        "footprint_area": 75.22,
        "ridge_length": 0.0,
        "roof_type": "flat",
        "dis_closest_building": 6.63,
        "dis_closest_tree": 2.66,
        "dis_coast": -1.0,
        "lat": 33.237590852924825,
        "dis_water_reservoir": 26.69,
        "business_name": null,
        "business_use": null
      },
      {
        "footprint_id": 44067460000,
        "addr": null,
        "dis_vegetation": 8.15,
        "region": "Kyūshū",
        "air_conditioner_area": 0.0,
        "building_volume": null,
        "floors_number": null,
        "flat_area": 28.29,
        "roof_condition": null,
        "lon": 131.60355231741875,
        "tree_height": null,
        "dis_fire_station": 1.18,
        "building_ground_height": null,
        "solar_panel_area": 0.0,
        "dis_road": 0.0,
        "tree_overhang": 0.0,
        "roof_material": "metal",
        "building_height": null,
        "gable_length": 0.0,
        "air_conditioner_count": 0,
        "footprint_area": 45.72,
        "ridge_length": 0.0,
        "roof_type": "flat",
        "dis_closest_building": 9.4,
        "dis_closest_tree": 21.69,
        "dis_coast": -1.0,
        "lat": 33.237519830224265,
        "dis_water_reservoir": 26.68,
        "business_name": null,
        "business_use": null
      }
    ],
    "confidence": 100
  },
  "msg": "OK",
  "status": 200
}
```


=> Batch Location API
```json
{
  "data": [
    {
      "parcel": {
        "parcel_id": 44067525,
        "region": "Kyūshū",
        "sports_fields_area": 0.0,
        "solar_panel_ground_area": 0.0,
        "solar_panel_buildings_area": 0.0,
        "prefecture": "Ōita"
      },
      "buildings": [
        {
          "footprint_id": 44067525000,
          "addr": null,
          "dis_vegetation": 40.0,
          "region": "Kyūshū",
          "air_conditioner_area": 0.0,
          "building_volume": 125.24,
          "floors_number": 4,
          "flat_area": 31.31,
          "roof_condition": null,
          "lon": 131.60521045721748,
          "tree_height": -1.0,
          "dis_fire_station": 1.07,
          "building_ground_height": 3.0,
          "solar_panel_area": 0.0,
          "dis_road": 0.01,
          "tree_overhang": 0.0,
          "roof_material": "metal",
          "building_height": 13.0,
          "gable_length": 0.0,
          "air_conditioner_count": 0,
          "footprint_area": 31.31,
          "ridge_length": 0.0,
          "roof_type": "flat",
          "dis_closest_building": 0.5,
          "dis_closest_tree": 9.5,
          "dis_coast": -1.0,
          "lat": 33.23623574099318,
          "dis_water_reservoir": 26.89,
          "business_name": null,
          "business_use": null
        }
      ],
      "confidence": 100,
      "correlationId": "1"
    },
    {
      "parcel": {
        "parcel_id": 44067280,
        "region": "Kyūshū",
        "sports_fields_area": 0.0,
        "solar_panel_ground_area": 0.0,
        "solar_panel_buildings_area": 0.0,
        "prefecture": "Ōita"
      },
      "buildings": [
        {
          "footprint_id": 44067280000,
          "addr": null,
          "dis_vegetation": 18.07,
          "region": "Kyūshū",
          "air_conditioner_area": 0.0,
          "building_volume": 94.4,
          "floors_number": 2,
          "flat_area": 43.89,
          "roof_condition": null,
          "lon": 131.6045745588111,
          "tree_height": -1.0,
          "dis_fire_station": 1.07,
          "building_ground_height": 4.0,
          "solar_panel_area": 0.0,
          "dis_road": 0.0,
          "tree_overhang": 0.0,
          "roof_material": "concrete",
          "building_height": 8.0,
          "gable_length": 0.0,
          "air_conditioner_count": 0,
          "footprint_area": 47.2,
          "ridge_length": 0.0,
          "roof_type": "flat",
          "dis_closest_building": 2.44,
          "dis_closest_tree": 11.72,
          "dis_coast": -1.0,
          "lat": 33.23836613118673,
          "dis_water_reservoir": 26.66,
          "business_name": null,
          "business_use": null
        }
      ],
      "confidence": 100,
      "correlationId": "2"
    }
  ],
  "msg": "OK",
  "status": 200
}
```