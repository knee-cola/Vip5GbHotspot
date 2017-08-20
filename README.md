# Vip5GbHotspot

A Tasker project which turns your phone into a 5GB mobile hotspot

## What's this?

This is a Tasker project built for automatic renewal of [Vipme daily option (5GB)](http://www.vipnet.hr/privatni/mobilni-internet-na-bonove) for VIP prepaid broadband.

## What is Vipme daily option?

The Vipme broadband chgarges 1 HRK (Croatian Kuna) per megabyte, which is really expensive (1 GB costs 1,000 HRK = $200).

There is however a 24h option which includes 5GB of traffic and costs only 10 HRK.

## What's the problem?

The problem with the daily option is that once the 25 hour period ends the mobile operator automatically starts charging the expensive 1 HRK per MB fee. In other words the 5GB option is not extended automatically - it needs to be done manually.

## How does this project solve the problem?

This [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) project solves the problem by automating the manual 5GB option activation.

After the 24 h period has expired (or 5GB has been used up) it automatically re-activates the option via SMS.

User can select the period in which this task is to be done (i.e. for the next 7 days), which is really usefull when on holidays.

# How to install

1 Install [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) on your phone
2 download the XML project file on your phone
3 run Tasker
4 from menu select ``Data`` -> ``Restore`` -> ``Use Local Backup`` and select the downloaded XML file
5 add a **Tasker Shortcut** to you home screen - whe presented a task list select the **Vip5GbHotspot** 

# How to use

1 on your home screeen find the **Vip5GbHotspot** icon (created in the step 5 of the installation)
2 use the slider to select the end of the period in which you wish the 5GB daily option to be automatically extended
3 tap the **Activate** button

Immediately after tapping the **Activate** button the Tasker will do the following:

* it will send an SMS requesting activation of the 5GB daily option 
* it will disable mobile data and WiFi hotspot (so no mony is wasted until the option has been activated)
* upon receivend an SMS confirming that the option has been activated mobile data and portable WiFi hotspot will be automatically enables.

All the activity of the app can be monitored via SMS messages, which are exchanged duting it's operation.

# Technical docs

## App entry point

The entry point of the app is the **Vip5GbHotspot** task, which shows a screen (a Tasker scene) via which the app can be activated.

There are two screens:

* **SchedulerScene** - is shown while option period is not yet defined - it allows the service to be activated
* **StatusScene** - is shown while the option period is defined - it allows tohe service to be stopped

## Option activation sequence

Here's the activity sequence which occures when an option is activated:
* ``Vip5GbOptionRequest`` task is invoked, which sends an SMS to the broadband operator requesting the option activation
* the VipNet operator responds with an SMS, which contains list of options
* Tasker activates ``onHandShakeSMS`` profile which in turn runs the ``onOptionHandshake`` task
* the ``onOptionHandshake`` task send a new SMS selecting the daily 5GB option
* the VipNet responds by sending an SMS confirmin it has received the request

After the request has been issued it usually takes 5 minutes for the VipNet operator to activate the option:

* VipNet responds with a new SMS confirming that the option has been activated
* Tasker activates ``onConfirmationSMS`` profile which in turn runs the ``onOptionActivated`` task
* the ``onOptionActivated`` does the following
	* saves uses the current time to calculate the time at which the 24h period ends
	* enables the ``CronProfile5min`` profile, which will check if the 24h period has expired
	* enables mobile data and WiFi hotspot
	* notifies the user that the option has been activated

The period and option end are written to a file on SD card, so that the service can recover from a device being turned off / restarted.

## 24h option end detection

The option end is checked every 5 minutes by the ``CronProfile5min`` profile, which runs the ``CronTask`` task.

``CronTask`` first disables mobile data and WiFi hotspot, so that no money is wasted while the option is inactive.
It then compares the current time with the period end timestamp calculated in the ``onOptionActivated`` task. If the 24h option has expired, the task checks if the options should still be extended or the period end has been reached (the one set by the user).

If the option is to be extended then ``Vip5GbOptionRequest`` task is invoked and the procedure contines when the VipNet operator responds with an SMS (see **Option activation sequence**).

## Other tasks

Here's a list of all the tasks:

* ``Vip5GbHotspot`` - display the screen for the user - this is the entry point
* ``BootStartup`` - app initialization - is called then the device boots from the ``Device Boot`` profile
* scheduling tasks
	* ``CronInit`` - checks if the ``CronProfile5min`` profile should be enabled or not - is called from the ``BootStartup`` after the app has been initialized
	* ``CronStart`` -  enables ``CronProfile5min`` profile - is called whenb the user preses the **Activate** button
	* ``CronTask`` - check if the 24h optin has expired - is called every 5 minutes by the ``CronProfile5min`` profile
	* ``CronStop`` - stops the app and clears all the data
* ``Vip5GbOptionnRequest`` - sends an SMS requesting the activation of the 5GB option 
* event handler tasks
	* ``onOptionHandshake`` - sends a SMS selecting the 5GB option - is called from the ``onHandShakeSMS`` profile
	* ``onOptionActivated`` - calculates the 24h option period end time and enables the ``CronProfile5min`` profile - is called when an SMS confiming the option activation has been received
	* ``onOptionDepleted`` - activates a new 25h 5GB option - is called when an SMS is received informing us that the 5GB has been used up
	* ``onOptionActivationFail`` - notifies the user that the option activation has failed 
	* ``onNotifyClick`` - is called when the user dismisses the notification - activates the SMS app so that the user can check the cause of the problem 
* utility tasks
	* ``Write2File`` - saves the param value to the file (used at startup)
	* ``disableMobileData`` - disables mobile data and WiFi hotspot 
	* ``enableMobileData`` - enables mobile data and WiFi hotspot 
