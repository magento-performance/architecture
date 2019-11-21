# Phase 1

With phase 1 we are planning to develop next functionality: 
- Endpoint to receive file
- Endpoint to start processing
- Endpoint to receive status

## File Upload Endpoint
### Source Upload endpoint

So import will start from uploading import Source file. Currently we will support "csv" files format

POST  `/V1/import/source/csv`
 
This request can accept files from different sources:
- Local file path
- Direct link to the file
- Base64 encoded file content 
 
#### Local file path

Path is relative from Magento Root folder

```
{
  "source": {
      "import_data": "var/catalog_product.csv",
      "import_type": "local_path",
      "uuid": "UUID",
      "format": {
          "csv_separator": "string",
          "csv_enclosure": "string",
          "csv_delimiter": "string",
          "multiple_value_separator": "string"
      }
  }
}
```

#### Direct link to the file

```
{
  "source": {
      "import_data": "http://some.domain/file.csv",
      "import_type": "external",
      "uuid": "UUID",
      "format": {
          "csv_separator": "string",
          "csv_enclosure": "string",
          "csv_delimiter": "string",
          "multiple_value_separator": "string"
      }
  }
}
```

#### Base64 encoded file content 

```
{
  "source": {
      "import_data": "c2t1LHN0b3JlX3ZpZXdfY29kZSxhdHRyaWJ1dGVfc2V0X2NvZGUscHJvZHVjdF90eXBlLGNhdGVnb3JpZXMscHJvZHVjdF93ZWJzaXRlcyxuYW1lLGRlc2NyaXB0aW9uLHNob3J0X2Rlc2NyaXB0aW9uLHdlaWdodCxwcm9kdWN0X29ubGluZSx0YXhfY2xhc3NfbmFtZSx2aXNpYmlsaXR5LHBya......",
      "import_type": "base64_encoded_data",
      "uuid": "UUID",
      "format": {
          "csv_separator": "string",
          "csv_enclosure": "string",
          "csv_delimiter": "string",
          "multiple_value_separator": "string"
      }
  }
}
```

Import of big file also can be divided in several parts.
For this case we have separate endpoint

POST  `/V1/import/source/csv/partial/`

Input request will looks like:

```
{
  "source": {
      "import_data": "c2t1LHN0b3JlX3ZpZXdfY29kZSxhdHRyaWJ1dGVfc2V0X2NvZGUscHJvZHVjdF90eXBlLGNhdGVnb3JpZXMscHJvZHVjdF93ZWJzaXRlcyxuYW1lLGRlc2NyaXB0aW9uLHNob3J0X2Rlc2NyaXB0aW9uLHdlaWdodCxwcm9kdWN0X29ubGluZSx0YXhfY2xhc3NfbmFtZSx2aXNpYmlsaXR5LHBya...",
      "data_hash" : "sha256 encoded data of the full 'import_data' value"
      "pieces_count": "5"
      "piece_number": "1",
      "import_type": "base64_encoded_data",
      "uuid": "UUID",
      "format": {
         "csv_separator": "string",
         "csv_enclosure": "string",
         "csv_delimiter": "string",
         "multiple_value_separator": "string"
    }
  }
}
```
where *import_data* is a 1/N part of the whole content, and *data_hash* contains sha256 hash of full import_data body.

`pieces_count` - its an amount of pieces that will be transferred for 1 file. We need it to be sure that import is completed and then we could detect if it was successfully finished or failed

`piece_number` - its a number that detects which part of file currently transferred. This is required to have to support Asynchronous File import when we dont need to send parts in correct sequence

Those parts could be send asynchronously. They will be merged together after all data are transferred.

### Return values

As return user will receive:

| Key | Value |
| --- | --- |
| uuid | Imported File ID |
| status | Status of this file. Possible values: completed, uploaded, failed |
| error | Error message if exists |

Example:

```
{
    "uuid": null,
    "status": null,
    "error": null,
    "source": {
        // Source object is coming here
    }
}
```

### Update Imported Source Format

Its possible also to Update Format 

PUT  `/V1/import/source/csv/:uuid`

```
{
  "source": {
      "uuid": "uuid",
      "format": {
          "csv_separator": "string",
          "csv_enclosure": "string",
          "csv_delimiter": "string",
          "multiple_value_separator": "string"
      }
  }
}
```

### Delete Imported Source Format

DELETE  `/V1/import/source/:uuid`

### Get List of sources

GET `/V1/import/sources/?searchCriteria`

Will return list of Source that was uploaded before.

```
{
  "sources": [
  {
      "uuid": "uuid",
      "format": {
          "csv_separator": "string",
          "csv_enclosure": "string",
          "csv_delimiter": "string",
          "multiple_value_separator": "string"
      }
  },
  {
        "uuid": "uuid",
        "format": {
            "csv_separator": "string",
            "csv_enclosure": "string",
            "csv_delimiter": "string",
            "multiple_value_separator": "string"
        }
    }
    .....
  ],
  "search_criteria": {
      "filter_groups": [
        {
          "filters": [
            {
              "field": "string",
              "value": "string",
              "condition_type": "string"
            }
          ]
        }
      ],
      "sort_orders": [
        {
          "field": "string",
          "direction": "string"
        }
      ],
      "page_size": 0,
      "current_page": 0
    },
    "total_count": 0
}
```

## Start File Import Endpoint
### Main endpoint

Current Endpoint starts import process based on FileID. Where Module will read file, split it into message and send to Async API

POST  `/V1/import/start/{uuid}`

`type` - its an import type like: catalog_product, catalog_category, customer, order ... etc...

Start File Import

```
{
    "importConfig": {
        "import_type": "catalog_product",
        "import_strategy": "add_update, delete, replace",
        "validation_strategy": "string",
        "allowed_error_count": 0,
        "import_image_archive": "string",
        "import_images_file_dir": "string",
        "mapping": [
              {
                  "name": "string",
                  "source_path": "string",
                  "target_path": "string",
                  "target_value": "mixed",
                  "processing_rules": [
                        {
                              "sort": int,
                              "function": "string",
                              "args": "array"
                        }
                  ]
              }
        ]
    }
}
```

| Key | Value |
| --- | --- |
| uuid | UUID that was returned by source upload call |
| behaviour | Import behaviour (add_update, delete, update, add, replace) |
| import_image_archive | Relative path to product images archive file |
| import_images_file_dir | Relative path to product images files |
| validation_strategy | Moved from main standard Import, not sure if we will use if |
| allowed_error_count | How many errors allowed to be during the import |
| mapping | Data format mapping |


#### Return

```
{
	"uuid": string
	"status": "proccessing",
	"error": "string"
}
```

### Get import status

Receive information about imported file
GET  `/V1/import/:uuid`

#### Return

Will be returned list of objects that we tried to import

```
{
  "status": "string", 
  "error": "string",
  "uuid": 0,
  "entity_type": "catalog_product, customers ....",
  "user_id": "User ID who created this request",
  "user_type": "User Type who created this request",
  "items": [
  {
    "uuid": 0,
    "status": "",
    "serialized_data": "",
    "result_serialized_data": "",
    "error_code":"",
    "result_message":""
  }]
}
```

| Key | Value |
| --- | --- |
| uuid | Imported File ID |
| status | Status of this file. Possible values: completed, not_completed, error |
| error | Error message if exists |
| entity_type | Import type: eg. customers, products, etc ... |
| items | List of items that were imported. As an array |

### Get single import operation status

Receive information about imported file
GET  `/V1/import/operation/:uuid`

#### Return

Will be returned specific object that we tried to import

```
{
  "status": "string", 
  "error": "string",
  "uuid": 0,
  "entity_type": "catalog_product, customers ....",
  "user_id": "User ID who created this request",
  "user_type": "User Type who created this request",
  "items": [
  {
    "uuid": 0,
    "status": "",
    "serialized_data": "",
    "result_serialized_data": "",
    "error_code":"",
    "result_message":""
  }]
}
```

| Key | Value |
| --- | --- |
| uuid | Imported File ID |
| status | Status of this file. Possible values: completed, not_completed, error |
| error | Error message if exists |
| entity_type | Import type: eg. customers, products, etc ... |
| items | Item that were requested. For do not change interfaces it will be still as an array, but always one item |

##### And Item object will contain

This values are based on magento "magento_operation" table

| Key | Value |
| --- | --- |
| uuid | Entity UUID - defined by customer OR auto-generated by system |
| status | Import status. "pending, failed, processing, completed" |
| serialized_data | Data that was send to Magento, via Async endpoint |
| result_serialized_data | Data that Magento returned for this object after import |
| error_code | Error code |
| result_message | Result message of operation execution |


## Repeatable call endpoint

Main idea, is in case some operation import failure, that user could align input data and repeat import for this particular item

PUT  `/V1/import/operation-id/{uuid}`
 
```
{
  "serialized_data":""
}
```

### Return

```
[]
```
