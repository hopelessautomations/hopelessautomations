---
layout: post
title: Zemismart M515EGB
subtitle: Updated Roller Shade Driver
tags: [Smarthome, Adventures, Tuya, Zemismart]
share-img: /assets/img/zemismart_logo1.jpeg
thumbnail-img: /assets/img/zemismart_logo1.jpeg
---
<div class="alert alert-info">
  <strong>Info!</strong> The post is not complete yet.
</div>

![Zemismart Curtains and Blind motors](/assets/img/zemismart_curtains.png)

Let the battle for the best, cheaper and easier to integrate locally with Home Assistant begin!

I have recently decided to install smart curtains / blinds on a few of my many windows.  
Especially for the room where I have my parrots if the curtains can open and close automatically at set times I can have the parrots quiet until the time is right, otherwise they start "living" when the sun rises.  

Having looked around on my favorite online store (aliexpress) I saw there are plenty of choices right now but my choices are always go towards the ones I know alright it can be hacked (WiFi and Zigbee for me)  

The first room I want to complete with smart curtains and blinds is ofcourse the room where I have the Parrots, it's a very large Living Room with 5 windows and a 'big open door' dividing the room.

2 of the 5 windows already have normal (dumb) blinds with a bead string to open and close manually, I plan on using this motor on one of them and see how it goes. On the 'big open door' I bought a 3.2 meter Zigbee curtains rail and some nice long curtains to close it up and the rest of the windows I bought a blinds kit on Amazon.DE
First window I decided to modify already had regular blinds that open and close rotating a bead string so I decided to buy the newest version for it from Zemismart.  
The device wasn't yet added to [Supported Tasmota Devices](https://templates.blakadder.com/all.html) so the calling was strong.

## Zemismart M515EGB the Updated Roller Shade Driver

I was surprised the item arrived to Europe under 2 weeks.
Packaging looked great and solid and my only real complain about it (and from what I've experienced seems to be a brand problem) are the English manuals or the lack of them.  
This one actually had a rough translation to English but some others I have bought.. let's just say I was happy they weren't my firsts and I already knew more or less how to operate them.  

First thing after unboxing was clearly starting up tuya-converter and see if I could backup the stock firmware and also flash it OTA.  
The poor English instructions only made it harder to understand how to pair the device with the USB WiFi stick and the RF remote controller that came with it. But after that I could confirm on tuya-converter that OTA flashing was possible and without further thinking and decided to flash Tasmota on it.  

After successfully flashing Tasmota and connect it to my network I opened up Tasmota console and setup Tuya MCU as module and configured the Web log level to More debug with <code>Backlog weblog 4; module 54</code> waited for it to reboot and for my surprise there weren't any Tuya messages showing up.

As always I have put the "cart before the horse" only making it harder on myself, but OK.. It's not like it's the first time.. lol

Having managed to open the USB WiFi stick fairly easy I decided to flash the stock firmware back and start from the beginning debugging the original messages from the APP to the device itself.

After connecting to Tuya App I noticed my problem was how to connect the stick to the motor itself which toke awhile to figure out but having that done and connecting the esp8266 to my FTDI stick I saw the normal Tuya messages on the terminal.

Saved all the messages incase I needed later on and flashed Tasmota OTA again with tuya-converter leaving the usb stick paired with the device.  
All that done I sended the same backlog command as before <code>Backlog weblog 4; module 54</code> and Tuya messages started floating on my console.  
I have noticed that during my debug on the Tuya app I had a lot more DpIDs than with Tasmota but all the important ones were here and working.

### Tasmota
On Tasmota console this was the log for `SerialSend5 55aa0001000000`.
~~~
CMD: SerialSend5 55aa0001000000

MQT: tele/Zemismart_M515EGB/RESULT = {"TuyaReceived":{"Data":"55AA0301002A7B2270223A227135796D68746F613673646A796D7778222C2276223A22312E302E30222C226D223A307D73","Cmnd":1,"CmndData":"7B2270223A227135796D68746F613673646A796D7778222C2276223A22312E302E30222C226D223A307D"}}
TYA: MCU Product ID: {"p":"q5ymhtoa6sdjymwx","v":"1.0.0","m":0}

MQT: tele/Zemismart_M515EGB/RESULT = {"TuyaReceived":{"Data":"55AA03070005010400010115","Cmnd":7,"CmndData":"0104000101","DpType4Id1":1,"1":{"DpId":1,"DpIdType":4,"DpIdData":"01"}}}
TYA: fnId=0 is set for dpId=1

MQT: tele/Zemismart_M515EGB/RESULT = {"TuyaReceived":{"Data":"55AA03070008020200040000000019","Cmnd":7,"CmndData":"0202000400000000","DpType2Id2":0,"2":{"DpId":2,"DpIdType":2,"DpIdData":"00000000"}}}
TYA: fnId=0 is set for dpId=2

MQT: tele/Zemismart_M515EGB/RESULT = {"TuyaReceived":{"Data":"55AA03070005050100010015","Cmnd":7,"CmndData":"0501000100","DpType1Id5":0,"5":{"DpId":5,"DpIdType":1,"DpIdData":"00"}}}
TYA: fnId=0 is set for dpId=5

MQT: tele/Zemismart_M515EGB/RESULT = {"TuyaReceived":{"Data":"55AA0307000507040001001A","Cmnd":7,"CmndData":"0704000100","DpType4Id7":0,"7":{"DpId":7,"DpIdType":4,"DpIdData":"00"}}}
TYA: fnId=11 is set for dpId=7

MQT: tele/Zemismart_M515EGB/RESULT = {"TuyaReceived":{"Data":"55AA030700050A050001001E","Cmnd":7,"CmndData":"0A05000100","DpType5Id10":"0x00","10":{"DpId":10,"DpIdType":5,"DpIdData":"00"}}}
TYA: fnId=0 is set for dpId=10
~~~

<p>The dpId chart:</p>

: -- : | : ---------- | : ------------------------
 dpId  |  Properties  |  Function 
1      | 0 / 1 / 2    | Open/Stop/Close
2      |              | Set motor position
5      | 0 / 1        | Change motor direction
7      |              | *not confirmed
10     | 0x00 or 0x01 | 0x00 OK / 0x01 Battery Low

The dpId 2 the properties are not very relevant here because in Tasmota I have mapped it as a Dimmer with `TuyaMCU 21,2` and mapped the on/off button to a unused dpId `TuyaMCU 11,7`and `So59 1`for better Home Assistant configuration.

### Home Assistant
After all was configured in Tasmota side I made it simple and added it to Home Assistant as a cover using the Dimmer as the position.

Here's my example:
{% highlight yaml %} 
cover:
  - platform: mqtt
    name: "Curtains Zemismart M515EGB"
    device_class: shade 
    command_topic: "cmnd/Zemismart_M515EGB/TuyaSend4"
    payload_open: "1,0"
    payload_close: "1,2"
    payload_stop: "1,1"
    position_open: 100
    position_closed: 0
    position_topic: "tele/Zemismart_M515EGB/STATE"
    value_template: '{{ value_json.Dimmer }}' 
    set_position_topic: "cmnd/Zemismart_M515EGB/Dimmer"
    set_position_template: '{{ 100 - position | int }}'
    availability_topic: "tele/Zemismart_M515EGB/LWT"
    payload_available: "Online"
    payload_not_available: "Offline"
    optimistic: true
{% endhighlight %}

Since the motor direction can be set using the RF controller remote and is basically a one time configuration I didn't bother adding it to my Home Assistant and the battery (dpId 10) is part of notification system a story for another post.
