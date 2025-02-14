## Common process of Transmitting a Payment Link to a Custom MIA NFC Tag
1. **Receiver** (the device that wants to receive the payment link):
   - Emulates a custom NFC tag with the MIA AID.
   - The tag supports write operations.

2. **Transmitter** (the device that wants to send the payment link):
   - Initiates the NFC tag discovery process.
   - Upon detecting the NFC tag, attempts to write the payment link to the MIA NFC tag.

## mia-tag

A module for emulating a custom MIA NFC tag to receive payment links - **Receiver** in the process described above.

## Features
- Supports write operations.
- Uses AID: `6d6466696e657267796d6961706179` (byte representation of the string "mdfinergymiapay").

## Installation
To add this module to your project, include the following dependencies:
- `mia-nfc-common-{version}-release.aar`
- `mia-tag-{version}-release.aar`

## Usage

### Step 1: Implement `MiaActivityIntentProvider` in Your Application Class
Your application's `Application` class must implement the `MiaActivityIntentProvider` interface. 
This interface is used by the `MiaTagService`, which emulates a MIA NFC tag, to start your Activity interested in processing a received MIA payment link.

```kotlin
/**
 * Interface for providing an [android.content.Intent] to launch an [android.app.Activity] when a payment MIA link is received.
 *
 * This interface should be implemented by the [android.app.Application] class of the app to ensure
 * global accessibility and allow proper handling of MIA links in various contexts,
 * including background services.
 */
interface MiaActivityIntentProvider {

    /**
     * Generates an [android.content.Intent] to launch the appropriate [android.app.Activity] for handling the received MIA link.
     *
     * @param receivedUrl The MIA payment link that needs to be processed.
     * @return An [android.content.Intent] configured with the necessary flags and data to start the desired [android.app.Activity].
     *
     * **Note:** The [android.content.Intent] must include `Intent.FLAG_ACTIVITY_NEW_TASK` to ensure
     * it can be started from a background context (e.g., from a Service).
     *
     * Additional flags can be added as needed. For example:
     * - `Intent.FLAG_ACTIVITY_NEW_TASK`
     * - `Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK`
     * - `Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_SINGLE_TOP`
     */
    fun getMiaActivityIntent(receivedUrl: String): Intent
}
```

### Step 2: Enable or Disable `MiaTagService`
You can enable or disable the `MiaTagService` dynamically using the following helper method:

```kotlin
/**
 * Explicitly enables or disables the [md.finergy.mia.nfc.tag.service.MiaTagService] without killing the app.
 *
 * @param isEnabled whether to enable or disable the component
 *
 * @see PackageManager.setComponentEnabledSetting
 * @see PackageManager.COMPONENT_ENABLED_STATE_ENABLED
 * @see PackageManager.COMPONENT_ENABLED_STATE_DISABLED
 */
fun Context.setMiaTagServiceEnabled(isEnabled: Boolean)
```

To check if the service is currently enabled, use:

```kotlin
fun Context.isMiaTagServiceEnabled(): Boolean
```

### Note 1: Setting the Preferred HCE Service
When your `Activity` is in the resumed state, you can set `MiaTagService` as the preferred service:

```kotlin
/**
 * Allows a foreground application to specify [md.finergy.mia.nfc.tag.service.MiaTagService] service as preferred
 * while a specific [android.app.Activity] is in the foreground.
 *
 * The specified [android.app.Activity] must currently be in resumed state. A good paradigm is to call this method
 * in your [android.app.Activity.onResume], and to call [android.app.Activity.unsetPreferredHceService] in your [android.app.Activity.onPause].
 *
 * For more details, see the description of the [android.nfc.cardemulation.CardEmulation.setPreferredService] method.
 *
 * Note: Use of this method requires the
 * [android.content.pm.PackageManager.FEATURE_NFC_HOST_CARD_EMULATION] to be present on the device.
 */
fun Activity.setPreferredMiaTagService(): Result<Boolean>
```

### Note 2: Override the service name visible to the user
Override the string resource:
```xml
<string name="mia_tag_service_name">MIA Tag Service</string>
```
to better associate it with your application. You can use your app's name, for example.
The user will see this name in the NFC tag selection dialog if their device has multiple tags with the same AID.

### Note 3: No Need to Modify `AndroidManifest.xml`
You do not need to manually update your `AndroidManifest.xml` file. The dependency already defines the required permissions and services:

```xml
<uses-permission android:name="android.permission.NFC" />

<application>
    <service
        android:name=".service.MiaTagService"
        android:enabled="false"
        android:exported="true"
        android:permission="android.permission.BIND_NFC_SERVICE">
        <intent-filter>
            <action android:name="android.nfc.cardemulation.action.HOST_APDU_SERVICE" />
            <category android:name="android.intent.category.DEFAULT" />
        </intent-filter>
        <meta-data
            android:name="android.nfc.cardemulation.host_apdu_service"
            android:resource="@xml/mia_tag_service" />
    </service>
</application>
```

## Compatibility
- Android 5.0 (API 21) and above
- Requires NFC hardware support and HCE (Host Card Emulation) feature

To check NFC capabilities, use the helper methods from `mia-nfc-common-{version}-release.aar`:
```kotlin
/** Checks whether the device can communicate using Near-Field Communication (NFC). **/
fun Context.isNfcSupported(): Boolean

/** Checks whether the device supports host-based NFC card emulation (HCE). **/
fun Context.isHceSupported(): Boolean
```
