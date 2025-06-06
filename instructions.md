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
#### Zendesk Integration Services (ZIS)
##### List Integrations
Use List Itegrations (/api/services/zis/registry/integrations) to get the description and name of every ZIS integration installed in the instance.
This API will return a JSON object witn an "integrations" array.  The array will look like this:
{
    "integrations": [
        {
            "description": "whiterose test intgeration",
            "jwt_public_key": "<<JWT.token>>",
            "name": "z3n-whiterose_first_test_webhook",
            "zendesk_oauth_client": {
                "id": 1260800799110,
                "identifier": "zis_z3n-whiterose_first_test_webhook",
                "secret": "************"
            }
        }
    ]
}
##### List Bundles

Use List Bundles (/api/services/zis/registry/<<integration_name>>/bundles) to get the UUID of each ZIS bundle associated with an integration (as identified by the name of the integration from the List Integrations API above).

This API will return a JSON object with a "bundles" array.  The array will look like this:
{
    "bundles": [
        {
            "description": "When a ticket is created, scan it for keywords and put them in fields",
            "integration": "whiterose_ticket_field_extraction_dev",
            "name": "Put field data in fields",
            "uuid": "0445b9f1-2888-4648-b014-1b6c3cb311f4",
            "zendesk_account_id": 10890301,
            "zis_template_version": "2019-10-14"
        }
    ],
    "meta": {
        "after": "0445b9f1-2888-4648-b014-1b6c3cb311f4",
        "before": "0445b9f1-2888-4648-b014-1b6c3cb311f4",
        "has_more": false
    }
}

##### Get Bundle By UUID
Use Get BundleByUUID () to get the contents of the bundle identified by the uuid from the list of Bundles (see List Bundles above)

The API will return a bundle, which is described here: https://developer.zendesk.com/documentation/integration-services/getting-started/anatomy-zis-bundle/

The bundle consists of metadata and resources.

Inside  the resources object,  each named child element is either an action (identified by the type "ZIS::Action::Http"), a flow (Indentified the the type "ZIS::Flow"), or a JobSpec (Identified by the type "ZIS::JobSpec").

A small example bundle looks like this
{
    "uuid": "c2ea14ba-74e7-46cc-a504-127dc822ff1e",
    "name": "post_tickets_to_webhook",
    "integration": "z3n-whiterose_first_test_webhook",
    "zendesk_account_id": 10890301,
    "description": "Post ZD Ticket create events to webhook",
    "zis_template_version": "2019-02-28",
    "resources": {
        "post-to-webhook-site": {
            "type": "ZIS::Action::Http",
            "properties": {
                "definition": {
                    "headers": [
                        {
                            "key": "X-Zendesk-Ticket-Id",
                            "value": "{{$.ticketId}}"
                        }
                    ],
                    "method": "POST",
                    "requestBody": {
                        "webhook": {
                            "description": "{{$.ticketDescription}}",
                            "subject": "{{$.ticketSubject}}"
                        }
                    },
                    "url": "https://webhook.site/2f9c68c5-e118-403f-83b1-a000965895a2"
                },
                "name": "post-to-webhook-site"
            }
        },
        "post_tickets_to_webhook_flow": {
            "type": "ZIS::Flow",
            "properties": {
                "definition": {
                    "StartAt": "Zendesk.TicketCreated",
                    "States": {
                        "Zendesk.TicketCreated": {
                            "ActionName": "zis:z3n-whiterose_first_test_webhook:action:post-to-webhook-site",
                            "End": true,
                            "Parameters": {
                                "ticketDescription.$": "{{$.input.ticket.description}}",
                                "ticketId.$": "{{$.input.ticket.id}}",
                                "ticketSubject.$": "{{$.input.ticket.subject}}"
                            },
                            "Type": "Action"
                        }
                    }
                },
                "name": "post_tickets_to_webhook_flow"
            }
        },
        "ticket_created_jobspec": {
            "type": "ZIS::JobSpec",
            "properties": {
                "event_source": "support",
                "event_type": "ticket.Created",
                "flow_name": "zis:z3n-whiterose_first_test_webhook:flow:ticket_created",
                "name": "ticket_created_jobspec"
            }
        }
    }
}

###### Reading a bundle's ZIS::Flow
Each ZIS flow has a type, and properties eleements.  Properties is an object with a name and a definition.  The definition object has a "StartAt" element which identifies which state to start at.  The defiitaion also includes a states element which contains one or more states.  The states will be traversed by following the Amazon States Language (ASL) standard as defined here: https://states-language.net

The fully branching state machine is defined as a traversal from the StartAt state to each next state until end is true.  Branching conditions (such as the choice state) will provide multiple paths that might be traversed.





