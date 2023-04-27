
## Install 

The NuGet package is available to download at [nuget.org](https://www.nuget.org/packages/Saml2.Authentication)

``` csharp
dotnet add package Saml2.Authentication
```
## Setup

The code below will set up the saml2 options and will add authentication middleware to the request pipeline.

!!! warning

    If you have an existing metadata file, please make sure the folwwing values match with the values in the Saml2 options.   

### Mapping existing metadata file data (skip if this does not apply)  

Replace the Saml2 option value with the actual values in your Service Provider (SP) metadata file. The following is an example:

???+ "EntityId"

    &lt;EntityDescriptor xmlns="urn:oasis:names&#58;tc:SAML:2.0:metadata" ID="SM21ad0d447a0ff34a5d6d1d36893ae472426cb9f6d7" {==entityID="example.sp.com"==}&gt; 
    
    ``` cs title="Saml2 option values"
    options.EntityId = "example.sp.com"
    ```
   
???+ "Assertion Consumer Service"

     &lt;AssertionConsumerService Binding="urn:oasis:names&#58;tc:SAML:2.0:bindings:{==HTTP-POST==}" Location="https://localhost:5001/{==saml2-signin==}"  index="0" isDefault="true" /&gt;

    ``` cs title="Saml2 option values"
    options.ResponseProtocolBinding = Saml2ResponseProtocolBinding.FormPost //this corresponds the HTTP-POST value
    options.CallbackPath = new PathString("saml2-signin")
    ```

???+ Login

     &lt;SPSSODescriptor protocolSupportEnumeration="urn:oasis:names&#58;tc:SAML:2.0:protocol" {==AuthnRequestsSigned="true"==} {==WantAssertionsSigned="true"==}&gt;

    ``` cs title="Saml2 option values"
    options.AuthenticationRequestSigned = "true"
    options.WantAssertionsSigned  ="true"
    ```

???+ Logout

     &lt;SingleLogoutService Binding="urn:oasis:names&#58;tc:SAML:2.0:bindings:{==HTTP-POST==}" Location="https://localhost:5001/{==signedout==}"&gt;

    ``` cs title="Saml2 option values"
    options.ResponseLogoutBinding = Saml2ResponseLogoutBinding.FormPost //this corresponds the HTTP-POST value
    options.SignOutPath = new PathString("/signedout")
    ```

???+ "NameIDFormat"

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

    The signing amd encryption certificates must be the same ones used when the metadata file was created. 


## Configuration


=== "Basic"

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

        //eg "https://idp.example.com/idp/metadata.xml
        options.MetadataAddress = "[identity_provider_url_metadata_address]" ;
        options.EntityId = "[service_provider_entity_id_value]"; 
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

=== "Rich"

    ``` cs title="Program.cs" 

    builder.Services.AddAuthentication(sharedOptions =>
    {
        sharedOptions.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        sharedOptions.DefaultSignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        sharedOptions.DefaultChallengeScheme = Saml2Defaults.AuthenticationScheme;
    })
    .AddSaml2(options =>
    {
        options.AuthenticationScheme = Saml2Defaults.AuthenticationScheme; //default is "Saml2"
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

        //signin (samlRequest)
        options.AuthenticationMethod = Saml2AuthenticationBehaviour.FormPost; //send saml requests as http post - front channel
        options.RequestedAuthnContext = RequestedAuthnContextTypes.FormsAuthentication(); //optional - use forms authentication
        options.ResponseProtocolBinding = Saml2ResponseProtocolBinding.FormPost; //Idp post back saml assertion          
        options.CallbackPath = new PathString("/saml2-signin"); //default is "saml2-signin"

        // certificate used to sign the saml request        
        options.SigningCertificate = ["x509_certificate_that_signs_requests_to_idp"] //optional 

        // certificate used to decrypt the incoming saml assertion
        options.EncryptingCertificate = ["x509_certificate_that_decrypts_incoming_saml_assertion"] //optional 
       
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
            //the certificate used to sign the entire metadata file
            Signature = ["signing_x509_certificate"]
        };    
    
        //events
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

    var app = builder.Build();

    //code 

    //UseAuthentication adds authentication middleware to the request pipeline.
    app.UseAuthentication();
    app.UseAuthorization();

    //code 

    app.Run();
    ```



