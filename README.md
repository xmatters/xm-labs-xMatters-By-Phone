# Initiate xMatters Event by Phone (AKA, Phone Initiation)

This integration will help you to initiate xMatters notifications by calling a Phone number. It allows you to select the type of notification, the group you want to target and record a message over the phone that will be transcribed, URL shortened and sent as an xMatters notification.

<kbd>
  <img src="https://github.com/xmatters/xMatters-Labs/raw/master/media/disclaimer.png">
</kbd>

# Pre-Requisites

- Twilio account (https://www.twilio.com)
- Twilio Phone number with Calling capabilities.
- Bitly Account for shortening recording URLS. [get one](https://www.bitly.com).
- xMatters account - If you don't have one, [get one](https://www.xmatters.com)!

# Files

- Twilio function files

  - [xm_settings](TwilioFunctions/xm_settings.txt)
  - [xm_action](TwilioFunctions/xm_action.txt)
  - [xm_group](TwilioFunctions/xm_group.txt)
  - [xm_record](TwilioFunctions/xm_record.txt)
  - [xm_confirmRec](TwilioFunctions/xm_confirmRec.txt)
  - [xm_shorten](TwilioFunctions/xm_shorten.txt)
  - [xm_message](TwilioFunctions/xm_message.txt)

- [xMatters Initiate Event via Phone Call Communication Plan](xMattersPlan/InitiateEventviaPhoneCall.zip)

# How it works

- Calling a Twilio Voice enabled phone number will allow you to create a new xMatters event targeting a group of your choice.
- This integration initiate a Twilio function that will play a series of prompts to the caller.
- The caller can press digits on their phone to control the xMatters event that is created.
- The user can record a message over the phone. The audio recording is transcribed and an mp3 audio file is stored on Twilio.
- The URL to the audio recording is shortened using Bitly and both the transcription text and a shortened link to the recording is sent as part of the xMatters Notification.
- The caller ID of the calling phone must belong to a user inside of xMatters and must be configured in the Twilio function. Multiple users can initiate an event but they must be valid xMatters users.
- The Twilio functions will provide telephone prompts to guide you through initiate an xMatters notification.
- You can decide to send a regular xMatters Alert/Notification or an xMatters Conference Bridge.
- You can predefine up to 9 xMatters groups that can be target with your notification. You can select the group you would like to target using xMatters from phone prompts.
- The content of the message must be configured in the xMatters communication plan.

## Integration Function Workflow

1. User Calls Twilio Voice enabled phone number.

2) Twilio Function **xm_settings** is initiated.
   - This file contains all the settings for the integration
   - Performs authentication based on the calling phone Caller ID.
   - Phones that block / hide the Caller ID will not be able to use this integrations.

3. Twilio Function **xm_action** is initiated.
   - Prompts the user to create a regular xMatters Alert or an Alert with a Conference Bridge.
   - This sets a variable so _xm_message_ will send the event to the proper xMatters Inbound Integration URL.
   - 1. **Alert** -> **On-Call Alert**
   - 2. **Conference Bridge** -> **On-Call Conference**

```
  Hey there, What's the problem? Press 1 to alert an xMatters group. Press 2 to start an xMatters conference bridge.
```

4. Twilio Function **xm_group** is initiated.
   - Prompts the user to select a target group for the notification.
   - Configure up to 9 groups as options in the prompts.
   - This step can be disabled and you can set default recipients in xMatters instead.
   - This step is disabled by setting zero groups in xm_settings.

```
What group would you like to alert. Press 1 for xyz. Press 2 for jkl. Press 3 for mno.
What group would you like to invite to the conference. Press 1 for xyz. Press 2 for jkl. Press 3 for mno.
```

5. Twilio Function **xm_record** is initiated.
   - Prompts the user to record a message over the phone.

```
Record your message after the beep. Press 1 if you are happy with your recording, any other key to restart.
```

6. Twilio Function **xm_confirm** is initiated.
   - Handles the case where user wants to re-record the message.
   - Moves the script to the next function if the user is happy with the recording.

```
Record your message after the beep. Press 1 if you are happy with your recording, any other key to restart.
```

7. Twilio Function **xm_shorten** is initiated.
   - Sends the recording URL to Bitly where the long form URL is shortened.
   - If there is an error shortening the URL we will continue on using the long version of the URL.

```
Long Recording URL:
https://api.twilio.com/2010-04-01/Accounts/AC3e82fb9a36dc3babf7617/Recordings/RE9f98203f4201a34920fd1748c8

Converted to:
http:/bit.ly/2sjdsis2
```

8. Twilio Function **xm_message** is initiated.
   - Plays some fun messages while we are waiting for the recording transcription to complete.
   - This function continuously loops checking the transcription status each time.
   - The time this step takes to complete will vary depending on the length of the recorded message.
   - When the transcription status changes to "completed" an API call to xMatters is made to initiate the new event.
   - The user is notified if the event was successfully created in xMatters or if there was a problem injecting the event.
   - Recordings / Transcriptions have a length limitation from 2 seconds up to 2 minutes.

**Wait Messages**

```
- Patients can be difficult but the rewards are great, your transcription is not quite finished yet.
- Maybe it's time for a joke... Just kidding we will be done very soon.
- You have been very patient I must say.
- I will be quiet now but im still working on it and will let you know as soon as its done.
```

**Event Sent to xMatters**

```
- Sit back and relax, xMatters is taking care of business. Goodbye.
- Oops, something went wrong. The event has not been sent. You will need to send this event directly from xMatters.
```

9. xMatters Inbound Integration Script.

   - This script gets the xMatters recipients out of the payload.
   - This script adds trascription text to xMatters properties "Description" and "Short Description".

# Installation

## Create an xMatters Integration User

This integration requires a user who can authenticate REST web service calls when injecting events.

This user needs to be able to work with events, but does not need to update administrative settings. While you can use the default Company Supervisor role to authenticate REST web service calls, the best method is to create a user specifically for this integration with the "REST Web Service User" role that includes the permissions and capabilities.

**Note**: If you are installing this integration into an xMatters trial instance, you don't need to create a new user. Instead, locate the "Integration User" sample user that was automatically configured with the REST Web Service User role when your instance was created and assign them a new password. You can then skip ahead to the next section.

**Note 2**: This user won't be able to "see" other users with the Company Supervisor role and so won't be able to authenticate

**To create an integration user:**

1. Log in to the target xMatters system.
2. On the **Users** tab, click the **Add New User** icon.
3. Enter the appropriate information for your new user.
   Example User Name **Twilio_API_User**
4. Assign the user the **REST Web Service User** role.
5. Click **Save**.

Make a note of the user name and password that you set; you will need them when configuring other parts of this integration.

The integration user name and password will be needed when configuring the Twilio xm_settings Function:
**xm_user** and **xm_pass**

<br><br>

## Import and Configure the xMatters Communication Plan

The next step is to import the communication plan.

To import the communication plan:

1. In the target xMatters system, on the **Developer** tab, click **Import Plan**.
2. Click **Browse**, and then locate the downloaded communication plan: [xMatters Initiate Event via Phone Call Communication Plan](xMattersPlan/InitiateEventviaPhoneCall.zip)
3. Click **Import Plan**.
4. Once the communication plan has been imported, click **Plan Disabled** to enable the plan.
5. In the **Edit** drop-down list, select **Access Permissions**.
6. Add the **Twilio_API_User** or Make the Plan **Accessible by All**.
7. **Save Changes**.
8. In the **Edit** drop-down list, select **Forms**.

9. For the **On-Call Resource Alert** form, in the **Not Deployed** drop-down list, click **Enable for Web Service**.
10. After you Enable for Web Service, the drop-down list label will change to **Web Service**.
11. In the **Web Service** drop-down list, click **Sender Permissions**.
12. Add the **Twilio_API_User** you created above, and then click **Save Changes**.

Repeat steps 6 to 9 for **On-Call Resource Conference** form.

**Special Note:** The **Twilio_API_User** will always initiate the xMatters event for this integration. A separate setting inside the Twilio **xm_settings** function will control whether the calling phone/person is allowed to initiate an xMatters event using this integration. Assuming the person is calling from a phone with a caller ID matching a user in xMatters and the xm_settings configuration, an event will be created by the **Twilio_API_User**.

<br><br>

## Get the xMatters Inbound Integration Endpoints

1. In the **Edit** drop down list on the _xMatters Initiate Event via Phone Call Communication Plan_ Click **Integration Builder**.
2. On the Integration Builder tab, expand the list of inbound integrations.
3. Click on either **On-Call Alert** or **On-Call Conference**.
4. Ensure **URL Authentication** is set.
5. In the **How to trigger the integration** section change the Authenticating User to **Twilio_API_User**.

You must supervise this user and it must have a Web Service User Role. If you cannot select the **Twilio_API_User** if it because you do not supervise that user or they do not have the REST Web Service User role.

6. Copy the URL listed under the **Trigger** section.

You will need this URL for both **On-Call Alert** and **On-Call Conference**.

<br><br>

## Create a Bitly Account

1. Go to [Bitly.com](https://www.bitly.com) and create a new account.

2. Hamburger Menu -> Your profile -> Generic Access Token

3. Enter your Bitly password.

4. Click Generate Token.

5. Copy Access token.

6. Save the token as we will use it in our Twilio xm_settings function later.

<br><br>

## Create Twilio Account

1. Create a Free Twilio account [here](https://www.twilio.com/try-twilio)

## Purchase Twilio Voice Number.

1. Search "Buy a Number" and click "Buy a Number":

   <kbd>
     <img src="/media/Buy-Number.png" width="250px">
   </kbd>

2. Perform a Twilio phone number search.


    <kbd>
      <img src="/media/Phone-Search.png" width="550px">
    </kbd>

3. Make sure the number you select has Voice Capabilities and Buy Number.

   <kbd>
     <img src="/media/Buy-Phone.png" width="550px">
   </kbd>

## Create Twilio Functions

1. Click the **Hamburger** icon.

   <kbd>
     <img src="/media/Burger.png">
   </kbd>

2) Scroll down to the bottom of the menu and go to **Runtime**.

   <kbd>
     <img src="/media/Runtime.png">
   </kbd>

3) Click on **Functions -> Manage**.

   <kbd>
     <img src="/media/Functions-Manage.png">
   </kbd>

4) Click **Red Plus** button.

   <kbd>
     <img src="/media/Add-Function.png">
   </kbd>

5) Select **Blank**

   <kbd>
     <img src="/media/Blank.png">
   </kbd>

6) Click **Create**

   <kbd>
     <img src="/media/Create.png">
   </kbd>

7) Give the Function a **Name**. _See below for proper Names_.

8) Set the Function **Path**. _See below for proper Paths_.

9) Clear Twilio Code and copy the code from this repository to Twilio. We will configure these in the next section.

10) **Save**.

11) Repeat steps 4 to 9 for each function listed below..

Appropriate Names and Paths can be found below.

- Function Name does not need to match what is shown below. You can give any name.
- Function Path Must match what is shown below.

   <kbd>
      <img src="/media/Functions-top.png" width="550px">
    </kbd>

   <kbd>
      <img src="/media/Functions-bottom.png" width="550px">
    </kbd>

Copy the Code below to Twilio Functions

- [xm_settings](TwilioFunctions/xm_settings.txt)
- [xm_action](TwilioFunctions/xm_action.txt)
- [xm_group](TwilioFunctions/xm_group.txt)
- [xm_record](TwilioFunctions/xm_record.txt)
- [xm_confirmRec](TwilioFunctions/xm_confirmRec.txt)
- [xm_shorten](TwilioFunctions/xm_shorten.txt)
- [xm_message](TwilioFunctions/xm_message.txt)

11. Create a new Twilio API key by clicking on **API Keys**

<kbd>
  <img src="/media/API-Key.png" width="200px">
</kbd>

12. Click the + to add a new Key and give it an appropriate name, then hit Create Key.

<kbd>
  <img src="/media/API-Key-Value.png" width="500px">
</kbd>

13. Copy the SID and Secret to a safe place, **this will be the only time the secret is shown**.

<br><br>

## Configure Twilio "xm_settings" Functions

1. Go back to Functions -> Manage Screen.

   <kbd> 
     <img src="/media/Functions-Manage.png">
   </kbd>

2. Click on the function with path **xm_settings**. _May have the name: 01-xMatters-Notify-Settings_

3) Update the Settings Function Script between Lines 11 to 72. Instructions about each setting is provided below:

<br><br>

### Twilio Function Paths

```js
//Twilio Function Paths
setting.twilio_path = 'https://twilio-function-url.134.twil.io/';
```

| **Twilio Function Paths** |                                                |
| ------------------------- | ---------------------------------------------- |
| setting.twilio_path       | The base path to all of your twilio functions. |

<kbd>
  <img src="/media/Function-Path.png">
</kbd>

<br><br>

### xMatters Base URL and Webservice User

```js
// xMatters Base URL
// Do not include trailing slash
xmatters = 'https://company.xmatters.com';

// xMatters Webservice User name and password
xm_user = 'REST';
xm_pass = 'rest password';
```

| **xMatters Base URL and Webservice User** |                                                                                  |
| ----------------------------------------- | -------------------------------------------------------------------------------- |
| xmatters                                  | The base URL of your xMatters environment. Example: https://company.xmatters.com |
| xm_user                                   | The username of the xMatters Twilio_API_User                                     |
| xm_pass                                   | The password of the xMatters Twilio_API_User                                     |

### Populate Twilio SID and Secret

These came from the Twilio API Key above

```js
// Twilio SID and Secret API key for authenticating
// into the Twilio API for transcription
settings.twilio_sid = 'LONG-SID-VALUE-HERE';
settings.twilio_sec = 'LONG-SECRET-HERE';
```

| **Twilio API Credentials** |                                        |
| -------------------------- | -------------------------------------- |
| settings.twilio_sid        | The SID value of the Twilio API key    |
| settings.twilio_sec        | The secret value of the Twilio API key |

<br><br>

### Inbound Integration URLS

```js
// This is the Inbound Integration url for "On-Call Alert"
setting.url_alert =
  'https://company.xmatters.com/api/integration/1/functions/29467509-76-b0af-c70585021d8e/triggers?apiKey=2675d2d5-16f3-4c7e-4e9549451662';

// This is the Inbound Integration url for "On-Call Conference Bridge"
setting.url_conf =
  'https://company.xmatters.com/api/integration/1/functions/a77562ae-05f-b870-bda997117b5c/triggers?apiKey=f58a9959-7233--95ff-eec3c7443e2f';
```

| **Integration URLS** | [Get the xMatters Inbound Integration Endpoints](#get-the-xmatters-inbound-integration-endpoints) |
| -------------------- | ------------------------------------------------------------------------------------------------- |
| setting.url_alert    | The inbound integration endpoint ULR for the On-Call Alert Communication plan.                    |
| setting.url_conf     | The inbound integration endpoint ULR for the On-Call Conference Communication plan.               |

<br><br>

### xMatters Authenticated Users

```js
// List of xMatters users that are allowed to initiate this integration
// The caller ID initiating this event must be a voice device listed on an xMatters user profile
Authorized_Users = ['userid1', 'userid2', 'userid3'];
```

- This is a comma separated list of xMatters userID's. Each UserID must be in quotations and separated by a comma.
- The integration will check if a user listed here has a Voice device in xMatters matching the caller ID of the person calling your Twilio number to initiate an xMatters event.
- If the caller ID that is calling your Twilio number matches a voice device of a user in xMatters listed in the Authorized_Users array, the function will proceed.
- If the caller ID that is calling your Twilio number does not match a voice device of a user in xMatters listed in the Authorized_Users array, the function will play a not authorized message and terminate the call.

<br><br>

### Group Configuration

```js
// Set the number of groups you would like to be able to target from this integration.
// The number set below must match the number of groups you have in the xMatters_Groups and Speak_Groups arrays below.
// If you set this value to 0 you must add groups / users as default recipients in the xMatters form layout.
setting.NumberofGroups = 3;

// Set the xMatters Group Names.
// Group names must match the name of your xMatters groups exactly.
// You can add additional Groups by adding parameters to the array below.
// This array must have the same number of elements as the Speak_Group array below.
setting.xMatters_Groups = ['CAB Approval', 'Cloud DevOps', 'Database Infrastructure'];

// Set the Twilio Group Name.
// You can add additional punctuation, spaces, dashes, and upper case letters to group names to help the text to speech engine pronounce it better.
// This array must have the same number of elements as the xMatters_Groups array above.
setting.Speak_Groups = ['C.A.B. Approval', 'Cloud DevOps.', 'Database Infrastructure'];
```

| **Integration URLS**    |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| setting.NumberofGroups  | The number of Groups listed in **setting.xMatters_Groups**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| setting.xMatters_Groups | Listing of Group Names, exactly how they are configured in xMatters. You will be able to send the xMatters notifications to any of the groups listed here. You can have up to 9 groups. If you want to target multiple groups at the same time we suggest creating a group with child groups and targeting the parent group from this integration. You could also use xMatters userID's to target individuals. If you dont want to use this option set it to setting.xMatters_Groups = []; Be sure to do the same for setting.Speak_Groups. |
| setting.Speak_Groups    | Your group names may not always be easy for the Twilio text-to-speech engine to read. This option lets you type your groups names in the same order as in **settings.xMatters_Groups** but spell them in a way that the text-to-speech engine understands. You must have the same number of groups listed here in the same order as **settings.xMatters_Groups**. Failing to do so will cause unintended behaviours.                                                                                                                        |

<br><br>

### Bitley Access Token

```js
// Bitley Access Token
setting.bitly_token = 'scasoiueco23a432jcndl3s43a4cjdsalijfaweiud';
```

## Create a Bitly Account

- Use your Bitly Access Token generated [here](#create-a-bitly-account).

<br><br>

## Add Libraries to Twilio Configuration

1. Click the **Hamburger** icon.

   <kbd>
     <img src="/media/Burger.png">
   </kbd>

2) Scroll down to the bottom of the menu and go to **Runtime**.

   <kbd>
     <img src="/media/Runtime.png">
   </kbd>

3) Click on **Functions -> Configuration**.

   <kbd>
     <img src="/media/Function-Config.png">
   </kbd>

4. Import the following NPM Modules to your Twilio Application. There will be several dependencies already installed, make sure to leave these.

| Name    | Version |
| ------- | ------- |
| got     | 6.7.1   |
| request | 2.83.0  |

Once you have finished it should look like this:

  <kbd>
    <img src="/media/Dependencies.png" width="550px">
  </kbd>

5. Click **Save**.

<br><br>

## Configure Twilio Voice Number.

1. Click the **Hamburger** icon.

   <kbd>
     <img src="/media/Burger.png">
   </kbd>

2. Go to **Phone Numbers**.

   <kbd>
     <img src="/media/Menu-Phone-Numbers.png"  width="250px">
   </kbd>

3. Go to **Manage Numbers**.

   <kbd>
      <img src="/media/Menu-Manage-Numbers.png"  width="250px">
    </kbd>

4. Click on the phone number you just purchased.

5. In the **Voice & Fax** section configure what happens when a **Call Comes in**, select the function relating to xm_settings.

**CALL COMES IN**
FUNCTION
`01-xMatters-Notify-Settings`

\*\* If you named your functions differently that described in this integration **01-xMatters-Notify-Settings** may be different. You want to select the function for xm_settings.

<kbd>
  <img src="/media/Call-in.png" width="550px">
</kbd>

<br><br>

## Test the Integration.

1. Call your Twilio number.

2. Test your configuration and all of the call prompts making sure it works as expected.

<br><br>

# Troubleshooting

\*If you call your Twilio voice number and hear _"We are sorry but your application has an error"_ this is a problem with one of your Twilio functions or you have not configured your Twilio Voice number to call the **xm_settings** function when a call comes in.

\*If you call your Twilio voice number and hear _"Sorry you are not authorized for this, please speak to your xMatters admin"_ you should:

1. Ensure the phone calling your Twilio initiation number is **NOT** set to private calling. Twilio must be able to detect the caller id of the calling phone to properly determine if the calling phone is authorized to create an xMatters event.

2. Check the **Authorized_Users** variable on Line 43 of xm_settings script.

- Authorized_Users variable must list a valid xMatters UserID.
- One of the listed users must have a voice device configured in xMatters with a phone number that matches the caller id of the initiating phone.

3. Check **xm_user** and **xm_pass** variable on Lines 24 and 25 of xm_settings script.

   - If Twilio cannot authenticate into xMatters, it will not be able to properly detect if the initiating phone is authorized to create an event.

4. Check **xmatters** variable on Lines 21 of xm_settings script.

- This variable must contain the base url for xMatters and must not contain a trailing slash.
  example: 'https://acme.xmatters.com'

5. Disable authentication check. Not recommended as this will allow anyone to initiate an xMatters event.

Modify Line 201 of **xm_settings** to be:

```js
return 'Yes';
```

\*Most of the issue you could encounter have to do with the configuration of the **xm_settings** function in Twilio.
Use this Twilio resource for help with debugging functions:
https://www.twilio.com/docs/runtime/functions/debugging

\*Alternatively, you can check the Inbound Integration Activity Log in xMatters:
https://help.xmatters.com/ondemand/xmodwelcome/integrationbuilder/create-inbound-updates.htm
