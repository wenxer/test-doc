# API Reference

## Introduction

MeasureAI provides an API interface for developers to call our measurement services. JSON is returned by all API responses, including [errors](#errors). 

### Requirements

You need to [register](https://measure.productai.com/trial) a MeasureAI account, and get the Access Key ID. The Access Key ID is available via the ProductAI Console: 

1. Login to [ProductAI Console](https://console.productai.com/login).
2. Click your username on the top right corner, choose “My Account”.
3. Copy the `access_key_id` in “API Key Information” panel. 

### Image Specifications

Images sent to the API either as an image file handler object, in string form as `base64`, or as an image URL. 

- Image size: 1024 X 1024 or higher. 
- Supports `.jpg`, `.jpeg` or `.png` formats. 
- Image file size should be less than `15 MB`. 

## Authentication

Authenticate your account by including your `access_key_id` in API requests. You can manage your API `access_key_id` in the [ProductAI Console](https://console.productai.com/login). Your API `access_key_id` carry many privileges, so be sure to keep them secure! Do not share your secret API `access_key_id` in publicly accessible areas such as GitHub, client-side code, and so forth.

Authentication to the API is performed via HTTP Header. 

````
-H 'x-ca-version: 1.0'
-H 'x-ca-accesskeyid: ACCESS_KEY_ID' 
````


## Measurements

Make a request by sending an image, either as an image file handler object, in string form as `base64`, or as an image URL. The API can be invoked using POST method, in JSON format, to: `https://api.productai.com/measure/_0000203/predict`


### Request

The request `body` should contain the image data for measurements. Only 1 input parameter is required; if multiple input parameters are provided, the API will use first available one. 

| Param | Required | Type | Description |
|-------|---------|------|-------------|
| image_base64 | Yes | String | Image encoded in `base64` |
| image_url | Yes | String | Image url |
| image_file | File | Image file handler | Image file, e.g. `.jpg` |


Example:

````
curl -H "Content-Type:application/json" \
    -H 'x-ca-version: 1.0' \
    -H 'x-ca-accesskeyid: ACCESS_KEY_ID' \
    -X POST \
        --data '{"image_url":"http://image_url"}' \
        'https://api.productai.com/measure/_0000203/predict'
````

### Response

All API responses are returned in JSON. 

- `request_id` is a unique ID for the API request. 
- `results` contains the measurement results. 


| Field | Type | Description |
|-------|------|-------------|
| `measurements` | Object | Measurements in `mm`, as JSON object. For details see [Label Reference](#label-reference). |
| `get_image_seconds` | Float | Duration for fetching or uploading image |
| `process_seconds` | Float | Duration for processing API request|


Example:

````
{
    "request_id": "f9e0953b-b7b2-40aa-b60f-2b577cca0324",
    "results": {
        "measurements": {
            "arm-hole": 215,
            "body-length": 550,
            "body-width": 390,
            "chest-circumference": 775,
            "hem-circumference": 820,
            "hem-width": 410,
            "length": 515,
            "raglan-sleeve-length": 355,
            "shoulder-width": 345,
            "sleeve-circumference": 290,
            "sleeve-length": 165,
            "sleeve-width": 145
        },
        "get_image_seconds": 0.520123,
        "process_seconds": 1.234
    }
}
````

## Batch Process

Batch Process API create asynchronous tasks convenient for handling large number of images. You can pass a list of images, and check for results at a later time. The API can be invoked by using POST method, in JSON format, to: `https://api.productai.com/measure/_0000203/batch/process`. 

You can [check task status](#check-task-status), and if its completed successfully, download the results from the output url. 


### Request

The request `body` should contain a name for the batch processing request, and a list of images for processing. The images can be passed as a `JSON` array, a `.csv` file, or `.json` file. Only 1 input parameter is required; if multiple input parameters are provided, the API will use first available one. 


| Param | Required | Type | Description |
|-------|---------|------|-------------|
| name | Yes | String | Task name |
| items | Array | Array of [Item](#item-object) objects. |
| item_json_file | File | `.json` file, with an [Item](#item-object) object for each line |
| item_csv_file | File | `.csv` file with 2 columns, `image_url` and `item_id` |


#### Item Object

| Param | Required | Type | Description |
|-------|---------|------|-------------|
| `item_url` | Yes | String | URL path to image |
| `item_id` | No | String | A meta tag for identifying this item |



Example:

````
payload = '{
    "name":"tshirts_task",
    "items":[
        {
            "image_url":"http://image_url",
            "item_id":"tshirt_0001"
        }
    ]
}'

curl -H "Content-Type:application/json" \
    -H 'x-ca-version: 1.0' \
    -H 'x-ca-accesskeyid: ACCESS_KEY_ID' \
    -X POST \
        --data $payload \
        'https://api.productai.com/measure/_0000203/batch/process'
````

### Response

All API responses are returned in JSON

- `request_id` is a unique ID for the API request. 
- `results` contains the results of the task request. 

Tasks

| Field | Type | Description |
|-------|------|-------------|
| id | String | Task ID used to [check task status](#check-task-status). Note -- this is different from `request_id`. |
| name | String | Task name |
| status | Integer | [Task Status Code](#check-task-status) |
| output_url | String | URL to output `.json` file containing results. `output_url` will be null until task is finished. |
| count | Integer | Total num of items received in request |
| input_url | String | URL to request input file |
| cid   | String | Client ID |
| create_at | LongInt | Timestamp of task created |
| updated_at | LongInt | Timestamp of last task update |



Example:

````
{
    "request_id": "0562dde8-c3ba-11e8-9e6e-0242ac1c2404",
    "results": {
        "cid": "3479",
        "count": 3,
        "create_at": 1538206435,
        "id": "5baf2ae3ec4a130011638cc7",
        "input_url": "http://path_to_input.json",
        "name": "tshirts_1000",
        "output_url": null,
        "status": 1,
        "update_at": 1538206436
    }
}
````


### Check Task Status

Batch Process API creates asynchronous tasks for process large number of items. You may check task status and download output results if completed successfully. 

Task status API may be invoked by using GET method, with task ID as parameter: 

`https://api.productai.cn/measure/_0000203/batch/process/<task_id>`. 


| status code | Description |
|-------------|-------------|
| 0 | Pending |
| 1 | Started |
| 2 | Completed successfully |
| 4 | Finished with error |


Example: 

````
curl -H "Content-Type:application/json" \
    -H 'x-ca-version: 1.0' \
    -H 'x-ca-accesskeyid: ACCESS_KEY_ID' \
    -X GET 'https://api.productai.com/measure/_0000203/batch/process/<task_id>'

````

## Label Reference

MeasureAI currently supports 4 types of garments:
- Tops, such as Tshirts, Polo shirts, Dress shirts, Blouses. 
- Pants, such as shorts and trousers
- Dress
- Skirt
- Coats (beta testing!)

Different garment types have different measurement labels. All labels are returned in English by default. All measurement values are in millimeter `mm`. 


#### Tops

| Label | Chinese `zh` | Japanese `ja` |
|-------|--------------|---------------|
| body-width | 身宽 | 身幅 |
| shoulder-width | 肩宽 | 肩幅 |
| chest-circumference  | 胸围 | 胸囲 |
| hem-width | 下摆宽 | 裾幅 |
| hem-circumference | 下摆围 | 裾まわり |
| sleeve-width | 袖宽 | 袖幅 |
| sleeve-circumference | 袖围 | 袖まわり |
| sleeve-length | 袖长 | 袖丈 |
| raglan-sleeve-length | 套袖长度  | 裄丈 |
| arm-hole | 袖孔宽  | アームホール |
| body-length  | 身长 | 身丈 |
| length | 衣长 | 着丈 |

#### Pants

| Label | Chinese `zh` | Japanese `ja` |
|-------|--------------|---------------|
| waist-width | 腰宽 | ウエスト幅 |
| waist-circumference | 腰围 | ウエスト |
| hip-width  | 臀宽 | ヒップ幅 |
| hip-circumference | 臀围 | ヒップ |
| thigh-width  | 大腿宽 | ワタリ幅 |
| thigh-circumference | 大腿围 | 腿まわり |
| hem-width | 裤脚宽 | 裾幅 |
| hem-circumference  | 裤脚围  | 裾まわり |
| rise-length | 前浪长 | 股上 |
| inseam  | 内侧长 | 股下 |
| length  | 裤长  | パンツの長さ |


#### Dress

| Label | Chinese `zh` | Japanese `ja` |
|-------|--------------|---------------|
| body-width | 身宽 | 身幅 |
| shoulder-width | 肩宽 | 肩幅 |
| chest-circumference  | 胸围 | 胸囲 |
| hem-width | 下摆宽 | 裾幅 |
| hem-circumference | 下摆围 | 裾まわり |
| sleeve-width | 袖宽 | 袖幅 |
| sleeve-circumference | 袖围 | 袖まわり |
| sleeve-length | 袖长 | 袖丈 |
| raglan-sleeve-length | 套袖长度  | 裄丈 |
| arm-hole | 袖孔宽  | アームホール |
| body-length  | 身长 | 身丈 |
| length | 衣长 | 着丈 |
| waist-width | 腰宽 | ウエスト幅 |
| waist-circumference | 腰围 | ウエスト |
| hip-width  | 臀宽 | ヒップ幅 |
| hip-circumference | 臀围 | ヒップ |

#### Skirt

| Label | Chinese `zh` | Japanese `ja` |
|-------|--------------|---------------|
| waist-width | 腰宽 | ウエスト幅 |
| waist-circumference | 腰围 | ウエスト |
| hem-width | 下摆宽 | 裾幅 |
| hem-circumference | 下摆围 | 裾まわり |
| length | 衣长 | 着丈 |

#### Coat

| Label | Chinese `zh` | Japanese `ja` |
|-------|--------------|---------------|
| body-width | 身宽 | 身幅 |
| shoulder-width | 肩宽 | 肩幅 |
| chest-circumference  | 胸围 | 胸囲 |
| hem-width | 下摆宽 | 裾幅 |
| hem-circumference | 下摆围 | 裾まわり |
| sleeve-width | 袖宽 | 袖幅 |
| sleeve-circumference | 袖围 | 袖まわり |
| sleeve-length | 袖长 | 袖丈 |
| arm-hole | 袖孔宽  | アームホール |
| body-length  | 身长 | 身丈 |
| length | 衣长 | 着丈 |


## Errors

All errors are returned in JSON format. 

| error_code | Description | 
|------------|-------------|
| 2001 | Image parameter is not provided. Should provide image in
`image_file`, `image_url` or `image_base64` parameters. |
| 2002 | The image size is too small. Please see [Image Specifications](#image-specifications) for requirements. |
| 2003 | The image file size is too large. Please see [Image Specifications](#image-specifications) for requirements. | 
| 2004 | The image file type is not supported. Please see [Image Specifications](#image-specifications) for requirements. |
| 2005 | Can’t retrieve image through the url. |
| 2006 | Timeout error. The image download time is more than 10 seconds. |
| 2007 | Image download failed. |
| 2008 | The image url is invalid. | 
| 2009 | Failed to recognize MeasureKit marker in the image. |
| 2010 | Garment detection error. |
| 2011 | Failed to detect garment in image. | 
| 2012 | Multiple garments were detected in the image. |
| 2013 | The garment type is not supported for measurement. |
| 3004 | `ACCESS_KEY_ID` incorrect. |
| 3005 | `ACCESS_KEY_ID` expired. |
| 3007 | Request missing `ACCESS_KEY_ID` |
| 4001 | Request data not provided. Should provide `items`, 
`item_json_file` or `item_csv_file`. |
| 4002 | Request data exceeded max limit `50,000`. Please contact your account representative. |
| 4003 | Item is missing or has invalid `image_url` |
| 4004 | Request data is empty | 
| 4005 | Task ID invalid | 



