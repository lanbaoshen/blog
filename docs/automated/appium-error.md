# Appium Error

When you choose Appium to do Android UI automated testing, 
you will definitely encounter many errors that you can't figure out, which will make the test impossible.

This article will summarize some errors and solutions I encountered in actual production using Appium.


## Switch User ID

When the first appium operation executed after my script switches the android user id, it raises the following error:

> Appium Settings app is not running after 5000ms

Obviously, when switching users, the previous user's appium process has ended. 
Appium need to reconnect device to perform operations on the new user. 

Because the apps (`io.appium.settings`, `io.appium.uiautomator2.server`, `io.appium.uiautomator2.server.test`) 
that appium depends on is installed in the previous user space, Appium will try to start directly instead of installing dependencies.

The solution is also very simple. After switching user id, just uninstall the app first.


## TCP:8200

When one of my team member was repeatedly running a long script (about ten hours), appium randomly throw the following error:

> An unknown server-side error occurred while processing the command. 
> Original error: 'POST /element' cannot be proxied to UiAutomator2 server because the instrumentation process is not running (probably crashed). 
> Check the server log and/or the logcat output for more details 

I check `appium.log` and found that, when the test ended due to the below error and appium was doing the teardown operation, 
appium try to remove device forward via command, but it failed:

> Unable to remove port forward 'Error executing adbExec. 
> Original error: 'Command 'adb -P 5037 -s emulator-5554 forward --remove tcp\:8200' exited with code 1'; 
> Stderr: 'error: listener 'tcp:8200' not found'; Code: '1''

When appium is running, it will forward `system: 8200` to `device: 6790`. After the test is completed, appium will remove this forwarding.

So this error is caused by the fact that the forwarding of `system: 8200` was removed during the test.

I checked the **adb process** in the system and the **appium process** in the device, and neither of them restarted or crashed. 

Then I checked the connection log with `dmesg -T`, I found that there was a log of USB reconnection before the error occurred.

So this is a hardware connection problem, just replace the USB cable and the problem is solved.


---

TODO ...

---
