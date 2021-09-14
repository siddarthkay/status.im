---
id: index
title: Build Status Yourself
description: Build Status and participate in a Better Web
---

# Build Status Yourself and Participate in a Better Web

### 0. Prerequisites

- [Git](https://git-scm.com/)
- [cURL](https://curl.se/)
- [GNU Make](https://www.gnu.org/software/make/)

</br>

On Debian/Ubuntu simply use:
```sh
sudo apt install -y git curl make
```

### 1. Clone the repository

```sh
git clone https://github.com/status-im/status-react
cd status-react
```

### 2. Install Dependencies

We use [Nix](https://nixos.org/nix) package manager to create custom shells with all the necessary tools with minimal impact to the user's system. This setup has only been tested following systems:

* Linux - Ubuntu 20.04, Manjaro, [Arch](https://wiki.archlinux.org/index.php/Nix)
* macOS - With XCode 11.1

</br>
If it doesn't work for you on another Linux distribution, please install all dependencies manually based on the list below.

Most `Makefile` targets call [a script](https://github.com/status-im/status-react/blob/develop/nix/scripts/shell.sh) which will implicitly install all the necessary tools and dependencies. To do this it auto-accepts the Android SDK license agreements.

The [pre-defined Nix shells](https://github.com/status-im/status-react/blob/develop/nix/shells.nix) install the following tools:

* Go, OpenJDK 8, Python 2.7
* Clojure and shadow-cljs
* Node.js & Yarn
* React Native CLI and Watchman
* Android SDK & NDK
* Gradle & Maven
* Fastlane & Cocoapods
* CMake & extra-cmake-modules
* Make, Git, cURL, wget, unZip

</br>

__WARNING:__ Downloading all of these can take more than an hour depending on your machine and internet speed.

__WARNING:__ On macOS, the build environment is set up to rely on XCode 11.5. To allow an older version edit `version` in [nix/overlay.nix](https://github.com/status-im/status-react/blob/develop/nix/overlay.nix) file (`xcodeWrapper`).

For more information about our Nix setup [read our README](https://github.com/status-im/status-react/blob/develop/nix/README.md) or [watch our Nix presentations](https://github.com/status-im/status-react/blob/develop/nix/README.md#resources).

### 3. Running development processes

To build Status need to run two processes: Clojure compiler and React Native packager.

Keep both processes running when you develop Status. Restarting them might be requires when updating dependencies or changing Nix configuration.

#### A. Clojure compiler

In the first terminal window, just run:

```sh
make run-clojure
```

By doing this you will start the compilation of ClojureScript sources. You should wait until it shows you `Build completed` before running the React Native packager.

You can also start a debugger server using:
```sh
make run-re-frisk
```

For additional information check the following: [clj-rn](https://github.com/status-im/clj-rn) & [re-frisk](https://github.com/flexsurfer/re-frisk)

#### B. React Native packager

Do this in the second terminal window:

```sh
make run-metro
```

Which starts Metro bundler and watches JavaScript code changes.

### 4. Build and run the application itself

#### iOS (macOS only)

```sh
make run-ios
```

If you wish to specify the simulator, just run `make run-ios SIMULATOR="iPhone 7"`.
You can check your available devices by running `xcrun simctl list devices` from the console.

You can also start XCode and run the application there. Execute `open ios/StatusIm.xcworkspace`, select a device/simulator and click **Run**.

*Note:* Of course, you need to install XCode first in both cases. Just go to Mac AppStore to do this.

#### Android

```sh
make run-android
```

- *Optional:* If you want to use AVD (Android Virtual Device, emulator), please, check [this documentation](https://developer.android.com/studio/run/emulator);
- *Optional:* If you don't like AVD, you can also use [Genymotion](https://genymotion.com);

</br>
Check the following docs if you still have problems:

- [macOS](https://gist.github.com/patrickhammond/4ddbe49a67e5eb1b9c03);
- [Ubuntu Linux](https://gist.github.com/zhy0/66d4c5eb3bcfca54be2a0018c3058931);
- [Arch Linux](https://wiki.archlinux.org/index.php/android) (can also be useful for other Linux distributions).

# Advanced Build Options

### Running commands in Nix shell

If you want to run any of the commands by hand - like Gradle or Fastlane - you can open a Nix shell using either of:
```sh
make shell TARGET=android
```
```sh
make shell TARGET=ios
```
You can read more about our Nix shells [in the docs](https://github.com/status-im/status-react/blob/develop/nix/DETAILS.md#shells).

### Building and using forks of status-go

If you need to use a branch of a `status-go` fork as a dependency of `status-react`, you specify it using an update script:

```sh
scripts/update-status-go.sh <rev>
```

Where `rev` is a branch name, tag, or commit SHA1 you want to build. The script will save the indicated commit hash along with other information in the `status-go-version.json` file.

If you are using a GitHub fork of `status-go` repo, export the `STATUS_GO_OWNER` environment variable when running the script.

### Building local `status-go` repository

If instead you need to use a locally checked-out `status-go` repository as a dependency of `status-react`, you can achieve that by defining the `STATUS_GO_SRC_OVERRIDE`
environment variable:

```sh
export STATUS_GO_SRC_OVERRIDE=$GOPATH/src/github.com/status-im/status-go
# Any command that you run from now on will use the specified status-go location.
make release-android
```

or for a one-off build:

```sh
make release-android STATUS_GO_SRC_OVERRIDE=$GOPATH/src/github.com/status-im/status-go
```

# Troubleshooting

### Inspecting app DB inside Clojure REPL

E.g. if you want to check existing accounts in the device, run this function in the REPL:

```clojure
(get-in @re-frame.db.app-db [:accounts/accounts])
```

### Inspecting current app state in re-frisk web UI

Assuming re-frisk is running in port 4567, you can just navigate to `http://localhost:4567/` in a web browser to monitor app state and events.

### Updating dependencies

* `make nix-update-pods` - iOS CocoaPods dependencies (updates `ios/Podfile` and `ios/Podfile.loc`)
* `make nix-update-gradle` - Android Gradle/Maven dependencies (updates `nix/deps/gradle/deps.json`)
* `make nix-update-clojure` - Clojure Maven dependencies (updates `nix/deps/clojure/deps.json`)
* `make nix-update-gems` - Fastlane Ruby dependencies (updates `fastlane/Gemfile.lock` and `fastlane/gemset.nix`)

### I have issues compiling on Xcode 10

Some developers are experiencing errors compiling for iOS on Xcode 10 on macOS Mojave:

```log
error: Build input file cannot be found:

'status-react/node_modules/react-native/third-party/double-conversion-1.1.6/src/cached-powers.cc'
```

To fix similar errors run the following commands:

```sh
{ cd ios && pod update }
{ cd node_modules/react-native/scripts && ./ios-install-third-party.sh }
{ cd node_modules/react-native/third-party/glog-0.3.4/ && ../../scripts/ios-configure-glog.sh }
```

Now you should be able to compile again. The issue reference is [here](https://github.com/facebook/react-native/issues/21168#issuecomment-422431294).
