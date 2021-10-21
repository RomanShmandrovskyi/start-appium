# Appium setup
All next steps that have `MacOS only` words are mandatory to run tests on iOS.  

Note: you might need to run commands with superuser rights - `sudo`

1. Install Homebrew `Mac OS only`:
   ```
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
   ```

2. Install ffmpeg:
   ```
   brew install ffmpeg
   ```

3. Install node.js and npm:  
   Mac OS:
   ```
   brew install node
   ```  
   Ubuntu:
   ```
   apt install nodejs
   ```  
   Check node version: `node -v`  
   Check npm version: `npm -v`

4. Install appium-doctor:
   ```
   npm install appium-doctor -g
   ```  
   Run it in terminal (command: `appium-doctor`) to see what is needed for appium correct work.  
   Run each time after config something and until necessary dependencies will be installed (not forget run command in new terminal);

5. Install Carthage `Mac OS only`:
   ```
   brew install carthage
   ```

6. Install Xcode `Mac OS only`: find on App Store;
7. Install Android Studio: official [web-site](https://developer.android.com/studio) or [JetBrains Toolbox](https://www.jetbrains.com/toolbox-app/);
8. Set env variables: `JAVA_HOME` and `ANDROID_HOME`;
9. Modify env variable `PATH` by adding `$JAVA_HOME/bin`, `$ANDROID_HOME/tools` and `$ANDROID_HOME/platform-tools`  
   You must set this in `.bash_profile` file for Mac OS or in `.bashrc` for Ubuntu. Create this file if not exist: `touch .bash_profile` in your User folder.   
   Your `.bash_profile` file must have this lines:

     ```shell
     #some script code here
     
     export JAVA_HOME=<path_to_jvm>
     export ANDROID_HOME=<path_to_android>
     export PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platfrom-tools:$JAVA_HOME/bin
     ```   

10. Install Appium:
    ```
    npm install -g appium --unsafe-perm=true --allow-root
    ```

11. Try to run appium by executing in terminal:
    ```
    appium
    ```  
    It must start appium server on your machine and respond something like:
    ```
    [Appium] Welcome to Appium v1.19.1
    [Appium] Appium REST http interface listener started on 0.0.0.0:4723
    ```  
    
## Android configurations
### Device configs
1. Go to `Settings > About phone` swipe to the end to find `Build number` option. Try to tap several times until you not see the message: `You are now a developer!`;
2. Now go to `Settings > System > Advanced > Developer options` and enable `USB debugging` option;
3. It will be fine also disable screen locking. Go to `Settings > Security > Screen lock` and choose `None`.

### Machine configs
1. Install `ADB` if not installed with Android Studio (try to run: `adb --version` in terminal):  
   Mac OS:
   ```
   brew cask install android-platform-tools
   ```
   Ubuntu:
   ```
   sudo apt-get install android-tools-adb
   ```
2. Try to create and run some Android simulator (in AVD Manager) or connect a real device via USB and run in terminal:
   ```
   adb devices
   ```  
   Must respond something like:
   ```
   List of devices attached
   emulator-5554     device
   94SAX0EN23        device
   ```  
   First value - `emulator-5554` - `UDID` of android device (or simulator)

3. Keep `chromedriver` version up to date located in `src/main/resources/driver`.  
   You can download [here](https://chromedriver.storage.googleapis.com/index.html) version you need (on the device in Chrome check in `Settings > About Chrome > Application version`).    
   **Note: it depends on your OS also. There are different drivers for Ubuntu and Mac OS**
   
## IOS configurations
You can find [here](https://github.com/appium/appium-xcuitest-driver/blob/master/docs/real-device-config.md) appium docs with screenshots.

### Device configs
Next configurations make sense only for **real** iOS devices.   
All steps are already included if you want to run tests on iOS simulator.

1. Connect your iPhone to Mac via lightning and unlock it.
2. Open Xcode. Go `Window > Devices and Simulators`. Verify that your iPhone is displayed on the left panel, there are no errors and wait for your iPhone be recognised.
   Try to execute `instruments -s devices` in terminal to see your device name and UDID.
   If there are some errors try click rmb on device name in `Devices and Simulators` left menu and choose `Unpair Device`, then reject cabel from iPhone and connect it again.
3. On iPhone open `Settings > Developer` and here enable `Enable UI Automation`.
4. As for an android device you can change screen locking options. Go to `Settings > Face ID & Passcode > Turn Passcode Off` and follow instructions.

### Mac OS configurations
1. Go to folder where appium was installed (by default: `usr/local/lib/node_modules/appium`). Open Appium `node_modules` folder here.
2. Find `appium-webdriveragent` folder, open terminal inside and execute next commands:
   ```
   mkdir -p Resources/WebDriverAgent.bundle
   ```
3. Open file `WebDriverAgent.xcodeproj` via Xcode.  
   If there are some alerts with `File is locked…` click `Don’t Unlock` each time.
4. In Xcode on left side (with file tree) choose root project file - `WebDriverAgent` - then on right side in targets click on `WebDriverAgentLib`.
5. Go to `Signing & Capabilities`, select `Automatically manage signing` checkbox (if not selected) and select a `Team`.  
   If there is no team - add your own (or create new one) Apple ID, but in this way you will need to update Provisioning Profile each week. The best way is to join the project development team (if it exists) on `developer.apple.com`.  
6. Choose `WebDriverAgentRunner` target and do the same in `Signing & Capabilities`.
7. You likely will have an error: `Failed to register bundle identifier`.  
   If yes - go to `Build Settings > Packaging > Product Bundle Identifier`.  
   Try to change it value on some other (ex. `com.facebook.WebDriverAgentRunner` -> `any.other.id.you.want`).   
   Go back to `Signing & Capabilities` each time to verify if errors fix.  
   **Note: you can create only 10 unique ids per week on one Apple ID!!!**
8. When you find some free bundle ID, go back to `WebDriverAgentLib` and change it bundle ID to the same as in `WebDriverAgentRunner` (ex. `com.facebook.WebDriverAgentLib` -> `any.other.id.you.want`).
9. Go to `IntegrationApp` target and do the same things here (select `Automatically manage signing`, `Team` and use the same bundle ID).
10. In Xcode go to `Product > Clean Build Folder` and then `Product > Build`. Wait on alert `Build Succeeded`.  
    Xcode will ask to enter your System password several times.
11. Go back to terminal (you must still be in `appium-webdriveragent` folder) and execute next command:
    ```
    xcodebuild -project WebDriverAgent.xcodeproj -scheme WebDriverAgentRunner -allowProvisioningUpdates -destination 'id=<your_device_UDID>' test
    ```  
    In command change `<your_device_UDID>` to your device UDID.
12. (Real device only) If you provide your own Apple ID in Xcode `Signing & Capabilities` wait until testing failed ( TEST FAILED ).   
    In your iPhone applications list try to find `WebDriverAgentRunner-Runner`.  
    Try to open it, you must get an alert with message: `Untrusted Developer`. Click cancel.  
    On device go to `Settings > General > Profiles & Device Management`.  
    In topic `DEVELOPER APP` click on `“Apple Development: your_apple_id@host.com”`, then click on `Trust “Apple Development: your_apple_id@host.com”` and confirm it and execute command one more time.

    If you provide your project team account it must build from the first time. If not - rebuild one more time or reboot your Mac.

13. If testing successful you should have the output like:
    ```  
    Test Case '-[UITestingUITests testRunner]' started.
        t =     0.00s Start Test at 2020-08-10 16:42:57.571
        t =     0.00s Set Up
    ```  
14. `Optional` Open new terminal tab and run:
    ```
    curl -X GET -H "Content-Type: application/json;charset=UTF-8, accept: application/json" http://<device_IP>:8100/status
    ```
    Should respond something like:
    ```json
    {
      "value" : {
        "message" : "WebDriverAgent is ready to accept commands",
        "state" : "success",
        "os" : {
          "testmanagerdVersion" : 28,
          "name" : "iOS",
          "sdkVersion" : "13.4",
          "version" : "13.6"
        },
        "ios" : {
          "ip" : "192.168.0.104"
        },
        "ready" : true,
        "build" : {
          "time" : "Aug 10 2020 16:31:52",
          "productBundleIdentifier" : "com.facebook.WebDriverAgentRunner"
        }
      },
      "sessionId" : null
    }
    ```
    To get `device_IP`, go on device to `Settings > WiFi > info icon of current network > IP Address`. WiFi network must be the same as on your machine.
