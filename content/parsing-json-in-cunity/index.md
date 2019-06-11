---
title: "Parsing JSON in C#/Unity"
description: "In a world where almost everything is exposed with a REST API, JSON parsing becomes skill number 1 (unless you get helper/wrapper functions in an SDK or find some cheeky asset that does this for you…"
date: "2018-08-23T22:35:49.126Z"
categories: 
  - Csharp
  - Unity3d
  - Json
  - Programming

published: true
canonical_link: https://medium.com/@MissAmaraKay/parsing-json-in-c-unity-573d1e339b6f
redirect_from:
  - /parsing-json-in-c-unity-573d1e339b6f
---

![Minified and Compressed Code (not C#) Photo by [Markus Spiske](https://unsplash.com/photos/8OyKWQgBsKQ?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/coding?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](./asset-1)

In a world where almost everything is exposed with a REST API, JSON parsing becomes skill number 1 (unless you get helper/wrapper functions in an SDK or find some cheeky asset that does this for you, but this isn’t about that!). What happens if you have to do it by hand?

Here’s a quick walkthrough for parsing JSON into something you can work with in C# land (which happens to include Unity).

#### Call It

Using Postman or ARC (Advanced Rest Client) figure out what the response looks like. I use ARC for something like this.

Plug in the request URL, make sure the method says POST, supply the proper header and body info:

![Advanced REST Client with a POST](./asset-2.png)

Now we have the response JSON.

#### \[Optional\] Validate It

Copy & paste sometimes backfires like it did to me and I had to use a JSON linter to realize my “ wasn’t a really a “. You might encounter other bad JSON things, so try something like [JSONLint](https://jsonlint.com/).

You may continue.

#### Tranform It

This JSON is pretty simple looking, but some of them are not. In order to use Unity’s built in [JsonUtility](https://docs.unity3d.com/ScriptReference/JsonUtility.html), I need to build a class (or classes) with my JSON structure.

Again, this is a simple JSON so this is partially overkill. I used [json2csharp](http://json2csharp.com/) to generate my classes.

![json2csharp generating classes from JSON](./asset-3.png)

Copy this.

#### Build It

Unity, specifically JsonUtility, is a special beast, so we can’t just copy over what json2csharp gave us, but we are close. Clean up the name “_\_\_invalid\_name\_\_index_” so it just says “index” and get ride of all the _{ get; set; }_ because those are going to give you nothing but trouble here.

![C# classes for JSON](./asset-4.png)

“RootObject” made exactly no sense to me, so I renamed it MaxResults. I nested my classes as well.

#### Code It

Finally, we actually have to use this thing in the rest of the code, right? Right!

![C# code for working with UnityWebRequest returning JSON](./asset-5.png)

I take my response string, use _JsonUtility.FromJson_ to turn it into classes, and then use some sweet dot notation to get what I need, in this case the first prediction’s caption will do nicely.

#### Run It

![A Deep Learning model thinks this is “a red and white fire hydrant sitting on the side of a road” but its my Unity project for taking a webcam picture and doing image caption generation. I’m holding a Chick-fil-a cup to the camera.](./asset-6.png)

Clearly, we nailed it. 🚒

See my [last post](https://blog.goodaudience.com/unity-max-model-asset-exchange-b4fc0a0f3f1d) if you are confused. I didn’t train this model.
