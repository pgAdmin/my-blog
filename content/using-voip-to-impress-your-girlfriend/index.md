---
title: "Using VOIP to impress your girlfriend"
description: "Back in college (almost 7 years ago), I was a nerd and internet-based solutions were part of my daily life. Be it mobile subscriptions…"
date: "2020-03-07T14:58:42.990Z"
categories: []
published: true
canonical_link: https://medium.com/@punit.g7/using-voip-to-impress-your-girlfriend-524517cf9982
redirect_from:
  - /using-voip-to-impress-your-girlfriend-524517cf9982
---

Back in college (almost 7 years ago), I was a nerd and internet-based solutions were part of my daily life. Be it mobile subscriptions, online shopping, movie streaming or banking. You name it and at some point, I would have used it.

The 20th birthday of my then-girlfriend was approaching and I had to figure out a creative way of making it special for her. So, I thought of giving cloud telephony or VOIP a try.

The idea was to create a computer program that would do the following:  
1\. Trigger a phone call at exactly 11:59 pm  
2\. Play a customized “Happy Birthday” song for her  
3\. Then trigger a phone call to me and patch the connection to the original call, for me to talk to her

To execute this plan, first I had to find a VOIP provider that was **_not_** exclusive to enterprises and had a pay-as-you-go plan. It should provide service to Indian customers and should not ask for a credit card or provide free credits. Also, it must have an SDK in PHP or Python (those were the only two languages I knew).

After going through a dozen of websites, I shortlisted 3 probable options — [telapi.com](http://telapi.com) (now, [zang.io](http://zang.io)), [plivo.com](http://plivo.com) and [twilio.com](http://twilio.com)

Twilio.com had been around for a while but required a credit card.  
Telapi.com had recently been acquired by Zang, had shutdown any free plans and was exclusive to the US at that time.  
Plivo.com was a perfect option because it had a pay-as-you-go plan and only required you to buy credits before usage. They even allowed whitelisting a few numbers that could be used for testing without getting charged.

Plivo has very straightforward documentation explaining the different parts of a call process. Authentication is performed with a par of auth\_id and auth\_token provided in their dashboard.

undefined

With basic knowledge of HTTP requests and XML, anyone could use it to implement the program. I started by learning how to make a [call](https://www.plivo.com/docs/voice/detailed-reference/api/call#the-call-object). As the first step, I set up a cron job to run this python script at 11:59 pm

```
import plivo

client = plivo.RestClient(
  auth_id='your_auth_id',
  auth_token='your_auth_token'
)
my_phone_number = '+919898989898'
her_phone_number = '+918989898989'
answer_url = 'https://example.com/assets/steps.xml'

response = client.calls.create(
  from=my_phone_number,
  to=her_phone_number,
  answer_url=answer_url,
  answer_method='POST'
)
```

The response of the above request is a JSON which looks like:

```
{
  "message": "call fired",
  "request_uuid": "9834029e-58b6–11e1-b8b7-a5bd0e4e126f",
  "api_id": "97ceeb52–58b6–11e1–86da-77300b68f8bb"
}
```

Note: For applying any further actions on the active call programmatically, \`request\_uuid\` is very important.

The contents of **answer\_url (**https://example.com/assets/steps.xml**)** are invoked once the receiver picks up the call.  
The 2nd step was to play the song. This is how the XML file would look:

```
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  <Play>https://example.com/assets/happy_birthday.mp3</Play></Response>
```

The 3rd step was for it to call me once the song completes. So I had to modify it slightly like this:

```
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  <Play>https://example.com/assets/happy_birthday.mp3</Play>
  <Dial callerId="+919898989898">
    <Number>+919898989898</Number>
  </Dial>
</Response>
```

To make it a little fancier, I also introduced a speech action where a robotic voice said “Connecting this call to Punit”. The update XML file looked like this:

```
<?xml version="1.0" encoding="UTF-8"?>
<Response>
  <Play>https://example.com/assets/happy_birthday.mp3</Play>
  <Speak>Connecting this call to Punit</Speak>
  <Dial callerId="+919898989898">
    <Number>+919898989898</Number>
  </Dial>
</Response>
```

---

It worked pretty smoothly. She was indeed surprised with this unique plan.

Stay tuned for more of my tech hacks!
