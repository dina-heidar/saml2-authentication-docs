
## Install 

The NuGet package is available to download at [nuget.org](https://www.nuget.org/packages/Saml2.Authentication)

``` csharp
dotnet add package Saml2.Authentication
```
## Setup

The code below will set up the saml2 options and will add authentication middleware to the request pipeline.

!!! warning


    If you have an existing metadata file, go to [Existing metadata](#existing-metadata) section to set the Saml option values to the existing metadata values.

    If the metadata file is unavailabe, please contact the Identity Provider (IdP) admininstrator to request the configured values.

## Configuration

Below is the minimal configuration needed to send a signed or unsigned Saml request. 

!!! important 

    When `options.AuthenticationRequestSigned=true`, all outgoing authentication requests will be signed. The `options.SigningCertificate`  certificate (`*.pfx`) must be provided at this point.

    If the IdP has an existing encrypting certificate for Sp (your application), then the `options.EncryptingCertificate` certificate 
    (`*.pfx`) must be provided.

=== "Unsigned requests"

    ``` cs title="Program.cs" 
    
    builder.Services.AddAuthentication(sharedOptions =>
    {
        sharedOptions.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        sharedOptions.DefaultSignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        sharedOptions.DefaultChallengeScheme = Saml2Defaults.AuthenticationScheme;
    })
    .AddSaml2(options =>
    {
        options.SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;

        //the IdP metadata address (e.g. "https://idp.example.com/idp/metadata.xml)
        options.MetadataAddress = "[identity_provider_url_metadata_address]" ;

        //these must match with an existing sp metadata file (if it exists)
        options.EntityId = "[service_provider_entity_id_value]";         
        options.RequireMessageSigned = false; //IdP sign incoming entire saml response - default is false
        options.WantAssertionsSigned = false; //IdP sign incoming  assertion (claims) section - default is false
        options.AuthenticationRequestSigned = false; //sign outgoing authentication request - default is false
        options.ResponseProtocolBinding = Saml2ResponseProtocolBinding.FormPost; //Idp post back saml response - default is Http-Post        
        options.CallbackPath = new PathString("/saml2-signin"); //the middleware endpoint where the response receive the saml response - default is "saml2-signin"
        options.SignOutPath = new PathString("/signedout"); //the middleware endpoint where the IdP will send the logout response - default is "saml2-signout"
    }
    .AddCookie();

    builder.Services.AddAuthorization();

    var app = builder.Build();

    //code 

    //UseAuthentication adds authentication middleware to the request pipeline.
    app.UseAuthentication();
    app.UseAuthorization();

    //code 

    app.Run();
    ```

=== "Signed requests and decrypting"

    ``` cs title="Program.cs" 

    builder.Services.AddAuthentication(sharedOptions =>
    {
        sharedOptions.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        sharedOptions.DefaultSignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        sharedOptions.DefaultChallengeScheme = Saml2Defaults.AuthenticationScheme;
    })
    .AddSaml2(options =>
    {   
        options.SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;

        //the IdP metadata address (e.g. "https://idp.example.com/idp/metadata.xml)
        options.MetadataAddress = "[identity_provider_url_metadata_address]" ;

        //these must match with an existing sp metadata file (if it exists)
        options.EntityId = "[service_provider_entity_id_value]";         
        options.RequireMessageSigned = false; //IdP sign incoming entire saml response - default is false
        options.WantAssertionsSigned = true; //IdP sign incoming  assertion (claims) section - default is false
        options.AuthenticationRequestSigned = true; //sign outgoing authentication request - default is false
        options.ResponseProtocolBinding = Saml2ResponseProtocolBinding.FormPost; //Idp post back saml response - default is Http-Post        
        options.CallbackPath = new PathString("/saml2-signin"); //the middleware endpoint where the response receive the saml response - default is "saml2-signin"
        options.SignOutPath = new PathString("/signedout"); //the middleware endpoint where the IdP will send the logout response - default is "saml2-signout"
        
        // certificate used to sign the saml request        
        options.SigningCertificate = ["x509_certificate_that_signs_requests_to_idp"] 

        // certificate used to decrypt the incoming saml assertion
        options.EncryptingCertificate = ["x509_certificate_that_decrypts_incoming_saml_assertion"] 
    })
    .AddCookie();

    builder.Services.AddAuthorization();

    var app = builder.Build();

    //code 

    //UseAuthentication adds authentication middleware to the request pipeline.
    app.UseAuthentication();
    app.UseAuthorization();

    //code 

    app.Run();
    ```

### Additional options (optional)

For a complete list of options, go to [List of options](list-of-options.md)

#### Force sign-in every time
```cs title="Program.cs"
options.ForceAuthn = true; 
```

#### Override default sign-in form

```cs title="Program.cs"
options.RequestedAuthnContext = RequestedAuthnContextTypes.FormsAuthentication(); //use forms authentication
```
#### Generate a metadata file

```cs title="Program.cs"
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
    //the certificate used to sign the entire metadata file
    Signature = ["signing_x509_certificate"]
};  

```  

#### Events

```cs title="Program.cs"
//events
options.Events.OnTicketReceived = context =>
{
    return Task.FromResult(0);
};
options.Events.OnRemoteFailure = context =>
{
    return Task.FromResult(0);
};
```

## Existing metadata

Replace the Saml2 option value with the actual values in your Service Provider (SP) metadata file. The following is an example:

???+ "EntityId"

    &lt;EntityDescriptor xmlns="urn:oasis:names&#58;tc:SAML:2.0:metadata" ID="SM21ad0d447a0ff34a5d6d1d36893ae472426cb9f6d7" {==entityID="example.sp.com"==}&gt; 
    
    ``` cs title="Saml2 option values"
    options.EntityId = "example.sp.com"
    ```
   
???+ "ResponseProtocolBinding and CallbackPath"

     &lt;AssertionConsumerService Binding="urn:oasis:names&#58;tc:SAML:2.0:bindings:{==HTTP-POST==}" Location="https://localhost:5001/{==saml2-signin==}"  index="0" isDefault="true" /&gt;

    ``` cs title="Saml2 option values"
    options.ResponseProtocolBinding = Saml2ResponseProtocolBinding.FormPost //this corresponds the HTTP-POST value
    options.CallbackPath = new PathString("saml2-signin")
    ```

???+ "AuthenticationRequestSigned and WantAssertionsSigned"

     &lt;SPSSODescriptor protocolSupportEnumeration="urn:oasis:names&#58;tc:SAML:2.0:protocol" {==AuthnRequestsSigned="true"==} {==WantAssertionsSigned="true"==}&gt;

    ``` cs title="Saml2 option values"
    options.AuthenticationRequestSigned = "true"
    options.WantAssertionsSigned  ="true"
    ```

???+ "ResponseLogoutBinding and SignOutPath"

     &lt;SingleLogoutService Binding="urn:oasis:names&#58;tc:SAML:2.0:bindings:{==HTTP-POST==}" Location="https://localhost:5001/{==signedout==}"&gt;

    ``` cs title="Saml2 option values"
    options.ResponseLogoutBinding = Saml2ResponseLogoutBinding.FormPost //this corresponds the HTTP-POST value
    options.SignOutPath = new PathString("/signedout")
    ```

???+ "NameIdPolicy"

     &lt;NameIDFormat&gt;urn:oasis:names&#58;tc:SAML:2.0:nameid-format:{==persistent==}&lt;NameIDFormat&gt;

    ``` cs title="Saml2 option values"
    options.NameIdPolicy = new NameIdPolicy
    {
        Format = NameIDFormats.Persistent //this corresponds the value
        SpNameQualifier = null,
        AllowCreate = true
    };
    ```

???+ Certificates

    The `options.Signing` and `options.Encryption` certificate values must be the same ones set in the metadata signing and/or encryption section when the metadata file was created. 
