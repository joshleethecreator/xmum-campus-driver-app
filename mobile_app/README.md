# Campus Driver Mobile App

Flutter client for the XMUM Campus Driver project.

---

## Requirements

- **Flutter SDK** (stable channel, 3.x or later)  
- **Dart SDK** (bundled with Flutter)  
- **Android Studio** or **Android command-line tools** with:
  - Android SDK
  - At least one Android Virtual Device (AVD)  
- **Java** (JDK 11+ recommended)  
- For iOS (optional, macOS only):
  - Xcode
  - CocoaPods

Verify your Flutter setup:

```bash
flutter doctor
```

---

## Installation

Clone the monorepo and fetch dependencies:

```bash
git clone <your-repo-url>
cd xmum-campus-driver-app/mobile_app
flutter pub get
```

---

## Running the App (CLI)

### 1. Start an Android emulator from the command line

List available AVDs:

```bash
~/Android/Sdk/emulator/emulator -list-avds
```

Start one (replace `<name>` with what you saw):

```bash
~/Android/Sdk/emulator/emulator -avd <name>
```

Wait until the emulator finishes booting (Android home screen is visible).

### 2. Run the Flutter app

From the `mobile_app` directory:

```bash
cd xmum-campus-driver-app/mobile_app
flutter run
```

If there are multiple devices, Flutter will ask you to choose one.  
You can also run directly on a specific emulator:

```bash
flutter devices          # shows device IDs
flutter run -d <device_id>
```

---

## Running the App (Android Studio)

1. Open **Android Studio**.  
2. Click **Open** and select the `mobile_app` folder in this repo.  
3. Open the **Device Manager** (phone icon in the toolbar).  
4. If you don’t have a device yet:
   - Click **Create device…**
   - Choose a device model (e.g. Pixel 4)
   - Download a system image if needed and finish the wizard.  
5. In Device Manager, click the **▶ (Play)** button next to your AVD to start the emulator.  
6. Once the emulator is booted, click the green **Run** button in Android Studio (or use **Run > Run 'main.dart'**).

Android Studio will build and deploy the app to the selected emulator.
