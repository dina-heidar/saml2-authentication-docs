## Options

The following are the list of available Saml2 options:

<table>
    <thead>
        <tr>
            <th>Option</th>
            <th>Description</th>
            <th>Value type</th>
            <th>Requirement</th>
            <th>Default value</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><h3>AssertionConsumerServiceIndex</h3></td>
            <td>Gets or sets the index value of the assertion consumer service. If this is populated, it will override the Callback, AssertionConsumerServiceUrl and the ResponseProtocolBinding values. Only the 'AssertionConsumerServiceIndex' will be sent for the AuthnRequest.</td>
            <td><code>ushort</code></td>
            <td>optional</td>
            <td><code>null</code></td>
        </tr>
        <tr>
            <td><h3>AssertionConsumerServiceUrl</h3></td>
            <td>Gets or sets the assertion consumer service URL value. This will override the CallBack value. If the <code>AssertionConsumerServiceIndex</code> is populated, the Callback, AssertionConsumerServiceUrl and the ResponseProtocolBinding values will not be sent within the AuthnRequest.</td>
            <td><code>Uri</code></td>
            <td>optional</td>
            <td><code></code></td>
        </tr>
        <tr>
            <td><h3>AuthenticationMethod</h3></td>
            <td>Gets or sets the authentication method value indicating whether [authentication request signed].The default value is <code>HTTP-Redirect</code>.</td>
            <td><code>Saml2AuthenticationBehaviour</code></td>
            <td>**required** _(already set by default)_</td>
            <td><code>Saml2AuthenticationBehaviour.RedirectGet</code></td>
        </tr>
    </tbody>
</table>

### AuthenticationMethod

| Value type                     | Requirement                            | Default value                              | 
| -----------                    | ------------------------------------   |-----------                                 |
| `Saml2AuthenticationBehaviour` | **required** _(already set by default)_|  `Saml2AuthenticationBehaviour.RedirectGet`|

Gets or sets the authentication method value indicating whether [authentication request signed].The default value is 'HTTP-Redirect'

### AuthenticationScheme	

| Value type    | Requirement                            | Default value   | 
| -----------   | ------------------------------------   |-----------      |
| `string`      | **required** _(already set by default)_|  `Saml2`        |

Gets or sets the authentication scheme. A different value may be assigned in order to use the same authentication middleware type more than once in a pipeline. 

### AutomaticAuthenticate	
_(Inherited from AuthenticationOptions)_</br>
If true the authentication middleware alter the request user coming in. If false the authentication middleware will only provide identity when explicitly indicated by the AuthenticationScheme.

### AutomaticChallenge	
_(Inherited from AuthenticationOptions)_</br>
If true the authentication middleware should handle automatic challenge. If false the authentication middleware will only alter responses when explicitly indicated by the AuthenticationScheme.

### BackchannelHttpHandler
The HttpMessageHandler used to communicate with remote identity provider. This cannot be set at the same time as BackchannelCertificateValidator unless the value can be downcast to a WebRequestHandler.

### BackchannelTimeout	
Gets or sets timeout value in milliseconds for back channel communications with the remote identity provider.

### CallbackPath	

| Value type    | Requirement                            | Default value   | 
| -----------   | ------------------------------------   |-----------      |
| `PathString`  | **required** _(already set by default)_| `/saml2-signin` |

The request path within the application's base path where the user-agent will be returned. The middleware will process this request when it arrives.

### ClaimsIssuer	
_(Inherited from AuthenticationOptions)_</br>
Gets or sets the issuer that should be used for any claims that are created


### CreateMetadataFile

| Value type    | Requirement                            | Default value   | 
| -----------   | ------------------------------------   |-----------      |
| `bool`        | optional                               | `false`         |

Gets or sets a value indicating whether to create metadata file or not. If set to "true" and there is no existing metadata file 
in the `DefaultMetadataFolderLocation` then the middleware will create the meetadata.xml file. 

### CookieConsentNeeded 

| Value type    | Requirement                            | Default value   | 
| -----------   | ------------------------------------   |-----------      |
| `bool`        | **required** _(already set by default)_| `true`          |

Gets or sets the cookie consent as essential or not it overrides the Cookie policy set. This is needed when signing in. 


### DefaultMetadataFolderLocation

| Value type    | Requirement                            | Default value   | 
| -----------   | ------------------------------------   |-----------      |
| `string`      | optional _(already set by default)_    | `wwwroot`       |

Gets or sets the default metadata file location. 


### DefaultMetadataFileName

| Value type    | Requirement                            | Default value   | 
| -----------   | ------------------------------------   |-----------      |
| `string`      | optional _(already set by default)_    | `Metadata.xml`  |

Gets or sets the default name of the metadata file. The default value of this folder is "Metadata.xml".

### Description	
_(Inherited from AuthenticationOptions)_</br>
Additional information about the authentication type which is made available to the application.

### DisplayName	

| Value type    | Requirement                            | Default value   | 
| -----------   | ------------------------------------   |-----------      |
| `string`      | optional                               | null            |

Get or sets the text that the user can display on a sign in user interface.

### DefaultRedirectUrl

| Value type    | Requirement                            | Default value   | 
| -----------   | ------------------------------------   |-----------      |
| `PathString`  | optional                               |  `/`            |

Gets or sets the default redirect URL. This URL is used by the SP to redirect the user back to after they log out.


### EncryptingCertificate

| Value type            | Requirement                            | Default value   | 
| -----------           | ------------------------------------   |-----------      |
| `X509Certificate2`    | optional                               |  `null`         |

Gets or sets the encrypting certificate. This is used to decrypt the encrypted assertion. Only RSA is supported.

### EntityId

| Value type            | Requirement                            | Default value   | 
| -----------           | ------------------------------------   |-----------      |
| `string`              | **required**                           |  `null`         |

Gets or sets the service provider entity identifier.

### ForceAuthn

| Value type    | Requirement                            | Default value   | 
| -----------   | ------------------------------------   |-----------      |
| `bool`        | optional _(already set by default)_    |  `false`        |

Gets or sets a value indicating whether authentication is required for every request.

### IdpSingleSignOnServiceLocationIndex 

| Value type    | Requirement     | Default value   | 
| -----------   | --------------  |-----------      |
| `ushort`      | optional        |  `null`         |

Gets or sets the location of the IdP single sign on service index. if `null`, the first SignleSignOnService location with configured 
protocol binding will be used.

### IdpSingleLogoutServiceLocationIndex

| Value type    | Requirement     | Default value   | 
| -----------   | --------------  |-----------      |
| `ushort`      | optional        |  `null`         |

Gets or sets the location of the IdP single logout service index. if null, the first SingleLogoutService location with configured protocol binding will be used.

### IsPassive 

| Value type    | Requirement     | Default value   | 
| -----------   | --------------  |-----------      |
| `bool`        | optional        |  `false`        |

Gets or sets a value indicating whether this instance is passive. There are a few common ways to re-authenticate a user with `IsPassive=true`. For example, Integrated Windows Auth (Kerberos) and x509 Cert Based Auth can both be done w/out visibly working with the user's experience. If combined with a `ForceAuthn = true` and `IsPassive = true` in the `AuthnRequest`, it should force the identity provider to re-authenticate the user if both conditions can be met.

### LogoutMethod

| Value type               | Requirement                               | Default value                        | 
| -----------              | --------------                            |-----------                           |
| `Saml2LogoutBehaviour`   | **required** _(already set by default)_   |  `Saml2LogoutBehaviour.RedirectGet`  |

Gets or sets the logout method.

### LogoutRequestSigned

| Value type               | Requirement                               | Default value  | 
| -----------              | --------------                            |-----------     |
| `bool`                   | **required** _(already set by default)_   |  `true`        |

Gets or sets a value indicating whether [logout request signed]. The 'LogoutRequest' message SHOULD be signed or otherwise authenticated and integrity protected by the protocol binding used to deliver the message.

### MaxAge

| Value type               | Requirement                               | Default value  | 
| -----------              | --------------                            |-----------     |
| `TimeSpan`               | optional  _(already set by default)_      |  `null`        |

Gets or sets the `max_age`. If set the `max_age` parameter will be sent with the authentication request. If the identity
provider has not actively authenticated the user within the length of time specified, the user will be prompted to
re-authenticate. By default no max_age is specified.

### Metadata

| Value type               | Requirement                               | Default value  | 
| -----------              | --------------                            |-----------     |
| `Saml2MetadataXml`       | optional                                  |  `null`        |

Generates the metadata file when `CreateMetadatFile=true`.

### MetadataAddress

| Value type          | Requirement               | Default value  | 
| -----------         | --------------            |-----------     |
| `string`            | **required**              |  `null`        |

Gets or sets the identity provider metadata. This can be an address or an xml file location.

### NameIdPolicy

| Value type          | Requirement               | Default value                            | 
| -----------         | --------------            |-----------                               |
| `NameIdPolicy`      | **required**              |  <code>new NameIdPolicy                  |
|                     |                           |    {                                     |
|                     |                           |        Format = NameIDFormats.Persistent,|
|                     |                           |        SpNameQualifier = null,           |
|                     |                           |       AllowCreate = true                 |
|                     |                           |   };                                     |
|                     |                           |  </code>                                 |


Gets or sets the name identifier format. **This is needed to perform logout and SLO**.

### RemoteAuthenticationTimeout	
Gets or sets the time limit for completing the authentication flow (15 minutes by default).

### RemoteSignOutPath

| Value type          | Requirement                             | Default value  | 
| -----------         | --------------                          |-----------     |
| `PathString`        | optional                                |  `null`        |

Gets or sets the remote sign out path. Requests received on this path will cause the handler to invoke SignOut using the SignOutScheme.

### RequestedAuthnContext


| Value type                      | Requirement       | Default value  | 
| -----------                     | --------------    |-----------     |
| `RequestedAuthenticationContext`| optional          |  `null`        |

Gets or sets the requested authn context.

### RequireHttpsMetadata

| Value type          | Requirement             | Default value  | 
| -----------         | --------------          |-----------     |
| `bool`              | optional                |  `false`       |

Gets or sets a value indicating whether the IdP requires Https.

### RequireMessageSigned

| Value type          | Requirement             | Default value  | 
| -----------         | --------------          |-----------     |
| `bool`              | optional                |  `false`       |

Gets or sets a bool value indicating that the Identity Provider must have the message signed. This needs to be set on the IdP side.

### ResponseLogoutBinding

Gets or sets the response logout binding.

### ResponseProtocolBinding

| Value type                        | Requirement             | Default value                                   | 
| -----------                       | --------------          |-----------                                      |
| `Saml2ResponseProtocolBinding`    | optional                |  `Saml2ResponseProtocolBinding.FormPost`        |

Gets or sets the response binding from the identity provider to the service provider. The response protocol binding 
can only be `HTTP-POST` or `HTTP-Artifact`. `HTTP-Redirect` is not allowed per standard as the response will typically 
exceed the URL length permitted by most user agents (browsers).

### Saml2CookieName

| Value type          | Requirement                             | Default value  | 
| -----------         | --------------                          |-----------     |
| `string`            | optional  _(already set by default)_    |  `Saml2`       |

Gets or sets the name of the saml2 cookie.

### SignOutPath

| Value type          | Requirement                              | Default value  | 
| -----------         | --------------                           |-----------     |
| `PathString`        | optional  _(already set by default)_     |  `Saml2`       |


Gets or sets the remote sign out path. This is used by the IdP to send back to after it logs the user out of the IdP session.

### SigningCertificate

| Value type          | Requirement                             | Default value  | 
| -----------         | --------------                          |-----------     |
| `X509Certificate2`  | optional                                |  `null`        |


Gets or sets the signing certificate. If present the outgoing requests will be signed using this certificate. 
The identity provider should be aware of the this public certificate. Both RSA and ECDSA is supported.


### SigningCertificateHashAlgorithmName

| Value type          | Requirement        | Default value                  | 
| -----------         | --------------     |-----------                     |
| `HashAlgorithmName` | optional           |  `HashAlgorithmName.SHA256`    |

Gets or sets the name of the signing certificate hash algorithm.

### SignInScheme	

Gets or sets the authentication scheme corresponding to the middleware responsible of persisting user's identity after a successful authentication. This value typically corresponds to a cookie middleware registered in the Startup class. When omitted, SignInScheme is used as a fallback value.

### SignedOutRedirectUri

Gets or sets the signed out redirect URI. This URI can be out of the application's domain. By default it points to the root.

### SignOutScheme 

The Authentication Scheme to use with SignOut on the SignOutPath. SignInScheme will be used if this value.

### UseTokenLifetime

| Value type          | Requirement        | Default value      | 
| -----------         | --------------     |-----------         |
| `bool`              | optional           |  `true`            |

Gets or sets a value indicating whether to use token lifetime.


### ValidateArtifact

| Value type          | Requirement        | Default value      | 
| -----------         | --------------     |-----------         |
| `bool`              | optional           |  `false`           |

Gets or sets a value indicating whether to validate artifact. This will validate the incoming SAML artifact value if `HTTP-Artifact` 
was set as protocol binding.

### ValidateAudience

| Value type          | Requirement        | Default value      | 
| -----------         | --------------     |-----------         |
| `bool`              | optional           |  `false`           |

Gets or sets a value indicating whether to validate audience.


### ValidAudiences

Gets or sets the valid audiences.If not set the service provider entity Id will be used.

### ValidateIssuer

| Value type          | Requirement        | Default value      | 
| -----------         | --------------     |-----------         |
| `bool`              | optional           |  `false`           |

 Gets or sets a value indicating whether to validate issuer.

### ValidIssuers

Gets or sets the valid issuers. If not set the entityId from the IdP will be used.

### VerifySignatureOnly 

| Value type          | Requirement        | Default value      | 
| -----------         | --------------     |-----------         |
| `bool`              | optional           |  `true`            |

Gets or sets the bool responsible for signature validation true to verify the signature only; 
false to verify both the signature and certificate (it'll do the chain verification to make sure the certificate is valid).


### WantAssertionsSigned

| Value type          | Requirement        | Default value      | 
| -----------         | --------------     |-----------         |
| `bool`              | optional           |  `false`           |

Gets or sets a value indicating whether to require signed assertion.
