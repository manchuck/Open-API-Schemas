# OAS Schemas references

How to have multiple representation of data in OAS

For this example, we will be following HAL.

## Schema Explanations

### User and Address

These schemas are used to validate the data heading into the database. It contains all the information that will be
stored on a user. This serves as our base, and we will reference it with the "child" schemas

```json
{
  "components": {
    "schemas": {
      "User": {
        "type": "object",
        "properties": {
          "user_id": {
            "description": "User II",
            "type": "string"
          },
          "first_name": {
            "type": "string",
            "description": "Users first name"
          },
          "last_name": {
            "type": "string",
            "description": "Users last name"
          },
          "full_name": {
            "type": "string",
            "description": "Users full name"
          },
          "email": {
            "type": "string",
            "description": "Users email address",
            "format": "email"
          },
          "profile": {
            "type": "string",
            "description": "Link to the profile picture"
          },
          "address": {
            "$ref": "#/components/schemas/Address"
          }
        }
      },
      "Address": {
        "type": "object",
        "additionalProperties": false,
        "required": [
          "country",
          "administrative_area",
          "locality",
          "postal_code",
          "thoroughfare"
        ],
        "description": "Address for the user",
        "properties": {
          "country": {
            "type": "string",
            "description": "Three Letter ISO country code"
          },
          "administrative_area": {
            "type": "string",
            "description": "State / Province / Region"
          },
          "sub_administrative_area": {
            "type": "string",
            "description": "County / District"
          },
          "locality": {
            "type": "string",
            "description": "City / Town"
          },
          "postal_code": {
            "type": "string",
            "description": "Postal Code / Zip Code"
          },
          "thoroughfare": {
            "type": "string",
            "description": "Street Address"
          },
          "premise": {
            "type": "string",
            "description": "Apartment / Suite / Box number etc"
          },
          "sub_premise": {
            "type": "string",
            "description": "Floor # / Room # / Building label etc"
          }
        }
      }
    }
  }
}
```

The reason for "Address" to be it's own schema since will not be returning it when ask for a list of all users.

### Hal Resources

Here we break out the response to allow for showing different representations of the resource depending on the route:

```json
{
  "components": {
    "schemas": {
      "HalLinks": {
        "type": "object",
        "description": "HAL Links",
        "properties": {
          "self": {
            "type": "object",
            "description": "Link to the resource",
            "required": [
              "href"
            ],
            "properties": {
              "href": {
                "type": "string",
                "format": "uri"
              }
            }
          },
          "next": {
            "type": "object",
            "description": "Link to the next resource",
            "required": [
              "href"
            ],
            "properties": {
              "href": {
                "type": "string",
                "format": "uri"
              }
            }
          },
          "prev": {
            "type": "object",
            "description": "Link to the previous resource",
            "required": [
              "href"
            ],
            "properties": {
              "href": {
                "type": "string",
                "format": "uri"
              }
            }
          },
          "profile": {
            "type": "object",
            "description": "Link to the users profile page",
            "required": [
              "href"
            ],
            "properties": {
              "href": {
                "type": "string",
                "format": "uri"
              }
            }
          }
        }
      },
      "HalUser": {
        "type": "object",
        "description": "User Response",
        "properties": {
          "_links": {
            "type": "object",
            "description": "HAL User Links",
            "allOf": [
              {
                "$ref": "#/components/schemas/HalLinks/properties/self"
              },
              {
                "$ref": "#/components/schemas/HalLinks/properties/profile"
              }
            ]
          }
        },
        "allOf": [
          {
            "$ref": "#/components/schemas/User/properties/user_id"
          },
          {
            "$ref": "#/components/schemas/User/properties/first_name"
          },
          {
            "$ref": "#/components/schemas/User/properties/last_name"
          },
          {
            "$ref": "#/components/schemas/User/properties/full_name"
          }
        ]
      },
      "HalUsers": {
        "type": "object",
        "description": "A Collection of Users",
        "properties": {
          "_embedded": {
            "type": "object",
            "properties": {
              "users": {
                "type": "array",
                "items": {
                  "$ref": "#/components/schemas/HalUser"
                }
              }
            }
          }
        }
      }
    }
  }
}
```

We pull the links out so we can easily reference the links. HalUser is the basic user representation

```json

{
  "schemas": {
    "components": {
      "HalUser": {
        "type": "object",
        "description": "User Response",
        "properties": {
          "_links": {
            "type": "object",
            "description": "HAL User Links",
            "allOf": [
              {
                "$ref": "#/components/schemas/HalLinks/properties/self"
              },
              {
                "$ref": "#/components/schemas/HalLinks/properties/profile"
              }
            ]
          }
        },
        "allOf": [
          {
            "$ref": "#/components/schemas/User/properties/user_id"
          },
          {
            "$ref": "#/components/schemas/User/properties/first_name"
          },
          {
            "$ref": "#/components/schemas/User/properties/last_name"
          },
          {
            "$ref": "#/components/schemas/User/properties/full_name"
          }
        ]
      }
    } 
  }
}
```

And the user as it would be in a collection

```json
{
  "schemas": {
    "components": {
      "HalUsers": {
        "type": "object",
        "description": "A Collection of Users",
        "properties": {
          "_embedded": {
            "type": "object",
            "properties": {
              "users": {
                "type": "array",
                "items": {
                  "$ref": "#/components/schemas/HalUser"
                }
              }
            }
          }
        }
      }
    }
  }
}
```

As you can see we are not showing the "address" for the user. When we are listing out users in `/users` there is no
need to show the address. How do we show the address when calling `/user/{user_id}`? Well we just reference the 
property in the response schema:

```json
{
  "paths": {
    "/users/{user_id}": {
      "get": {
        "operationId": "fetchUserById",
        "parameters": [
          {
            "name": "user_id",
            "in": "path",
            "description": "Id for the user",
            "required": true,
            "schema": {
              "description": "Unique identifier",
              "type": "string"
            },
            "style": "simple"
          }
        ],
        "responses": {
          "200": {
            "description": "A User HAL response",
            "content": {
              "application/hal+json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "address": {
                      "$ref": "#/components/schemas/Address"
                    }
                  },
                  "allOf": [
                    {
                      "$ref": "#/components/schemas/HalUser"
                    }
                  ]
                }
              }
            }
          }
        }
      }
    }
  }
}
```

And for showing what users look like when calling `/users`

```json
{
  "paths": {
    "/users": {
      "get": {
        "operationId": "fetchAllUsers",
        "summary": "Fetch Users",
        "responses": {
          "200": {
            "description": "",
            "content": {
              "application/hal+json": {
                "schema": {
                  "type": "object",
                  "required": [
                    "_embedded",
                    "_links",
                    "total_count",
                    "limit",
                    "offset"
                  ],
                  "properties": {
                    "total_count": {
                      "type": "number",
                      "description": "Total number of users"
                    },
                    "limit": {
                      "type": "number",
                      "description": "Limit of resources on the page"
                    },
                    "offset": {
                      "type": "string",
                      "description": "Next offset",
                      "nullable": true
                    },
                    "_embedded": {
                      "$ref": "#/components/schemas/HalUsers/properties/_embedded"
                    },
                    "_links": {
                      "type": "object",
                      "required": [
                        "self"
                      ],
                      "properties": {
                        "self": {
                          "$ref": "#/components/schemas/HalLinks/properties/self"
                        },
                        "next": {
                          "$ref": "#/components/schemas/HalLinks/properties/next"
                        },
                        "prev": {
                          "$ref": "#/components/schemas/HalLinks/properties/prev"
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```
