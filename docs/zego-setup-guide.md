# ZegoCloud React Native Setup Guide

To get ZegoCloud fully operational in your application, follow these precise steps.

## 1. Obtain Your API Keys (AppID and AppSign)

Before writing code, you must register your app on the ZegoCloud dashboard to get your credentials.

1. Go to the [ZegoCloud Console](https://console.zegocloud.com/) and register or log in.
2. Click **Create your project** and choose the **Voice & Video Call** template.
3. On the next screen, enter your project name (e.g., `LinqUp`) and choose **React Native** as the platform.
4. Once the project is created, navigate to the **Project Details** section.
5. Note down the following two critical keys needed for your app:
    - **`AppID`**: (A unique number like `123456789`)
    - **`AppSign`**: (A long string of characters)

> [!IMPORTANT]
> Never hardcode your `AppID` and `AppSign` directly into your GitHub repository for production. Store them securely in an `.env` file and read them via `process.env`.

---

## 2. Configure Native Permissions (For Expo)

Since ZegoCloud uses the camera and microphone natively, you must explicitly request permissions. Because you are using Expo (`app/(app)`), you need to configure your `app.json` to include the required native permissions during your EAS build so it generates the `Info.plist` and `AndroidManifest.xml` correctly.

### Add Plugins to `app.json`

Open your `app.json` file and add the `expo-build-properties` and ZegoCloud plugins into the `"plugins"` array:

```json
{
    "expo": {
        "plugins": [
            [
                "@zegocloud/zego-uikit-prebuilt-call-rn",
                {
                    "ios": {
                        "cameraPermission": "LinqUp needs access to your camera for video calls.",
                        "microphonePermission": "LinqUp needs access to your microphone for audio calls."
                    },
                    "android": {
                        "cameraPermission": "LinqUp needs access to your camera for video calls.",
                        "microphonePermission": "LinqUp needs access to your microphone for audio calls."
                    }
                }
            ],
            [
                "expo-build-properties",
                {
                    "ios": {
                        "useFrameworks": "static"
                    }
                }
            ]
        ]
    }
}
```

> [!NOTE]
> If you don't already have `expo-build-properties` installed, you must install it by running: `npx expo install expo-build-properties`.

---

## 3. Basic Test Screen

Once you have your keys and permissions configured, you can test it anywhere in your Expo Router app.

Here is the exact code snippet to trigger a test call:

```tsx
import React from "react";
import { View, StyleSheet } from "react-native";
// You must import the config layout style you want (e.g., ONE_ON_ONE, GROUP_CALL)
import {
    ZegoUIKitPrebuiltCall,
    ONE_ON_ONE_VIDEO_CALL_CONFIG,
} from "@zegocloud/zego-uikit-prebuilt-call-rn";

const CallScreen = () => {
    return (
        <View style={styles.container}>
            <ZegoUIKitPrebuiltCall
                appID={Number(process.env.EXPO_PUBLIC_ZEGO_APP_ID)} // Replace with your numeric AppID
                appSign={process.env.EXPO_PUBLIC_ZEGO_APP_SIGN} // Replace with your string AppSign
                userID={"user_123456"} // This should be unique for the current logged-in user
                userName={"Ovie User"} // Display Name for the UI
                callID={"test_call_id_1"} // Users joining the exact same callID will enter the same call!
                config={{
                    ...ONE_ON_ONE_VIDEO_CALL_CONFIG,
                    onCallEnd: (callID, reason, duration) => {
                        console.log("Call Ended", reason);
                        // Navigate back here, e.g., router.back()
                    },
                }}
            />
        </View>
    );
};

const styles = StyleSheet.create({
    container: { flex: 1 },
});

export default CallScreen;
```

---

## 4. Generate a New Development Build

Because you have added native modules (`zego-express-engine-reactnative`), the standard Expo Go app will **no longer work** for this testing call screen.
You must generate a new custom development client via EAS so that the native WebRTC code compiles into your app:

```bash
eas build --profile development --platform all
```
