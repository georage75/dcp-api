# 1. struct-api
Dual-Coin Vendor API Doc
Vendor should offer the following APIs

## 1.1. Encoding

The request and response data for all interfaces is formatted in UTF-8. Content for responses in all interfaces is formatted in JSON.

## 1.2. Common parameters in HTTP headers

Below common parameters in HTTP headers are recommended to added in request, facilitating debugging and tracing each request.

| Key          | Type   | Position | Required | Description                                                     |
| ------------ | ------ | -------- | -------- | --------------------------------------------------------------- |
| x-request-id | string | header   | false    | uuid for trace the request, not included in signing the request |

## 1.3. Authentication

## 1.4. Private API mandatory fields
- Client put Access Key in http request header X-Access-Key
- Client and Server use same private secret key to sign the request.
- Client must add `timestamp` (epoch in millisecond) field in request parameter (query string for GET, json body for POST), API server will check this timestamp, if `abs(server_timestamp - request_timestamp) > 5000`, the request will be rejected.
- `timestamp` must be integer, not quoted string.
- Client must add `signature` field in request parameter (query string for GET, json body for POST).
- For POST request, Header `Content-Type` should be set as `application/json`.

| Key       | Type   | Position      | Required | Description              |
| --------- | ------ | ------------- | -------- | ------------------------ |
| timestamp | int    | query or body | true     | epoch in millisecond     |
| signature | string | query or body | true     | signature of the request |

## 1.5. Signature algorithm

```python
import hashlib
import hmac

class Auth:
    def __init__(self, secret_key):
        self.secret_key = secret_key

    def encode_list(self, item_list):
        list_val = []
        for item in item_list:
            obj_val = self.encode_object(item)
            list_val.append(obj_val)
        output = "&".join(list_val)
        output = "[" + output + "]"
        return output

    def encode_object(self, param_map):
        sorted_keys = sorted(param_map.keys())
        # print("sorted_keys", sorted_keys)
        ret_list = []
        for key in sorted_keys:
            val = param_map[key]
            if isinstance(val, list):
                list_val = self.encode_list(val)
                ret_list.append(f"{key}={list_val}")
            elif isinstance(val, dict):
                # call encode_object recursively
                dict_val = self.encode_object(val)
                ret_list.append(f"{key}={dict_val}")
            elif isinstance(val, bool):
                bool_val = str(val).lower()
                ret_list.append(f"{key}={bool_val}")
            else:
                general_val = str(val)
                ret_list.append(f"{key}={general_val}")

        sorted_list = sorted(ret_list)
        output = "&".join(sorted_list)
        return output

    def get_signature(self, api_path, param_map):
        str_to_sign = api_path + "&" + self.encode_object(param_map)
        # print("str_to_sign = " + str_to_sign)
        sig = hmac.new(
            self.secret_key.encode("utf-8"),
            str_to_sign.encode("utf-8"),
            digestmod=hashlib.sha256,
        ).hexdigest()
        return sig
```
Explanation:
1. Request parameters: JSON Body for POST, query string for GET
1. Encode string to sign, for simple json object, sort your parameter keys alphabetically, and join them with '&' like ```'param1=value1&param2=value2'```, then get ```str_to_sign = api_path + '&' + 'param1=value1&param2=value2'```
- For nested array objects, encode each object and sort them alphabetically, join them with ```'&'``` and embraced with ```'[', ']'```
- e.g.```str_to_sign = api_path + '&' + 'param1=value1&array_key1=[array_item1&array_item2]'```.
1. ```Signature = hex(hmac_sha256(str_to_sign, secret_key))```
1. Add `signature` field to request parameter:
- for query string, add ```'&signature=YOUR_SIGNATURE'```
- for JSON body, add ```{"signature":YOUR_SIGNATURE}```


## 1.6. DCP Vendor Rest API


### 1.6.1 Product list


- Interface Details:

    - H5 list product

- GET `/structured/api/v1/product/list`

- Request Parameters

    - Request Parameter Location: query_string


| Parameter Name  | Type   | Mandatory | Description                                                                                              |
|:----------------|--------|:----------|:---------------------------------------------------------------------------------------------------------|
| strategy_id     | string | Y         | The ID of the strategy (string format).                                                                  |
| type            | string | Y         | Type of strategy (CALL for Bullish, PUT for Bearish, "" for no direction attribute).                     |
| protection_type | int    | Y         | Type of protection (0:NoProtectionType, 1:Protection, 2:Unprotected).                                    |
| base_currency   | string | Y         | Filter by base currency (e.g., "BTC" for BTC-USDT). Leave empty ("") for all currencies.                 |
| invest_currency | string | Y         | Filter by investment currency (e.g., "BTC" or "USDT" for BTC-USDT). Leave empty ("") for all currencies. |


- Response Results:
    - Response strings are as follows. General string text has been omitted. Please see Response Format Strings for more details.
- Data Structure & Description:


```shell
curl "https://mapi.matrixport.com/structured/api/v1/product/list?base_currency=%22%22&invest_currency=%22%22&protection_type=0&strategy_id=7184078087301488640"
```


```js
{
  "code": 0,
          "data": {
    "strategy_id": "7184078087301488640",
            "icon_url": "https://rd2-images.matrixtechfin.com/752edd04-08d1-4f02-8e4a-2fc5c31be98c.png",
            "name": [
      {
        "id": "0",
        "created_at": 0,
        "updated_at": 0,
        "tag_id": "0",
        "language": "en-US",
        "name": "Sharkfin"
      },
      {
        "id": "0",
        "created_at": 0,
        "updated_at": 0,
        "tag_id": "0",
        "language": "zh-CN",
        "name": "鲨鱼鳍"
      },
      {
        "id": "0",
        "created_at": 0,
        "updated_at": 0,
        "tag_id": "0",
        "language": "zh-TW",
        "name": "鯊魚鰭"
      }
    ],
            "introduction": [
      {
        "id": "0",
        "created_at": 0,
        "updated_at": 0,
        "tag_id": "0",
        "language": "en-US",
        "name": "鲨鱼鳍简介-en"
      },
      {
        "id": "0",
        "created_at": 0,
        "updated_at": 0,
        "tag_id": "0",
        "language": "zh-CN",
        "name": "鲨鱼鳍简介-cn"
      },
      {
        "id": "0",
        "created_at": 0,
        "updated_at": 0,
        "tag_id": "0",
        "language": "zh-TW",
        "name": "鲨鱼鳍简介-zh-tw"
      }
    ],
            "intro_url": "",
            "intro_link": [
      {
        "id": "0",
        "created_at": 0,
        "updated_at": 0,
        "tag_id": "0",
        "language": "en-US",
        "name": "https://www.baidu.com/"
      },
      {
        "id": "0",
        "created_at": 0,
        "updated_at": 0,
        "tag_id": "0",
        "language": "zh-CN",
        "name": "https://www.baidu.com/"
      },
      {
        "id": "0",
        "created_at": 0,
        "updated_at": 0,
        "tag_id": "0",
        "language": "zh-TW",
        "name": "https://www.baidu.com/"
      }
    ],
            "intro_pic": null,
            "aggregated_products": null,
            "guidance": [
      {
        "item_id": "0",
        "item_i18n": [
          {
            "language": "en-US",
            "content": "https://rd2-images.matrixtechfin.com/e4837bbc-9c60-44dc-b87f-e71e84618f69.jpeg"
          },
          {
            "language": "zh-CN",
            "content": "https://rd2-images.matrixtechfin.com/e69df72d-1ee3-47cf-a9d9-25e34dda50ff.jpeg"
          },
          {
            "language": "zh-TW",
            "content": "https://rd2-images.matrixtechfin.com/331408b1-9776-4f55-95ed-57bec9eb7b7a.jpeg"
          }
        ]
      }
    ]
  },
  "message": ""
}
```

###


| Parameter Name      | Type      | Description                                              |
|:--------------------|:----------|:---------------------------------------------------------|
| strategy_id         | string    | The unique identifier of the strategy                    |
| icon_url            | string    | Strategy icon url                                        |
| name                | array     | Name of product                                          |
| introduction        | array     | Introduction of Product                                  |
| intro_url           | string    | URL related to the introduction                          |
| intro_link          | array     | URL link related to the product                          |
| intro_pic           | array     | Array of pictures used for introducing the product       |
| aggregated_products | array     | Array of aggregated products associated with the product |
| guidance            | array     | Array Guidance of the product                            |

