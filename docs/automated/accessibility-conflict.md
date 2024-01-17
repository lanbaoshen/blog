# Accessibility Conflict

Automated test rack requires the injection of rotary input events using the car_service command, follow [Set up a rotary controller](https://source.android.com/docs/automotive/hmi/rotary_controller/app_developers?hl=en).

## Issue

The car_service command can work normally:

<!-- termynal -->

```
$ adb shell cmd car_service inject-rotary

Succeeded
```

However, when Appium is running, some commands to move focus do not work 
(Commands that do not change focus still work), but the stdout is still `Succeeded`.

After I force-stop `io.appium.uiautomator2.server`, all the commands work normally.  

Based on the phenomena, I guess there are two possibilities:

:   * Appium process intercepts the rotary input events injection.
    * The underlying service conflicts.

## Solution

Before this, I already knew that Appium were ultimately implemented based on Accessibility.  

I've tried to learn how to implement Focus from [Developing Apps without the Car UI Library](https://source.android.google.cn/docs/automotive/hmi/rotary_controller/app_developers_no_carui).

Luckily, I got important information in this doc, FocusParkingView and FocusArea are based on Accessibility too!

??? Quote
    
    **Implement FocusParkingView**

    You either can implement your own `FocusParkingView` or copy the class from the car-ui-library to your project.
    
    To implement `FocusParkingView`:
    
    Hard code the accessibility class name so that the `RotaryService` can recognize it:
    
    ```java
    @Override
    public CharSequence getAccessibilityClassName() {
        return "com.android.car.ui.FocusParkingView";
    }
    ```
    
    **Implement FocusArea**

    Like `FocusParkingView`, you can either implement your own `FocusArea` or copy the class from the car-ui-library to your project.
    
    To implement `FocusArea`:
    
    Hard code the accessibility class name so that rotary service can recognize it:

    ```java
    @Override
    public CharSequence getAccessibilityClassName() {
        return "com.android.car.ui.FocusArea";
    }
    ```

So, the root cause of this problem is Appium suppress Accessibility.  

Knowing why, I found a solution in [appium-uiautomator2-driver](https://github.com/appium/appium-uiautomator2-driver), set `appium:disableSuppressAccessibilityService` `ture` when init driver  

??? Quote

    |              Capability Name	              |                                                                                                 Description                                                                                                  |
    |:------------------------------------------:|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
    | appium:disableSuppressAccessibilityService | Being set to `true` tells the instrumentation process to not suppress accessibility services during the automated test. This might be useful if your automated test needs these services. `false` by default |

Everything is ok now! 🎉
