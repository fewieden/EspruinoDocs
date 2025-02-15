<!--- Copyright (c) 2019 Gordon Williams, Pur3 Ltd. See the file LICENSE for copying permission. -->
Gadgetbridge for Android
=========================

<span style="color:red">:warning: **Please view the correctly rendered version of this page at https://www.espruino.com/Gadgetbridge. Links, lists, videos, search, and other features will not work correctly when viewed on GitHub** :warning:</span>

* KEYWORDS: Gadgetbridge,Gadget bridge,Android,Notifications
* USES: Bangle.js

[Gadgetbridge](https://gadgetbridge.org/) is an Android application that allows you to use smartwatch-style notifications and health monitoring without the need for a proprietary application or web service. We also have `Bangle.js Gadgetbridge` on the Google Play store (see below)

*If you like Gadgetbridge, [please consider donating](https://liberapay.com/Gadgetbridge/donate)
to help support its continued development*


How to set up
-------------

* Install [Gadgetbridge](https://f-droid.org/packages/nodomain.freeyourgadget.gadgetbridge/) on your Android phone. See below for info on our `Bangle.js Gadgetbridge` app.
* On Bangle.js, install ONE OF (not both!):
  * [Android Integration app](https://banglejs.com/apps/#android) - this is the new and recommended way of interfacing to Gadgetbridge, which allows you to view all notifications in a list
  * [The Gadgetbridge Widget](https://banglejs.com/apps/#gbridge) - this is the old way of interfacing to Gadgetbridge - it displays just one notification at a time.
* Now ensure you disconnect your computer from Bangle.js
* Start the Gadgetbridge app and click the blue `+` in the bottom right
* Choose your Bangle.js device from the list
* Right now we'd suggest choosing `Don't pair` when prompted in order to get the most reliable connection
* Everything should now be working. From the menu in the top-left of the Gadgetbridge Android app you can choose `Debug` and can test out notifications/etc

**Does Gadgetbridge keep disconnecting from your Bangle?** It may be that your phone is doing some 'battery usage optimisation' and deciding that Gadgetbridge should be shut down. See https://dontkillmyapp.com/ for device-specific advice on how to stop this happening.

Bangle.js Gadgetbridge app
----------------------------

We are currently developing a slightly altered Gadgetbridge app for Bangle.js that can be listed on the Play Store.

To try it go to https://play.google.com/apps/testing/com.espruino.gadgetbridge.banglejs and click the button, then you can install straight from Google Play!

### HTTP requests

**Must be enabled first** by clicking the gear icon next to the Bangle.js you're connected to in Gadgetbridge, and then enabling `Allow Internet Access`

On Bangle.js send something like:

```
Bluetooth.println(JSON.stringify({t:"htt­p", url:"https://192.168.1.14/bangletest",xpath:"­/html/body/p/div[3]/a"}));
```

And Gadgetbridge will call `GB({t:"http",resp:"......"})` with the response. Right now you *must* use HTTPS (HTTP is not supported).


### Intents

**Must be enabled first** by clicking the gear icon next to the Bangle.js you're connected to in Gadgetbridge, and then enabling `Allow Intents`

On Bangle.js send something like:

```
Bluetooth.println(JSON.stringify({t:"int­ent",action:"com.sonyericsson.alarm.ALAR­M_ALERT",extra:{key1:"asdfas"}}));
```

Will send a Global Android intent. This can open cause certain apps/windows to open, or can be used with apps like `Tasker`.


### Weather

You can also get weather from Gadgetbridge. Install the [Weather Widget](https://banglejs.com/apps/#weather) and check out the `Read more...` link on the app page for more information. An additional app is required to forward the current weather into Gadgetbridge.

If you're using the Play Store 'Bangle.js Gadgetbridge' app, you need to set the package name to `com.espruino.gadgetbridge.banglejs`


How it works internally
--------------------------

### Messages sent to Bangle.js from Phone

Messages are wrapped in the text `"\x10" + "GB(...)\n"`, so that if they're
sent to a normal Espruino REPL the `GB` function will be executed with the
supplied data as the first argument.

Currently implemented messages are:

* `t:"notify", id:int, src,title,subject,body,sender,tel:string`  - new notification
* `t:"notify-", id:int`  - delete notification
* `t:"alarm", d:[{h:int,m:int,rep:int},...]`  - set alarms (rep=binary mask of weekdays)
* `t:"find", n:bool`  - findDevice
* `t:"vibrate", n:int`  - vibrate
* `t:"weather", temp,hum,txt,wind,loc`  - weather report
* `t:"musicstate", state:"play/pause",position,shuffle,repeat` - music play/pause/etc
* `t:"musicinfo", artist,album,track,dur,c(track count),n(track num)` - currently playing music track
* `t:"call", cmd:"accept/incoming/outgoing/reject/start/end", name: "name", number: "+491234"` - call
* `t:"act", hrm:bool, stp:bool, int:int`  - Enable realtime step counting, realtime heart rate. 'int' is the report interval in seconds

For example:

```
// new message
GB({"t":"notify","id":1575479849,"src":"Hangouts","title":"A Name","body":"message contents"})
// message changed
GB({"t":"notify~","id":1575479849,"body":"this changed"})
// remove message
GB({"t":"notify-","id":1575479849}); 
// maps navigation
GB({"t":"notify","id":1,"src":"Maps","title":"0 yd - High St","body":"Campton - 11:48 ETA","img":"Y2MBAA....AAAAAAAAAAAAAA="})
// music
GB({"t":"musicstate","state":"play","position":0,"shuffle":1,"repeat":1})
GB({"t":"musicinfo","artist":"My Artist","album":"My Album","track":"Track One","dur":241,"c":2,"n":2})
// Call coming in 
GB({"t":"call","cmd":"accept","name":"name","number":"+441234123123"})
// Set a single alarm, 6:30am, every day of the week
GB({"t":"alarm", "d":[{"h":"6","m":"30:","rep":127}]})
```

### Messages from Bangle.js to Phone

Any line beginning with `{` will be parsed as JSON by Gadgetbridge, so to
send a command, simply use `Bluetooth.println(JSON.stringify(json))`.

Available message types are:

* `t:"info", msg:"..."` - display info popup on phone
* `t:"warn", msg:"..."` - display warning popup on phone
* `t:"error", msg:"..."` - display error popup on phone
* `t:"status", bat:0..100, volt:float(voltage), chg:0/1` - status update
* `t:"findPhone", n:bool`
* `t:"music", n:"play/pause/next/previous/volumeup/volumedown"`
* `t:"call", n:"ACCEPT/END/INCOMING/OUTGOING/REJECT/START/IGNORE"`
* `t:"notify", id:int, n:"DISMISS,DISMISS_ALL/OPEN/MUTE/REPLY", `
  * if `REPLY` can use `tel:string(optional), msg:string`
* `t:"ver", fw1:string, fw2:string` - firmware versions - sent at connect time
* `t:"act", hrm:int, stp:int` - activity data - heart rate, steps since last call

For example:

```
Bluetooth.println(JSON.stringify({t:"info", msg:"Hello World"}))
```

will display a message on your phone's screen.

### Debugging

There's a [Gadgetbridge Debug](https://banglejs.com/apps/#gbdebug) app you can install. When running this
will show you the data that is being received from Gadgetbridge. See `Read more...` next to the app for
more information.

You can then disconnect Gadgetbridge, connect with the Web IDE and issue that command by pasting it 
into the left-hand side of the IDE - for example:

```
GB({"t":"notify","id":1575479849,"src":"Hangouts","title":"A Name","body":"message contents"})
```

If there are any errors shown you'll be able to see them and use them to debug what is happening.


Building Gadgetbridge
------------------------

If you want to build Gadgetbridge yourself there's proper documentation at https://codeberg.org/Freeyourgadget/Gadgetbridge/wiki/Developer-Documentation

Once you have the Android development tools on your system, all you need to do to build is:

```Bash
./gradlew assembleDebug
adb install app/build/outputs/apk/debug/app-debug.apk
```
