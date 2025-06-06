## purpose
This file will include APIs that I want to specify for AI built code, so that a LLM can read this file and know the methods and expected results of key public APIs that it's not great at locating


## General Instructions

Always enable Dark Mode following the Dark Mode Guidance and the Garden component option
### Dark Mode Guidance
Source: https://developer.zendesk.com/documentation/apps/app-developer-guide/dark-mode/

## APIs
### Zendesk
#### Custom Objects
##### List Custom Objects
Use List Custom Objects (GET /api/v2/custom_objects) to get all the custom objects in the account.  The results with be a JSON object with a "custom objects" array.  The array will look like this:
{
  "custom_objects": [
    {
      "url": "https://z3n-whiterose.zendesk.com/api/v2/custom_objects/01___field.json",
      "key": "01___field",
      "created_by_user_id": "1263528238050",
      "updated_by_user_id": "1263528238050",
      "created_at": "2024-11-25T20:42:45Z",
      "updated_at": "2024-11-25T20:42:45Z",
      "title": "01 - Field",
      "raw_title": "01 - Field",
      "title_pluralized": "01 - Fields",
      "raw_title_pluralized": "01 - Fields",
      "description": "",
      "raw_description": "",
      "include_in_list_view": true,
      "allows_photos": false
    }]}

##### Count Custom Object Records
Use Count Custom Object Records (GET /api/v2/custom_objects/{custom_object_key}/records/count) 

The count will be a "count" object with a "value" element
{
  "count": {
    "value": 2,
    "refreshed_at": "2025-06-05T21:05:08+00:00"
  }
}
