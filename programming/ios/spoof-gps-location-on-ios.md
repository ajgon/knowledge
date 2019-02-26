# Spoof GPS location on iOS

* Attach iOS device to your mac and trust it.
* Install XCode.
* Create a new project in XCode.
    * Single View App.
    * Make Bundle Identifier unique (easiest way is to use some uncommon string for Organization Identifier).
    * Do not include Core Data, Unit and UI Tests.
* In "General" tab:
    * In "Signing" submenu:
        * Tick on "Automatically manage signing"
        * Click "Add Account...", and add your personal iCloud account.
        * Set Team as your iCloud account ("Personal Team" type).
* In upper-left corner, select your device as a target for built app.
    ![](../.gitbook/assets/9e954be8-18e7-48d8-b62d-6698b2b226a6/4618e3b4.png)
* "Product"-\>"Run"
    * If your developer certificate is not verified, on your iOS device, go to "General"->"Device Management", then select
      your Developer App certificate to trust it.
    * Re-run app if necessary.
* When app succesfully launches, click "Location" icon at bottom of the XCode.
    ![](../.gitbook/assets/9e954be8-18e7-48d8-b62d-6698b2b226a6/e9821f53.png)
* Select the location from the list.
* For custom GPS coordinates:
    * Create a `*.gpx` file with following contents:
        ```xml
        <?xml version="1.0" encoding="UTF-8" standalone="no"?>

        <gpx xmlns="http://www.topografix.com/GPX/1/1" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.topografix.com/GPX/1/1 http://www.topografix.com/GPX/1/1/gpx.xsd" version="1.1" creator="gpx-poi.com">
          <wpt lat="52.479761" lon="62.185661">
            <ele>367.00000</ele>
            <time>2010-01-01T00:02:16Z</time>
          </wpt>
         </gpx>
        ```

    * Replace `lat` and `lon` values to whatever you like.
    * Click "Location" icon, and select: "Add GPX File to Project..."
    * Select your custom `*.gpx` file.
    * Click "Location" icon again, and select the name of your file from the list.
* When done, select "Product"-\>"Stop".
* Uninstall your dummy app from iOS device.
* Your location should be back to normal, if not - restart your iOS device.
