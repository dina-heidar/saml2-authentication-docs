
## Install 

The NuGet package is available to download at [nuget.org](https://www.nuget.org/packages/Saml2.Authentication)

``` csharp
dotnet add package Saml2.Authentication
```

## Setup

``` cs title="Program.cs" 

builder.Services.AddAuthentication(sharedOptions =>
{
    sharedOptions.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    sharedOptions.DefaultSignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    sharedOptions.DefaultChallengeScheme = Saml2Defaults.AuthenticationScheme;
})
.AddSaml2(options =>
{
    options.AuthenticationScheme = Saml2Defaults.AuthenticationScheme; //defaul is "Saml2"
    options.SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;

    //eg "https://idp.example.com/idp/metadata.xml
    options.MetadataAddress = "[identity_provider_url_metadata_address]" ;

    options.ForceAuthn = true; //optional
    options.VerifySignatureOnly = false;

    //these must match with sp metadata file
    options.EntityId = "[service_provider_entity_id_value]"; 
    options.RequireMessageSigned = false;
    options.WantAssertionsSigned = true; //default is false
    options.AuthenticationRequestSigned = true; //default is false            
    options.SigningCertificate = ["x509_certificate_that_signs_requests_to_idp"] //optional 
    options.EncryptingCertificate = ["x509_certificate_that_decrypts_incoming_saml_assertion"] //optional 

    //signin
    options.AuthenticationMethod = Saml2AuthenticationBehaviour.FormPost; //send saml requests as http post - front channel
    options.RequestedAuthnContext = RequestedAuthnContextTypes.FormsAuthentication(); //optional - use forms authentication
    options.ResponseProtocolBinding = Saml2ResponseProtocolBinding.FormPost; //send post back saml assertion          
    options.CallbackPath = new PathString("/saml2-signin"); //default is "saml2-signin"

    //logout
    options.SignOutPath = new PathString("/signedout"); //default is "saml2-signout"

    //to generate a metadata file
    //this will be created if there is no existing one
    //by default the metadata file will be placed in `wwwroot`
    //metadata creation items
    options.CreateMetadataFile = true; //default is `false`
    options.Metadata = new Saml2MetadataXml
    {
        CacheDuration = "360000", //optional
        ValidUntil = DateTime.UtcNow.AddDays(365), //optional
        Id = Guid.NewGuid().ToString(), //optional

        //optional
        ContactPersons = new ContactPerson
        {
            Company = "Example",
            ContactType = ContactType.Billing,
            EmailAddress = "jane.smith@example.com",
            TelephoneNumber = "123-234-1234",
            GivenName = "Smith",
            Surname = "Jane"
        },

        //optional
        Organization = new Organization
        {
            OrganizationDisplayName = "Example Enterprise",
            OrganizationName = "Department of Testing",
            OrganizationURL = new Uri("https://example.com"),
            Language = "en-US"
        },

        //optional
        // add an sp logo to the idp sign in page 
        UiInfo = new UiInfo
        {
            Language = "en-US",
            DisplayName = "Example site",
            Description = "Example site",
            InformationURL = new Uri("https://example.com"),
            PrivacyStatementURL = new Uri("https://example.com/privacy"),
            LogoHeight = "12",
            LogoWidth = "24",
            LogoUriValue = new Uri("https://example.comv/logo.png"),
            KeywordValues = new[] { "set", "ready", "go" }
        },

        //optional
        AttributeConsumingService = new AttributeConsumingService
        {
            ServiceDescriptions = "testing service",
            ServiceNames = "primary",
            RequestedAttributes = new RequestedAttribute[]
            {
                new RequestedAttribute
                {
                    Name ="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
                    NameFormat = "urn:oasis:names:tc:SAML:2.0:attrname-format:uri",
                    FriendlyName = "E-Mail Address",
                    IsRequiredField= true
                },
                new RequestedAttribute
                {
                    Name ="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname",
                    NameFormat = "urn:oasis:names:tc:SAML:2.0:attrname-format:uri",
                    FriendlyName = "Surname",
                    IsRequiredField= true
                },
                new RequestedAttribute
                {
                    Name ="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname",
                    NameFormat = "urn:oasis:names:tc:SAML:2.0:attrname-format:uri",
                    FriendlyName = "Given Name",
                    IsRequiredField= true
                }
            }
        },

        //optional 
        //sign the entire metadata file
        Signature = ["signing_x509_certificate"]
    };
    
    
    options.Events.OnTicketReceived = context =>
    {
        return Task.FromResult(0);
    };
    options.Events.OnRemoteFailure = context =>
    {
        return Task.FromResult(0);
    };
})
.AddCookie();

builder.Services.AddAuthorization();

```

### Required options

