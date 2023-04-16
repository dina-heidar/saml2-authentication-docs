# Saml2 Authentication

This tool that can be used with any .Net Core applications to authenticate using SAML2. This tool is not dependent on any .NET Framework libraries. It has been tested with ADFS and IdentityServer4 as well.

## Profiles and Bindings

This tool supports the following SAML profiles, message flows and bindings:

<table width="100%" border="1" center>
    <thead>
        <tr>
            <th>Profile</th>
            <th>Message Flows</th>
            <th>Binding</th>
        </tr>
    </thead>
    <tbody> 
        <tr>
            <td rowspan="4">Web SSO</td>
            <td rowspan="2"><code>&lt;AuthnRequest&gt;</code> from SP to IdP</td>
            <td>HTTP Redirect</td>            
        </tr>
        <tr>
            <td>HTTP POST</td>
        </tr>        
        <tr>            
            <td rowspan="2">IdP <code>&lt;Response&gt;</code> to SP</td>
            <td>HTTP POST</td>            
        </tr>
        <tr>
            <td>HTTP Artifact</td>
        </tr>
        <tr>
            <td rowspan="4">Single Logout</td>
            <td rowspan="2"><code>&lt;LogoutRequest&gt;</code></td>
            <td>HTTP Redirect</td>            
        </tr>
        <tr>
            <td>HTTP POST</td>
        </tr>       
        <tr>
            <td rowspan="2"><code>&lt;LogoutResponse&gt;</code></td>
            <td>HTTP Redirect</td>       
        </tr>
        <tr>
            <td>HTTP POST</td>
        </tr>
        <tr>
            <td rowspan="2">Metadata</td>
            <td >Consumption</td>
            <td></td>                 
        </tr>
        <tr>
            <td>Exchange</td>  
             <td></td>           
        </tr>        
  </tbody>
</table>
