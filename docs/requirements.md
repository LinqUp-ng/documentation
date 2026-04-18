# Library Requirements & Configuration

This document outlines the libraries required for the `linqUp` project, including installation commands and necessary `app.json`, `metro.config.js`, and `babel.config.js` configurations.

---

## ⚙️ Core Configuration Files

Some libraries require manual updates to your configuration files after installation.

### 1. `babel.config.js`

Required for **Reanimated** (used by Zoom Toolkit and Bottom Sheet).

```javascript
module.exports = function (api) {
    api.cache(true);
    return {
        presets: ["babel-preset-expo"],
        plugins: [
            "react-native-reanimated/plugin", // Must be last
        ],
    };
};
```

### 2. `metro.config.js`

Required for **React Native SVG Transformer**.

```javascript
const { getDefaultConfig } = require("expo/metro-config");

module.exports = (() => {
    const config = getDefaultConfig(__dirname);

    const { transformer, resolver } = config;

    config.transformer = {
        ...transformer,
        babelTransformerPath:
            require.resolve("react-native-svg-transformer/expo"),
    };
    config.resolver = {
        ...resolver,
        assetExts: resolver.assetExts.filter((ext) => ext !== "svg"),
        sourceExts: [...resolver.sourceExts, "svg"],
    };

    return config;
})();
```

---

## 🔐 1. Authentication & Security

### Apple Authentication

- **Installation:**
    ```bash
    npx expo install expo-apple-authentication
    ```
- **app.json Config:**
    ```json
    {
        "expo": {
            "plugins": ["expo-apple-authentication"]
        }
    }
    ```

### Google Authentication

- **Installation:**
    ```bash
    npx expo install @react-native-google-signin/google-signin
    ```

### Local Authentication (Biometrics)

- **Installation:**
    ```bash
    npx expo install expo-local-authentication
    ```
- **app.json Config:**
    ```json
    {
        "expo": {
            "plugins": ["expo-local-authentication"]
        }
    }
    ```

### Secure Storage & Keychain

- **Installation:**
    ```bash
    npx expo install expo-secure-store react-native-keychain
    ```

---

## 🎨 2. UI & Animations

### Legend List

Optimized list component for React Native.

- **Installation:**
    ```bash
    npm install @legendapp/list
    ```

### React Native SVG

- **Installation:**
    ```bash
    npx expo install react-native-svg
    ```
- **SVG Transformer Setup (Dev Dep):**
    ```bash
    npm install -D react-native-svg-transformer
    ```
    _(See Metro Config section above for setup)_

### UI Components & Transitions

- **Installation:**
    ```bash
    npx expo install expo-linear-gradient @gorhom/bottom-sheet react-native-shimmer-placeholder react-native-shadow-2
    ```
    _(Note: expo-image is already included by default)_

### Keyboard & Interaction

- **Installation:**
    ```bash
    npx expo install react-native-keyboard-controller enzomanuelmangano/pressto
    ```
    > [!NOTE]
    > `react-native-keyboard-controller` is an **Expo Module**. It is auto-linked and does **not** require an entry in the `plugins` array.

---

## 📱 3. Device Features & Media

### Image Picker

- **Installation:**
    ```bash
    npx expo install expo-image-picker
    ```
- **app.json Config:**
    ```json
    {
        "expo": {
            "plugins": [
                [
                    "expo-image-picker",
                    {
                        "photosPermission": "Allow linqUp to access your photos",
                        "cameraPermission": "Allow linqUp to access your camera"
                    }
                ]
            ]
        }
    }
    ```

### Audio (Expo Audio)

- **Installation:**
    ```bash
    npx expo install expo-audio
    ```
- **app.json Config:**
    ```json
    {
        "expo": {
            "plugins": [
                [
                    "expo-audio",
                    {
                        "microphonePermission": "Allow linqUp to access your microphone."
                    }
                ]
            ]
        }
    }
    ```

### Notifications

- **Installation:**
    ```bash
    npx expo install expo-notifications
    ```
- **app.json Config:**
    ```json
    {
        "expo": {
            "plugins": ["expo-notifications"]
        }
    }
    ```

### Maps

- **Installation:**
    ```bash
    npx expo install react-native-maps
    ```

---

## 🛠️ 4. State Management & Utilities

### Zustand & MMKV

- **Installation:**
    ```bash
    npm install zustand react-native-mmkv
    npx expo install @react-native-async-storage/async-storage
    ```

### Device Info & Constants

- **Installation:**
    ```bash
    npx expo install expo-device expo-constants
    ```

### Misc Utilities

- **Installation:**
    ```bash
    npx expo install @react-native-community/datetimepicker react-native-zoom-toolkit
    ```

### File System & Assets

- **Installation:**
    ```bash
    npx expo install expo-asset expo-file-system expo-image-manipulator
    ```

npm install moment
npm install libphonenumber-js

{/* expo location */}

npx expo install expo-location

{/* react native blurview */}

npm install @danielsaraldi/react-native-blur-view

{/* supabase sdk */}

npx expo install @supabase/supabase-js

{/* react native skia */}

npx expo install @shopify/react-native-skia

{/* portal */}

npm i @gorhom/portal

{/* watermelon db */}

npm install @nozbe/watermelondb

{/* clip board */}

npx expo install expo-clipboard

npm install @zegocloud/zego-uikit-prebuilt-call-rn zego-express-engine-reactnative zego-zim-react-native

npm i react-native-blurhash react-native-image-colors

npx expo install react-native-fast-confetti

npx expo install expo-camera

{/* required for supabase */}

npx expo install react-native-url-polyfill

{/* oauth */}

npx expo install expo-apple-authentication
npm i @react-native-google-signin/google-signin@latest
npm install aes-js

npm i react-native-qrcode-svg
