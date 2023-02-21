## How partners can set up a single sign-on experience with ArcGIS: part 1

## Part 1: Configure New Member Defaults in ArcGIS Enterprise

1.  Log in to ArcGIS Enterprise portal as an administrator.
2.  Under the Organization tab, go to Settings and New member defaults.
3.  Click the pencil icon under User Type.
4.  Select `Lite` as the default User type and select `Viewer` as the default Role.
5.  Click `Save`. 

## Part 2: Configure Auth0 as a SAML Identity Provider for ArcGIS Enterprise

1.  Log in to your Auth0 account and from your Auth0 dashboard, create a new Single Page Web Application.
2.  Select JavaScript as the technology you are using for your web app.
3.  Under the newly created app Settings, expand Advanced Settings.
4.  Select the Endpoints Settings and copy the `SAML Metadata URL`. 
5.  Log in to ArcGIS Enterprise portal as an administrator.
6.  Under the Organization tab, go to Settings and select Security.
7.  Under Logins select New SAML login. This is where Auth0 will be configured as a SAML Login Identity Provider.
8.  Select `One identity provider` and click `Next`.
9.  Provide a Name for the Organization.
10. Select the option for users to join Automatically. 
11. Under `Metadata source for Enterprise Identity Provider`, select `URL` and paste the SAML Metadata URL copied at Step 4.
12. Click `Save`.

## Part 3: Configure ArcGIS Enterprise as a Service Provider to Auth0

1.  Log in to ArcGIS Enterprise portal as an administrator.
2.  Under the Organization tab, go to Settings and select Security.
3.  Under `Logins`, select `Configure login` on the SAML login.
4.  Select `Download service provider metadata` to extract the ArcGIS Enterprise metadata required to further configure Auth0.
5.  Save and open the downloaded XML file.
6.  From XML file, copy the signin URL located in the `AssertionConsumerService` tag. The URL should have the following pattern:
	https://hostname.fqdn/webadaptor/sharing/rest/oauth2/saml/signin
7.  Return to the Single Page Application Settings in your Auth0 Dashboard.
8.  Select the `Addons` tab.
9.  Enable the `SAML2 WEB APP` addon.  The configuration dialog will open.
10. Select the `Settings` tab, and paste the URL copied in Step 6 into the `Application Callback URL` input box.
11. Scroll to the bottom of the configuration dialog and select `Enable` and `Save`.

## Part 4: Customize the Auth0 SAML Assertion

> **_NOTE:_** In this section of the instructions we will need to customize the SAML Assertion generated by Auth0.  For more information on these steps please review the Auth0 help topic on [Customizing SAML Assertions](https://auth0.com/docs/authenticate/protocols/saml/saml-configuration/customize-saml-assertions)  

1.  Return to the Single Page Application Settings in your Auth0 Dashboard.
2.  Select the `Addons` tab.
3.  Select `SAML2 WEB APP` to open the configuration dialog.
4.  Select the `Settings` tab, and replace the JSON text in the Settings dialog with the following:
    ```
    {
        "mappings": {
            "nickname":    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier",
            "email":       "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
            "name":        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
            "given_name":  "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname",
            "family_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname",
            "upn":         "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn",
            "groups":      "http://schemas.xmlsoap.org/claims/Group"
        }
    }
    ```
5. Save the settings.

## Part 5: Configure Advanced SAML Settings in ArcGIS Enterprise

1.  Log in to ArcGIS Enterprise portal as an administrator.
2.  Under the Organization tab, go to Settings and Security.
3.  Under `Logins`, select `Configure login` on the SAML login.
4.  Select `Show advanced settings`.
5.  Enable the following parameters:
    1) Enable Signed Request
    2) Enable Sign using SHA256
    3) Enable Propogate logout to Identity Provider
6.  Save the configured login properties.
7.  Open the ArcGIS Configuration text file and copy the signout URL located in the `SingleLogoutService` tag. The URL should have the following pattern:
	https://hostname.fqdn/webadaptor/sharing/rest/oauth2/saml/signout
8.  Return to the Single Page Application Settings in your Auth0 Dashboard.
9.  Select the `Addons` tab.
10. Select the `SAML2 WEB APP` addon.  The configuration dialog will open.
11. Select the `Settings` tab, and replace the JSON text in the Settings dialog with the following:
	**_NOTE:_** Be sure to use the URL copied from Step 7 as the callback parameter 
	```
    {
        "logout": {
            "callback": "REPLACE_WITH_SIGNOUT_URL_FROM_ARCGIS_METADATA",
            "slo_enabled": true
        },    
        "mappings": {
            "nickname":    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier",
            "email":       "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
            "name":        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
            "given_name":  "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname",
            "family_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname",
            "upn":         "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn",
            "groups":      "http://schemas.xmlsoap.org/claims/Group"
        }
    }
    ```
12. Save the settings.
13. Select `Settings` in your Auth0 Single Page Application.
14. Add the callout parameter copied from Step 7 under the `Allowed Callback URLs`.
15. Select `Save Changes`.

## Optional Step: Create Auth Pipeline Rule to Encrypt the AuthO Assertion (Recommended for Production)

> **_NOTE:_** While encrypting the SAML Assertion is not neccesary to complete this walkthrough, it is a recommended configuration for securing production environments.

> **_NOTE:_** Auth0 supports encrypting the SAML Assertion by using an Auth Pipeline Rule.  A Rule is a custom snippet of JavaScript code that allows an administrater to add custom logic to the authentication pipeline.  The rule created in this session will programatically provide the Public Key and Certificate to use when encrypting the SAML Assertion returned to ArcGIS Enterprise.  For more information regarding the steps outlined in the following section please see the **Send encrypted SAML authentication assertions** in the [Sign and Encrypt SAML Requests](https://auth0.com/docs/authenticate/protocols/saml/saml-sso-integrations/sign-and-encrypt-saml-requests) Auth0 help topic.

> **_NOTE:_** You must manually configure Auth0 to encrypt the SAML Assertion using a certificate generated by ArcGIS Enterprise.  This Certificate, and its Public Key, must be converted to a single line string so that it can be added to the JavaScript Block of the Auth0 Rule. 

> **_NOTE:_** This step requires OpenSSL to convert the contents of the ArcGIS Enterprise DER (Distinguished Encoding Rules) certificate for signing SAML assertions to a PEM (Primary Enhanced Mail) certifcate.  OpenSSL is readily available on Linux and MacOS.  If using Windows, you can use the Linux Subsystem for Windows 10 and install an Ubuntu Container, Cygwin, a docker container running Ubuntu, or install one of the existing versions for Windows.  The steps listed in these instructions were generated on a machine running MacOS.

> **_NOTE:_** This section uses the [Portal Administrator Directory](https://enterprise.arcgis.com/en/portal/latest/administer/windows/about-the-arcgis-portal-directory.htm) to export the ArcGIS Enterprise Certificate for signing SAML Assertions.  

1.  Open a browser and navigate to your Portal Administrator Directory:
    ```
    https://hostname.fqdn/webadaptor/portaladmin/
    ```
2.  Login as an administrative user
3.  Navigate to Security -> SSLCertificates
4.  Select the certificate named `samlcert`.
5.  Select `Export` to download the file.
6.  Open a terminal or command prompt and browse to the directory containing the DER encoded certificate.
7.  Convert the certifcate to a PEM certificate file using the following command:
    ```
    openssl x509 -inform der -in samlcert.cer -out samlcert.pem
    ```
8.  Extract the Public Key from generated PEM certificate file using the following command:
    ```
    openssl x509 -in samlcert.pem -pubkey -noout > samlcert_pubkey.pem
    ```
9.  Open the PEM files generated in steps 7 and 8.  To convert the multi-line text to a single line, replace all line endings (newlines) with `\n`.

    Here is a sample file before being editted:
    ```
    -----BEGIN PUBLIC KEY-----
    MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAlzzj6QkuEqnThoi5qTvr
    7baPvvPRQO08FVolVZ1XtPDwZUx6r4rOWaOuEdi8IKlkLyh5HjSHbYVE8PqJZ+2x
    Voq11rW4zQOf8zf6wM3nkn3AjrIgg1cBt51r5B9vnBsjjI7a2lWFS9ITY9scEtyu
    NT8Pxu20FuNAalb4vh5drRCZpufrwHtyFaanDLmcCGaPnnGuM5AU4ZynDqrS46Dp
    LFHWp1wb+/WU/Ix1LtC8UeO0gXyObMFARnA5L+XPYqESzjDxZ1mQrTMuXoVTo7Nm
    tclbeEWgMBytllsWnYg1UuWfvAfwt38YstlSy62VFo6002UUygXD/DLvxwJhN/3T
    AwIDAQAB
    -----END PUBLIC KEY-----
    ```    
    and the same file after being editted.  Please note, the text below is a single line regardless of how appears in the Markdown editor.

    ```
    -----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAlzzj6QkuEqnThoi5qTvr\n7baPvvPRQO08FVolVZ1XtPDwZUx6r4rOWaOuEdi8IKlkLyh5HjSHbYVE8PqJZ+2x\nVoq11rW4zQOf8zf6wM3nkn3AjrIgg1cBt51r5B9vnBsjjI7a2lWFS9ITY9scEtyu\nNT8Pxu20FuNAalb4vh5drRCZpufrwHtyFaanDLmcCGaPnnGuM5AU4ZynDqrS46Dp\nLFHWp1wb+/WU/Ix1LtC8UeO0gXyObMFARnA5L+XPYqESzjDxZ1mQrTMuXoVTo7Nm\ntclbeEWgMBytllsWnYg1UuWfvAfwt38YstlSy62VFo6002UUygXD/DLvxwJhN/3T\nAwIDAQAB\n-----END PUBLIC KEY-----\n
    ```
10. Connect to your Auth0 Dashboard.
> **_NOTE_** To complete the next set of steps you will need to know your Auth0 Single Page Application's Client ID.
11. Copy the `Client ID` from the `Basic Information` section of your Auth0 Single Page application's settings tab.
12. Select Auth Pipeline -> Rules.
13. Select `Create` to create a new rule.
14. Select the `Empty rule` template.
15. Provide a descriptive name for the rule
16. Copy the following JavaScript snippet into the Script editor.
    
    ```
    function (user, context, callback) {
        if (context.clientID === 'The Client ID copied in step 11') {
            context.samlConfiguration = (context.samlConfiguration || {});
            context.samlConfiguration.encryptionPublicKey = "-----BEGIN PUBLIC KEY-----[..entire string created in step 9]-----END PUBLIC KEY-----\n";
            context.samlConfiguration.encryptionCert = "-----BEGIN CERTIFICATE-----[..entire string created in step 9]-----END CERTIFICATE-----\n";
        }  
        return callback(null, user, context);
    }
    ```

17. Select `Save Changes`
18. Log onto Portal for ArcGIS Enterprise as an Administrator
19. Select Organization -> Settings -> Security.
20. In the `Logins` section, select `Configure Login`.
21. Select `Show advanced settings`.
22. Enable `Encrypt Assertion`.
23. Save the configuration changes.

## How partners can set up a single sign-on experience with ArcGIS: part 2

## Part 1: Download the Auth0 Single Page Application JavaScript Sample App

> **_NOTE:_** The Auth0 Sample App requires that either [NodeJS LTS](https://nodejs.org/en/download/) or [Docker](https://www.docker.com/) is installed and uses the [Auth0 JavaScript Login Tutorial](https://auth0.com/docs/quickstart/spa/vanillajs/01-login).

1.  Log into your Auth0 account. From your Auth0 dashboard, click on `Applications`, then select your ArcGIS app.
2.  Navigate to the `Quick Start` tab.
3.  Select `JavaScript` to open the JavaScript tutorial.
4.  In the JavaScript: Login tutorial, click `DOWNLOAD SAMPLE`. Follow the Auth0 Tutorial.

## Optional Step: Test the Auth0 JavaScript Sample App

1.  Open a Terminal or command prompt and browse to the extracted ***01-login*** folder.
2.  If you are running the application using NodeJS, you will use `npm install && npm start` to launch the application.
3.  If you are running the application using Docker, you will either use  `exec.sh` (Linux / MacOS) or `exec.ps1` (Powershell in Windows) to launch the application.
4.  Once the application is running, open a browser and navigate to http://localhost:3000
5.  Select the Login button and provide credentials for an Auth0 user to test that the application works.

## Part 2: Register an Application with ArcGIS Enterprise

1.  Log into your ArcGIS Enterprise organization as the user that will own the application.
2.  Under the `Content` tab, click on `New item`.
3.  In the New item dialogue box, select `Application`.
4.  For Application type, select `Web mapping`, then set the URL to http://localhost:3000 and click `Next`.
5.  Provide a title, tags and summary for the App Registration, then click `Save`.
6.  Once the item is created, click the `Settings` tab.
7.  Under Web Mapping Application, click `Register`.
8.  Set the Redirect URL to http://localhost:3000/map.html then click `Add`.
9.  Select `Register`.

## Part 3: Add the map.html page to the Auth0 JavaScript Sample App

> **_NOTE:_** The map.html file referred to in this section was created using two ArcGIS Maps SDK for JavaScript code samples available from https://developers.arcgis.com:
>  - [Intro to FeatureLayer](https://developers.arcgis.com/javascript/latest/sample-code/layers-featurelayer/)
>  - [Access ArcGIS Online items using OAuth 2.0](https://developers.arcgis.com/javascript/latest/sample-code/identity-oauth-basic/) 

1.  Create a new file called `map.html` in the `/01-login/public` folder of the Auth0 Sample App.
2.  Navigate to the map.html file on [GitHub](https://github.com/EsriCanada/How-partners-can-set-up-a-single-sign-on-experience-with-ArcGIS-blog-post-series/blob/main/map.html).
3.  Copy raw contents and paste to your map.html file in /01-login/public.
4.  In ArcGIS Enterprise, under the `Content` tab, click on your web app then click the `Settings` tab.
5.  Under Web Mapping Application, click `Registered Info`.
6.  Copy the `App ID`. You will use it in an upcoming step.
7.  In the `map.html` file in the /01-login/public folder, replace the following URLS:
	 1) `your_portal_url` – ArcGIS Enterprise portal URL
	 2) `your_app_id` – The App ID from your registered app in ArcGIS Enterprise
	 3) `your_fs_service_URL` – The secure layer you wish to use in the map. This example uses the SampleWorldCities service that is created by default when ArcGIS Enterprise is installed.
8.  `Save`.
9.  Open `/01-login.index.html`.
10. Find the `Home` navigation item. The code looks like this:
	```
    <li class="nav-item">
        <a href="/" class="nav-link route-link">Home</a>
    </li>
    ```
11. Add the following code under the `Home` item. This will cause users to navigate to the map.html page.
	```
	<li class="nav-item">
            <a href="/map.html" class="nav-link route-link">Map</a>
        </li>
	```
12. The code should now look like this:
    ```
    <ul class="navbar-nav mr-auto">
        <li class="nav-item">
            <a href="/" class="nav-link route-link">Home</a>
        </li>
        <li class="nav-item">
            <a href="/map.html" class="nav-link route-link">Map</a>
        </li>
    </ul>
    ```
13. Run the application.
14. Navigate to http://localhost:3000, log into the web application and then click on the `Map` tab to load the map.	

## Part 4: Disable the default ArcGIS Enterprise login screen

> **_NOTE:_** Before completing these steps, ensure that at least one Auth0 user is assigned the Administrator role in ArcGIS Enterprise. If an Auth0 user is not assigned as an administrator, portal administration will need to be performed programmatically. For more information on how to create an Auth0 user, review the Auth0 help topic [Create Users](https://auth0.com/docs/manage-users/user-accounts/create-users).

1.  Log into your ArcGIS Enterprise as an administrator.
2.  Under the `Organization` tab, go to `Members`. Ensure that there is at least one member in the ArcGIS Enterprise organization with an Administrator role and that the user also has an existing account in Auth0. 
3.  Under the `Organization` tab, go to `Settings` and select `Security`.
4.  Under `Logins`, disable the ArcGIS login.
5.  Navigate to http://localhost:3000, log into the web application and then click on the `Map` tab to load the map.
