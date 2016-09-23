# google-calendar-salesforce
Syncing google calendar with Salesforce using APEX

This is an attempt to connect Google Calendar with Salesforce. I have pieced together information from a variety of different sources.

Below are first step instructions from https://www.sundoginteractive.com/blog/connecting-to-google-api-using-oauth2-part-2-logging-into-google Big Shoutout to Terry, wouldn't have been able to figure this out without him.

Connecting to Google API using OAuth2 - Part 2 - Logging into Google
Now that the registered app is setup from our first blog post we can how login to Google to retrieve an authorization code.

When doing web service calls from Apex in Salesforce we must tell Salesforce which URLs we are trying to hit so Salesforce can trust those calls. In order to do this we must do the following:

1)	Log into Salesforce
2)	Go to Setup
3)	Under Administer, Security Controls, click on ‘Remote Site Settings’
4)	Setup a new ‘Remote Site’ for each of these URLs
a.	https://www.googleapis.com
b.	https://accounts.google.com
c.	https://spreadsheets.google.com
d.	https://www.google.com

If you do not do this you will get a message about the web service call not being allowed for that remote site.

To authenticate to the Google API the first thing that needs to be done is to send the user to Google in order to login so we can get an authorization code. As can be seen by the code in DoGoogleConnectCall:

The user is sent to Google because a PageReference points to accounts.google.com/o/oauth2/auth. The user is expected to enter his/her credentials to access the Google account and approve the permissions that are needed for the application to function.

In the code for redirecting to account.google.com the client id and redirect uri come from the details of the registered app that was created earlier. The Google documentation for the APIs does a thorough job of explaining the various query string parameters needed on the URL. The result of a successful login to Google is that the user is returned back to the URL defined in the redirect uri field. This will cause the constructor of the Apex controller ‘GoogleConnect()’ tied to the Visualforce page to run again, which will use logic to automatically start the requested action.

But how will the code know what program made this request in the first place? The code will know which action is requested because of the ‘state’ parameter in the request. Whatever is in this parameter will be passed back on the redirect uri so that it can be used to determine who requested the call. The code related to the ‘state’ parameter is first used in DoGoogleConnectCall(). It is next referenced in the constructor GoogleConnect() so the code will know which scenario to complete.

It is this access token that can be kept and used to make the desired API calls until it expires. Refresh tokens can also be retrieved so that the initial authentication process does not need to fully restart. The code for getting this access token can be found in the retrieveGoogleAccessToken() method.

Notice how we must specify the client secret from the registered app along with the client id. The authorizationCode is the value that was retrieved when we logged in. The googleClientID and googleSecretCode are values that come from the properties of the Google Registered App. The redirectURI is the URL of this same Visualforce page.

One of the hardest parts of any integration is getting all of the integration calls formatted just right. For each part of the request body and headers be careful to name them correctly. This call to get the access token returns a JSON string that is then parsed in the parseJSONToMap() method. Web service calls usually return XML or JSON. In this case we get JSON back, but in our other calls we will get XML back. Luckily Apex in Salesforce has some built-in classes that can help in parsing the JSON and XML correctly and efficiently.
