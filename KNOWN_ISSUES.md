## Starting activities from the background

When a custom MIA NFC tag receives a payment link, `MiaTagService` launches an Activity using 
an Intent obtained from the `MiaActivityIntentProvider::getMiaActivityIntent` method. 
This can happen even from the background. Android allows this because `MiaTagService` is 
a `HostApduService` that is bound by the system (for detailed information, see the [documentation](https://developer.android.com/guide/components/activities/background-starts#exceptions)).

This works well in general, but some phone manufacturers impose custom restrictions 
on launching Activities from the background. For example, on Xiaomi devices, users need to manually 
enable a specific permission in the settings to allow background Activity launches:
- App Info -> Other permissions -> enable "Display pop-up windows while running in the background"
- Settings -> Apps -> Permissions -> Other permissions -> App name -> enable "Display pop-up windows while running in the background"

Therefore, if there are issues with launching an Activity from the background, 
advise users to check for and enable any necessary custom permissions.
