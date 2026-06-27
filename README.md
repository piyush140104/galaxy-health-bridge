# Galaxy Health Bridge

Your Galaxy Watch finally talks to your iPhone.

<p align="center">
  <img src="docs/images/iphone.png" alt="iPhone Today tab" width="280" />
  &nbsp;&nbsp;&nbsp;
  <img src="docs/images/watch.png" alt="Galaxy Watch HealthBridge screen" width="220" />
</p>


If you wear a Samsung Galaxy Watch but use an iPhone, you already know the pain. Samsung Health does not run on iOS, and Apple Health only accepts data from apps installed on your phone. So your steps, heart rate, calories and distance just sit on the watch with no way out.

Galaxy Health Bridge fixes that. It is two small open source apps that work together over Bluetooth, with no cloud, no account and no subscription.

* A **Wear OS app** that runs on your Galaxy Watch, reads your health metrics straight from the watch sensors, and broadcasts them over Bluetooth LE.
* An **iOS app** that connects to the watch over Bluetooth, pulls the data, and writes it into Apple Health on your iPhone.

Your watch talks to your iPhone directly. Nothing leaves your two devices.

## What syncs today

* Steps
* Active calories
* Distance walked or run
* Heart rate (live, updates roughly every second while the iPhone is connected)
* Floors climbed

Sleep stages, SpO2 and workouts are on the roadmap. See `docs/ROADMAP.md`.

## What you need

* A Samsung Galaxy Watch 4 or newer (anything running Wear OS 3 or higher).
* An iPhone running iOS 16 or higher.
* A Mac to install the iPhone app once. You can use any Apple ID, even a free one.
* A computer with `adb` installed to load the watch app once.

Verified end-to-end on an iPhone 14 paired with a Galaxy Watch 4. Other models that meet the requirements above should work but have not been tested by the maintainers — please open an issue if you hit something on a different combination.

## How it works

```
 Galaxy Watch                       iPhone
 +---------------------+            +----------------------+
 | Wear Health         |            | Galaxy Health Bridge |
 | Services            |            | (this iOS app)       |
 |   |                 |            |   |                  |
 |   v                 |            |   v                  |
 | Local Room DB       |            | HealthKit            |
 |   |                 |            |   ^                  |
 |   v                 |            |   |                  |
 | BLE GATT server  ---+--BLE LE----+--> CoreBluetooth     |
 +---------------------+            +----------------------+
```

Every few seconds the watch reads new data from the Wear OS Health Services API, plus a live heart rate stream from the MeasureClient. It stores the latest daily totals in a tiny local database and exposes them on a custom Bluetooth service. When the iPhone connects, it asks the watch for everything newer than the last cursor it knows about, receives the data as small JSON frames, and writes each sample into Apple Health with a tag so you can see it came from the watch.

The wire protocol and the rest of the moving parts are documented in `docs/ARCHITECTURE.md`.

## Install

Pick your platform. You will install both apps.

* **iPhone:** see `docs/INSTALL_IOS.md`. You will need a Mac one time, a free Apple ID, and around 30 minutes.
* **Galaxy Watch:** see `docs/INSTALL_WEAROS.md`. You will need any computer with `adb`, and around 5 minutes.

Once both are installed, follow `docs/TESTING.md` to verify the round trip.

## Honest tradeoffs

A few things you should know before you start.

* **The iPhone app must be reinstalled every 7 days** if you use the free Apple ID route. You can either plug the iPhone back into your Mac and click Run in Xcode, or install AltStore which refreshes the signature automatically. Paying Apple the 99 USD per year developer fee removes this limitation.
* **HealthKit writes happen on the iPhone, not on the watch.** The watch is the Bluetooth broadcaster. The iPhone is the listener. So your iPhone needs to be unlocked with the app open for the sync to happen. Background sync is on the roadmap.
* **Your data never leaves the watch and iPhone.** There is no backend, no account, no analytics. If you uninstall the apps, the data that already made it into Apple Health stays there as normal health records.

## Build from source

You should not blindly trust binaries that touch your health data. That is the whole point of open source. You can read every line and build it yourself.

```bash
git clone https://github.com/<your-fork>/galaxy-health-bridge
cd galaxy-health-bridge
```

**Watch app**

```bash
cd apps/wearos
./gradlew :app:assembleDebug
```

The APK lands in `app/build/outputs/apk/debug/`.

**iPhone app**

```bash
cd apps/ios
brew install xcodegen
xcodegen
open GalaxyHealthBridge.xcodeproj
```

Then build and run from Xcode. See the install guides for the device side steps.

## Running the tests

Unit tests cover the BLE protocol parser, watch-to-iPhone timestamp rebasing and the canonical-sample to HealthKit mapping. Please keep them green on any pull request.

```bash
# Watch
cd apps/wearos && ./gradlew :app:testDebugUnitTest

# iPhone (regenerate the project first if you do not have it yet)
cd apps/ios && xcodegen generate && xcodebuild test \
  -project GalaxyHealthBridge.xcodeproj \
  -scheme GalaxyHealthBridge \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro,OS=18.6' \
  CODE_SIGNING_ALLOWED=NO
```

## Contributing

Issues and pull requests are welcome. Please read `CONTRIBUTING.md` first. There is a code of conduct in `CODE_OF_CONDUCT.md`. If you find a security issue, follow `SECURITY.md` instead of opening a public ticket.

This project is intentionally small. We are not taking outside funding and we are not running a cloud service. The plan is to stay a tiny, focused pair of apps that solve one problem well.

## License

MIT. See `LICENSE`.

Apple, HealthKit, iPhone and iOS are trademarks of Apple Inc. Samsung and Galaxy Watch are trademarks of Samsung Electronics. This project is not affiliated with or endorsed by Apple or Samsung.
