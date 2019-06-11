---
title: "Updating Your Unity Project to Watson SDK for Unity 3.1.0 (and Core SDK 0.2.0)"
description: "A number of breaking changes hit all the Watson SDK’s for this year’s first major release, but your existing code on older SDK versions should continue working just fine. This means I’ve been working…"
date: "2019-05-07T15:01:01.021Z"
categories: 
  - Ibm Watson
  - Unity3d
  - Programming

published: true
canonical_link: https://medium.com/@MissAmaraKay/updating-your-unity-project-to-watson-sdk-for-unity-3-1-0-and-core-sdk-0-2-0-6c9a3748ef65
redirect_from:
  - /updating-your-unity-project-to-watson-sdk-for-unity-3-1-0-and-core-sdk-0-2-0-6c9a3748ef65
---

![Gaming Keyboard — Photo by [Aidan Granberry](https://unsplash.com/photos/ak4hw4r6xio?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/code?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](./asset-1)

A number of breaking changes hit all the Watson SDK’s for this year’s first major release, but your existing code on older SDK versions should continue working just fine.



This means I’ve been working on updating any of my existing content, which has taken me longer than anticipated. Which also means anything new I wanted to work on (AR Foundation) has been delayed too.

Unfortunately, I’m not able to release any of my [ARKit related content](https://developer.ibm.com/patterns/build-an-ai-powered-ar-character-in-unity-with-arkit/) until this gets fixed: [https://github.com/watson-developer-cloud/unity-sdk/issues/566](https://github.com/watson-developer-cloud/unity-sdk/issues/566)

And doubly unfortunate, it looks like there is a similar issue in Android land: [https://github.com/watson-developer-cloud/unity-sdk/issues/515](https://github.com/watson-developer-cloud/unity-sdk/issues/515)

If by chance you are NOT running on a mobile device (or Magic Leap) and you are interested in using the newly updated Watson SDK for Unity, I’ve put together some steps to get your project up and running. I’ll eventually package this as a repo, but ideally I’m waiting for the device specific issues to get sorted out so I can test on ARKit, and ultimately move to using AR Foundation.

#### Remove Old Watson SDK Version

I just delete the directory in the Project window. I love deleting code so directories are extra satisfying! Thank you for your service, but also, thank you, next.

#### Add New Watson SDK for Unity AND Core SDK

Two steps here, so mind these instructions. These steps work with Watson SDK for Unity 3.1.0 and Core SDK 0.2.0.

**Git clone the** [**Watson SDK for Unity**](https://github.com/watson-developer-cloud/unity-sdk) straight into your Assets directory (or if you want to be choosy, put it somewhere else and prune what you don’t want bloating your project, let the examples or tests/scripts you don’t plan on using).

**Git clone the** [**Core SDK**](https://github.com/IBM/unity-sdk-core) straight into your Assets directory. This is a new step, as some of the functionality has been moved into a separate repo. You need this to run the entire Watson SDK for Unity experience.

#### Change Namespace

Your existing source code will likely have namespaces that look like this:

```
using IBM.Watson.DeveloperCloud.Services.Assistant.v1;
using IBM.Watson.DeveloperCloud.Services.TextToSpeech.v1;
using IBM.Watson.DeveloperCloud.Services.SpeechToText.v1;
using IBM.Watson.DeveloperCloud.Widgets;
using IBM.Watson.DeveloperCloud.DataTypes;
using IBM.Watson.DeveloperCloud.Utilities;
using IBM.Watson.DeveloperCloud.Logging;
using IBM.Watson.DeveloperCloud.Connection;
```

These namespaces should continue to work, which is why you don’t have to update to the new SDK version if you don’t want to.

Depending on what you are doing, your namespaces may look something like this:

```
using IBM.Watson.Assistant.V2;
using IBM.Watson.Assistant.V2.Model;
using IBM.Watson.SpeechToText.V1;
using IBM.Watson.TextToSpeech.V1;
using IBM.Cloud.SDK;
using IBM.Cloud.SDK.Utilities;
using IBM.Cloud.SDK.DataTypes;
```

Notice the services sit in `IBM.Watson` and the helper functions and utilities sit in `IBM.Cloud.SDK`. If you are missing either one of the SDKs, you’ll see red squiggle lines. Make sure you have added both SDKs to your Assets folder and let Unity reload the project, importing the SDK files.

#### Remove Username/Password References

At this point, you should be using new services with API keys, or you should be in the process of migrating over your old services with username/password to new services and creating new credentials — which will give you an API key.



The only fields you really need for most services are the api key and the URL, and even the URL is considered optional depending on what region you are in. Depending on what location your service is in, your URL may change.

Watson Assistant should look something like this now:

```
[Header("Watson Assistant")]
     [Tooltip("The service URL (optional). This defaults to \"https://gateway.watsonplatform.net/assistant/api\"")]
     [SerializeField]
     private string AssistantURL;
     [SerializeField]
     private string assistantId;
     [Tooltip("The apikey.")]
     [SerializeField]
     private string assistantIamApikey;
```

You’ll notice a lot less checking of what was passed to the Assistant as we don’t have to check for the combination of credentials.

```
Credentials asst_credentials = null;
 TokenOptions asst_tokenOptions = new TokenOptions()
 {
   IamApiKey = assistantIamApikey,
 };
 
 asst_credentials = new Credentials(asst_tokenOptions, AssistantURL);
 
 while (!asst_credentials.HasIamTokenData())
 yield return null;
 
 _assistant = new AssistantService(“2019–02–08”, asst_credentials);
 
 _assistant.CreateSession(OnCreateSession, assistantId);
 
 while (!sessionCreated)
 yield return null;
```

This snippet hardcodes a version date of `2019–02–08`.

You’ll also need to create a callback function for OnCreateSession (I think this is particularly relevant for Assistant V2, not V1). Which looks something like this:

```
private void OnCreateSession(DetailedResponse<SessionResponse> response, IBMError error)
     {
         Log.Debug("AvatarPatternError.OnCreateSession()", "Session: {0}", response.Result.SessionId);
         sessionId = response.Result.SessionId;
         sessionCreated = true;
     }
```

You can put your credentials in a single file and add that to your .gitignore to prevent sharing your credentials inadvertently in a scene file, but that’s up to you.

#### Change Type Names & Callback Functions

This one is going to be the hardest to explain, so I recommend looking under the hood at the SDK examples and scripts to see how your code might have changed.

Some places I noticed these changes included the Synthesize method for Text to Speech.

```
private void CallTextToSpeech(string outputText)
     {
         Debug.Log("Sent to Watson Text To Speech: " + outputText);
 
         byte[] synthesizeResponse = null;
         AudioClip clip = null;
 
         _textToSpeech.Synthesize(
             callback: (DetailedResponse<byte[]> response, IBMError error) =>
             {
                 synthesizeResponse = response.Result;
                 clip = WaveFile.ParseWAV("myClip", synthesizeResponse);
                 PlayClip(clip);
 
             },
             text: outputText,
             voice: "en-US_AllisonVoice",
             accept: "audio/wav"
         );
     }
```

The object you pass into Watson Assistant looks a little different, if not cleaner. This example doesn’t include passing in the context object.

```
var input = new MessageInput()
{
   Text = "Hello"
};
 
_assistant.Message(OnMessage, assistantId, sessionId, input);
```

Notice that session id, again I think this might just be for Assistant V2, so double check me.

The object from Watson Assistant’s callback looks different too, `DetailedResponse<MessageResponse>` giving you fantastic dot notation when working with the JSON response.

```
string intent = response.Result.Output.Intents[0].Intent;
```

Error handling has also changed with the addition of an IBMError type.

Those seem to be the most memorable, but take a look at the SDKs for more information.

#### Wrap Up

Hopefully that gets you through updating to the new Watson SDKs for Unity. As I mentioned, once the device issues have been resolved for at least iOS, I’ll update my [code pattern](https://developer.ibm.com/patterns/build-an-ai-powered-ar-character-in-unity-with-arkit/) and ARKit content for the new SDKs.

Did you update your project? Or are you sticking with the old version? Let me know!
