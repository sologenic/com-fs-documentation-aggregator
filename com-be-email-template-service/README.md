# Email Template Service

The email template service provides the following RESTFUL interface:

### System Email Templates Management(Sologenic admin only)

**Authenticated:**

* GET `/api/emailtemplate/system/get?type=...`: Retrieves an email template
* GET `/api/emailtemplate/system/list`: Retrieves a list of email templates
* POST `/api/emailtemplate/system/upsert`: Upsert a system email template
* DELETE `/api/emailtemplate/system/delete`: Deletes a system email template

### Organization Email Templates Management(Organization admin only by default)

**Authenticated:**

* GET `/api/emailtemplate/get`: Retrieves an email template
* GET `/api/emailtemplate/list`: Retrieves a list of email templates
* PUT `/api/emailtemplate/upsert`: Upsert an custom email template
* POST `/api/emailtemplate/reset`: Resets an custom email template to system(removes the custom template)

> **Note:** 
> - All admin endpoints are protected by role-based access control and default to the`ORGANIZATION_ADMINISTRATOR` role. Permissions are managed dynamically by Organization administrators on the fly.
> - If the organization-network specific template is not found, the system template is returned.

## GET /api/emailtemplate/system/get?type=...

(Sologenic admin only)

Retrieves a system email template by type

### Example call

```bash
curl -X GET \
  https://comsolotex-admin.sologenic.org/api/emailtemplate/system/get?type=1 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ..." \
  -H "Network: mainnet"
```

### Example response

```json
{
  "EmailTemplate": {
    "Type": 1,
    "OrganizationID": "",
    "Name": "System KYC Approval Template",
    "Subject": "Your KYC Verification is Approved",
    "HTML": "<!DOCTYPE html>\n<html>\n<head>\n    <meta charset=\"UTF-8\">\n    <title>User Verification Completed</title>\n    <style>...</style>\n</head>\n<body>\n    <div class=\"container\">\n        <div class=\"header\">\n            <h2>Sologenic KYC Verification</h2>\n        </div>\n        <div class=\"content\">\n            <h1>Hello, {{.UserName}}!</h1>\n            <p>Your KYC verification has been successfully completed. Thank you for choosing our service.</p>\n            <p>If you have any questions, feel free to contact us at support@sologenic.org.</p>\n        </div>\n        <div class=\"footer\">\n            <p>Best regards,<br/>The Sologenic Team</p>\n        </div>\n    </div>\n</body>\n</html>",
    "Description": "System template sent when a user's KYC is approved",
    "CreatedAt": {
      "seconds": 1740518732,
      "nanos": 171972000
    },
    "UpdatedAt": {
      "seconds": 1740518732,
      "nanos": 171974000
    },
    "Network": 0
  },
  "Audit": {
    "ChangedBy": "admin@sologenic.org",
    "ChangedAt": {
      "seconds": 1740518732,
      "nanos": 171974000
    },
    "Reason": "Initial template creation"
  }
}
```

## GET /api/emailtemplate/system/list

(Sologenic admin only)

Returns a list of all assets for admins

### Example call

```bash
curl -X GET \
  https://comsolotex-admin.sologenic.org/api/emailtemplate/system/list \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer: ...." \
  -H "Network: testnet"
```

### Example response

```json
{
    "EmailTemplates": [
        {
            "EmailTemplate": {
                "Type": 1,
                "OrganizationID": "",
                "Name": "System KYC Approval Template",
                "Subject": "Your KYC Verification is Approved",
                "HTML": "<!DOCTYPE html>\n...",
                "Description": "System template sent when a user's KYC is approved",
                "CreatedAt": {
                    "seconds": 1741636052,
                    "nanos": 876859000
                },
                "UpdatedAt": {
                    "seconds": 1741636052,
                    "nanos": 876860000
                },
                "Network": 0
            },
            "Audit": {
                "ChangedBy": "sg.be.test@gmail.com",
                "ChangedAt": {
                    "seconds": 1741636052,
                    "nanos": 876860000
                },
                "Reason": ""
            }
        },
        {
            "EmailTemplate": {
                "Type": 100,
                "OrganizationID": "",
                "Name": "System Organization Onboarding Template",
                "Subject": "Welcome to Solotex - Your Organization is Ready",
                "HTML": "<!DOCTYPE html>\n..",
                "Description": "System welcome email for new organizations onboarded to Solotex",
                "CreatedAt": {
                    "seconds": 1741634227,
                    "nanos": 863946000
                },
                "UpdatedAt": {
                    "seconds": 1741634227,
                    "nanos": 863947000
                },
                "Network": 0
            },
            "Audit": {
                "ChangedBy": "sg.be.test@gmail.com",
                "ChangedAt": {
                    "seconds": 1741634227,
                    "nanos": 863947000
                },
                "Reason": ""
            }
        },
        {
            "EmailTemplate": {
                "Type": 2,
                "OrganizationID": "",
                "Name": "System KYC Rejection Template",
                "Subject": "Your KYC Verification Status",
                "HTML": "<!DOCTYPE html>\n...",
                "Description": "System template sent when a user's KYC is rejected",
                "CreatedAt": {
                    "seconds": 1741636975,
                    "nanos": 185997000
                },
                "UpdatedAt": {
                    "seconds": 1741636975,
                    "nanos": 185998000
                },
                "Network": 0
            },
            "Audit": {
                "ChangedBy": "sg.be.test@gmail.com",
                "ChangedAt": {
                    "seconds": 1741636975,
                    "nanos": 185998000
                },
                "Reason": ""
            }
        }
    ]
}
```

## POST /api/emailtemplate/system/upsert

Creates a new system email template or updates an existing one

> **Note**: 
> - For creating a new template, there are some prerequisites to be followed. Please refer to the [Creating New System Email Template](#email-template-prerequisites) in `README.md` of `com-fs-email-template-model` for more details.

### Example call

```bash
curl -X POST \
  https://comsolotex-admin.sologenic.org/api/emailtemplate/system/upsert \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ..." \
  -H "Network: testnet" \
  -d '{
  "EmailTemplate": {
    "Type": 1,
    "Name": "System KYC Approval Template",
    "Subject": "Your KYC Verification is Approved",
    "Description": "System template sent when a user's KYC is approved",
    "HTML": "<!DOCTYPE html>\n<html>\n<head>\n    <meta charset=\"UTF-8\">\n    <title>User Verification Completed</title>\n    <style>\n        body {\n            font-family: Arial, sans-serif;\n            background-color: #f4f4f4;\n            margin: 0;\n            padding: 0;\n        }\n        .container {\n            width: 100%;\n            max-width: 600px;\n            margin: 0 auto;\n            background-color: #ffffff;\n            padding: 20px;\n            border-radius: 10px;\n            box-shadow: 0 0 10px #0000001a;\n        }\n        .header {\n            background-color: #007bff;\n            color: #ffffff;\n            padding: 10px;\n            border-radius: 10px 10px 0 0;\n            text-align: center;\n        }\n        .content {\n            padding: 20px;\n        }\n        .content h1 {\n            color: #333333;\n        }\n        .content p {\n            color: #555555;\n        }\n        .footer {\n            text-align: center;\n            color: #777777;\n            padding: 10px;\n            border-top: 1px solid #eeeeee;\n        }\n        .footer a {\n            color: #007bff;\n            text-decoration: none;\n        }\n        .button {\n            display: inline-block;\n            padding: 10px 20px;\n            margin-top: 20px;\n            background-color: #007bff;\n            color: #ffffff;\n            text-align: center;\n            border-radius: 5px;\n            text-decoration: none;\n            font-size: 16px;\n        }\n    </style>\n</head>\n<body>\n    <div class=\"container\">\n        <div class=\"header\">\n            <h2>Sologenic KYC Verification</h2>\n        </div>\n        <div class=\"content\">\n            <h1>Hello, {{.UserName}}!</h1>\n            <p>Your KYC verification has been successfully completed. Thank you for choosing our service.</p>\n            <p>If you have any questions, feel free to contact us at <a href=\"mailto:support@sologenic.org\">support@sologenic.org</a>.</p>\n            <a href=\"https://sologenic.org\" class=\"button\">Visit Our Website</a>\n        </div>\n        <div class=\"footer\">\n            <p>Best regards,<br/>The Sologenic Team</p>\n            <p>&copy; 2024 Sologenic. All rights reserved.</p>\n        </div>\n    </div>\n</body>\n</html>"
  },
  "Audit": {
    "Reason": "initial template creation"
  }
}'
```

## DELETE /api/emailtemplate/system/delete

(Sologenic admin only)

Deletes a system email template

### Example call

```bash
curl -X DELETE \
  https://comsolotex-admin.sologenic.org/api/emailtemplate/system/delete \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ..." \
  -H "Network: mainnet" \
  -d '{
  "EmailTemplate": {
    "Type": 1,
    "Name": "System KYC Approval Template",
    "Subject": "Your KYC Verification is Approved",
    "Description": "System template sent when a user's KYC is approved",
    "HTML": "<!DOCTYPE html>..."
  }
}'
```

## GET /api/emailtemplate/get?type=1

(Organization admin only)

Retrieves an organization-specific email template by type

### Example call

```bash
curl -X GET \
  https://comsolotex-admin.sologenic.org/api/emailtemplate/get?type=1 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ..." \
  -H "Network: testnet"
```

### Example response

```json
{
  "EmailTemplate": {
    "Type": 1,
    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
    "Network": 2,
    "Name": "Custom KYC Approval Template",
    "Subject": "Your KYC Verification is Complete",
    "HTML": "<!DOCTYPE html>...",
    "Description": "Organization-specific template sent when a user's KYC is approved",
    "CreatedAt": {
      "seconds": 1740519032,
      "nanos": 471972000
    },
    "UpdatedAt": {
      "seconds": 1740519032,
      "nanos": 471974000
    }
  },
  "Audit": {
    "ChangedBy": "org-admin@example.com",
    "ChangedAt": {
      "seconds": 1740519032,
      "nanos": 471974000
    }
  }
}
```

## GET /api/emailtemplate/list

(Organization admin only)

Retrieves all email templates for an organization

### Example call

```bash
curl -X GET \
  https://comsolotex-admin.sologenic.org/api/emailtemplate/list \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ..." \
  -H "Network: testnet"
```

### Example response

```json
{
    "EmailTemplates": [
                {
            "EmailTemplate": {
                "Type": 1,
                "OrganizationID": "215a551d-5691-91ce-f4a6-9284f40d1340",
                "Name": "KYC Approval Template - Org custom 2",
                "Subject": "Your KYC Verification is Approved",
                "HTML": "<!DOCTYPE html>\n...",
                "Description": "Custom: System template sent when a user's KYC is approved",
                "CreatedAt": {
                    "seconds": 1741641911,
                    "nanos": 741847000
                },
                "UpdatedAt": {
                    "seconds": 1741641911,
                    "nanos": 741848000
                },
                "Network": 1
            },
            "Audit": {
                "ChangedBy": "sg.be.org.test@gmail.com",
                "ChangedAt": {
                    "seconds": 1741641911,
                    "nanos": 741849000
                },
                "Reason": ""
            }
        },
        {
            "EmailTemplate": {
                "Type": 100,
                "OrganizationID": "",
                "Name": "System Organization Onboarding Template",
                "Subject": "Welcome to Solotex - Your Organization is Ready",
                "HTML": "<!DOCTYPE html>\n...",
                "Description": "System welcome email for new organizations onboarded to Solotex",
                "CreatedAt": {
                    "seconds": 1741634227,
                    "nanos": 863946000
                },
                "UpdatedAt": {
                    "seconds": 1741634227,
                    "nanos": 863947000
                },
                "Network": 0
            },
            "Audit": {
                "ChangedBy": "sg.be.test@gmail.com",
                "ChangedAt": {
                    "seconds": 1741634227,
                    "nanos": 863947000
                },
                "Reason": ""
            }
        },
        {
            "EmailTemplate": {
                "Type": 2,
                "OrganizationID": "",
                "Name": "System KYC Rejection Template",
                "Subject": "Your KYC Verification Status",
                "HTML": "<!DOCTYPE html>\n...",
                "Description": "System template sent when a user's KYC is rejected",
                "CreatedAt": {
                    "seconds": 1741636975,
                    "nanos": 185997000
                },
                "UpdatedAt": {
                    "seconds": 1741636975,
                    "nanos": 185998000
                },
                "Network": 0
            },
            "Audit": {
                "ChangedBy": "sg.be.test@gmail.com",
                "ChangedAt": {
                    "seconds": 1741636975,
                    "nanos": 185998000
                },
                "Reason": ""
            }
        }
    ]
}
```

## POST /api/emailtemplate/upsert

(Organization admin only)

Creates or updates an organization-specific email template.

### Example call

```bash
curl -X POST \
  https://comsolotex-admin.sologenic.org/api/emailtemplate/upsert \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ..." \
  -H "Network: testnet" \
  -d '{fic
  "EmailTemplate": {
    "Type": 1,
    "Name": "Custom KYC Approval Template",
    "Subject": "Your KYC Verification is Complete",
    "Description": "Organization-specific template sent when a user's KYC is approved",
    "HTML": "<!DOCTYPE html>\n<html>\n<head>\n    <meta charset=\"UTF-8\">\n    <title>Your KYC is Approved</title>\n    <style>...</style>\n</head>\n<body>\n    <div class=\"container\">\n        <div class=\"header\">\n            <h2>KYC Veriation Complete</h2>\n        </div>\n        <div class=\"content\">\n            <h1>Hello, {{.UserName}}!</h1>\n            <p>Your KYC verification has been approved. You may now access all platform features.</p>\n        </div>\n        <div class=\"footer\">\n            <p>Best regards,<br/>The Example Company Team</p>\n        </div>\n    </div>\n</body>\n</html>",
  }
}'
```

## POST /api/emailtemplate/reset

(Organization admin only)

Resets an organization-specific email template to the system template. (removes the custom template)

### Example call

```bash
curl -X DELETE \
  https://comsolotex-admin.sologenic.org/api/emailtemplate/reset \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ..." \
  -H "Network: mainnet" \
  -d '{
  "EmailTemplate": {
    "Type": 1,
    "OrganizationID": "215a551d-5691-91ce-f4a6-9284f40d1340",
    "Name": "Custom KYC Approval Template",
    "Subject": "Your KYC Verification is Approved",
    "Description": "Custom template sent when a user's KYC is approved",
    "HTML": "<!DOCTYPE html>..."
  }
}'
```

## Start parameters

The following start parameters are required:

* `ACCOUNT_STORE`: the account store endpoint (included from `github.com/sologenic/com-be-admin-account-store/`)
* `ROLE_STORE`: the role store endpoint (included from `github.com/sologenic/com-be-admin-role-store/`)
* `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/fs-feature-flag-model/`)
* `AUTH_FIREBASE_SERVICE`: the firebase authentication service endpoint
* `HTTP_CONFIG`: the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
* `EMAIL_TEMPLATE_STORE`: the email template store endpoint (included from `github.com/sologenic/com-be-email-template-store/`)
* `ORGANIZATION_STORE`: Included from github.com/sologenic/com-fs-organization-model. For setup see github.com/sologenic/com-fs-organization-model/client/README.
