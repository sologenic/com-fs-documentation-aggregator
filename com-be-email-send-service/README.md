# Email send service

This service is uncommon since it for internal use by other services, instead of the usual end user or admin facing http interface.
The service provides a gRPC endpoint for sending templated emails.

## Usage

### Sending an Email

To send an email, make a gRPC call to the `Send` endpoint with a `SendEmailRequest`:

```protobuf
message SendEmailRequest {
    string RecipientEmail = 1;
    string RecipientName = 2;
    emailtemplate.EmailTemplateType EmailTemplateType = 3;
    string OrganizationID = 4;
    metadata.Network Network = 5;
    optional language.Language Language = 6;
}
```

The service will:
1. Fetch the appropriate template from the email template service
2. Check for organization-specific custom templates
3. Apply translations if needed
4. Populate template data based on the template type
5. Send the email via SendGrid

### Template Data

Template data is handled through the `EmailTemplateDataRegistry` from the email template model. Each template type has a corresponding data structure defined in the registry.

For example, for KYC email templates:

```go
type KYCEmailData struct {
    UserName        string
    RejectionReason string
    ExternalUserID  string
    AccountID       string
    ClientComment   string
    AdminComment    string
}
```

### Language Support

The service supports email content translation based on the following conditions:
1. The template is not a custom organization template (custom template means the translation cannot be found, but is most likely already handled in the custom template)
2. A valid language is specified (not `LANG_NOT_USED`)
3. The language is not `ENGLISH`

### Adding New Template Types

To add support for new template types:

1. Define the template type in `com-fs-email-template-model/domain/emailtemplatedmn.go`
2. Register the new type in `EmailTemplateDataRegistry`

For detailed instructions on adding new template types, refer to the [email template model documentation](../com-fs-email-template-model/README.md).

## Application startup requirements

- `GRPC_PORT` - The port the gRPC server listens on
- `SENDGRID_CONFIG` - SendGrid API key for sending emails
- `EMAIL_TEMPLATE_STORE` - The email template grpc endpoint, see `github.com/sologenic/com-fs-email-template-model`
- `NOTIFICATION_PARSER_LIB` - The translation grpc endpoint, see `github.com/sologenic/com-fs-notification-translation-model`
- `GRPC_APPEND` - The suffix to append to gRPC service names
