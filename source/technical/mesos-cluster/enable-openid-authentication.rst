Enabling open-id connect authentication
---------------------------------------

Mesos/Marathon/Chronos do not support open-id connect authentication natively. 

A very simple solution is to front the mesos cluster with an Apache server that itself is capable of negotiating authentication for users.

The following configuration can be used to setup a reverse proxy that uses the module <span>***mod\_auth\_openidc***:</span>

``` syntaxhighlighter-pre
ServerName mesos.example.com

<VirtualHost *:443>
  ServerName mesos.example.com

  LoadModule auth_openidc_module /usr/lib/apache2/modules/mod_auth_openidc.so

  OIDCClaimPrefix                 "OIDC-"
  OIDCResponseType                "code"
  OIDCScope                       "openid email profile"
  OIDCProviderMetadataURL         https://iam.deep-hybrid-datacloud.eu/.well-known/openid-configuration
  OIDCClientID                    332e618b-d3bf-440d-aea1-6da2823aaece # replace with your client ID
  OIDCClientSecret                ****                                 # replace with your client secret
  OIDCProviderTokenEndpointAuth   client_secret_basic
  OIDCCryptoPassphrase            ****                                 # replace with your passphrase
  OIDCRedirectURI                 https://mesos.example.com/mesos/redirect_uri

  OIDCOAuthVerifyJwksUri "https://iam.deep-hybrid-datacloud.eu/jwk"

  <Location /mesos>
    AuthType openid-connect
    Require valid-user
    LogLevel debug
  </Location>

 <Location /marathon>
    AuthType oauth20
    Require valid-user
    LogLevel debug
    RequestHeader set Authorization "Basic YWRtaC46bTNzb3NNLjIwMTY="
 </Location>

 <Location /chronos>
    AuthType oauth20
    Require valid-user
    LogLevel debug
    RequestHeader set Authorization "Basic YWRtaZ46bTNzb3NDLjIwMTY="
 </Location>


  ProxyTimeout 1200
  ProxyRequests Off
  ProxyPreserveHost Off


  ProxyPass /mesos/ http://172.20.30.40:5050/
  ProxyPassReverse /mesos/ http://172.20.30.40:5050/

  ProxyPass /marathon/ http://172.20.30.40:8080/
  ProxyPassReverse /marathon/ http://172.20.30.40:8080/

  ProxyPass /chronos/ http://172.20.30.40:4400/
  ProxyPassReverse /chronos/ http://172.20.30.40:4400/


  RemoteIPHeader X-Forwarded-For



  ## Logging
  ErrorLog "/var/log/apache2/proxy_mesos_error_ssl.log"
  ServerSignature Off
  CustomLog "/var/log/apache2/proxy_mesos_access_ssl.log" combined

  ## SSL directives

  SSLProxyEngine on
  SSLEngine on
  SSLCertificateFile      "/etc/letsencrypt/live/mesos.example.com/fullchain.pem"
  SSLCertificateKeyFile   "/etc/letsencrypt/live/mesos.example.com/privkey.pem"
</VirtualHost>
```

<span>
</span>

<span>Note that Line 30 is needed if you have enabled basic HTTP authentication to protect your endpoints (in the example above, username/password authentication has been enable for Marathon).</span>

<span>In this case you need to add the Authorization header in the request to the backend. The hash can be computed with the following python script:</span>

``` syntaxhighlighter-pre
import base64
hash = base64.b64encode(b'user:password')
```

Once the proxy is up and running you can contact the cluster API endpoints using the IAM (open-id connect) token:

Marathon API endpoint: <a href="https://mesos.example.com/marathon" class="uri" class="external-link">https://mesos.example.com/marathon</a>

Chronos API endpoint: <a href="https://mesos.example.com/chronos" class="uri" class="external-link">https://mesos.example.com/chronos</a>

For example:

``` syntaxhighlighter-pre
curl -H "Authorization: bearer $IAM_ACCESS_TOKEN" -X GET https://mesos.example.com/marathon/v2/apps
```

If you want to allow users to access also the Web interfaces of Marathon and Chronos, then add the following configuration:

``` syntaxhighlighter-pre
  <Location /marathon-web>
    AuthType openid-connect
    Require valid-user
    LogLevel debug
    RequestHeader set Authorization "Basic YWRtaC46bTNzb3NNLjIwMTY="
  </Location>

  <Location /chronos-web>
    AuthType openid-connect
    Require valid-user
    LogLevel debug
    RequestHeader set Authorization "Basic YWRtaZ46bTNzb3NDLjIwMTY="
  </Location>

  ProxyPass /marathon-web/ http://172.20.30.40:8080/
  ProxyPassReverse /marathon-web/ http://172.20.30.40:8080/

  ProxyPass /chronos-web/ http://172.20.30.40:4400/
  ProxyPassReverse /chronos-web/ http://172.20.30.40:4400/
```

The Web UIs will be accessible at the following urls:

Marathon Web UI: <a href="https://mesos.example.com/marathon-web/" class="uri" class="external-link">https://mesos.example.com/marathon-web/</a>

Chronos Web UI: <a href="https://mesos.example.com/chronos-web/" class="uri" class="external-link">https://mesos.example.com/chronos-web/</a>
