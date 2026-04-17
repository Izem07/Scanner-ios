# Requirements Document

## Introduction

ScoutOpsScanner (scout_ops_scan v5.5.5+1) is a Flutter QR/barcode scanning app currently configured for Android and web. This feature converts the app to a fully iOS-targeted build, removes Android-specific configuration from the Dart/config layer, and establishes a GitHub Actions CI pipeline that produces an unsigned `.ipa` file on a `macos-latest` runner — suitable for sideloading via AltStore or Sideloadly without an Apple Developer account.

## Glossary

- **Runner**: The GitHub Actions virtual machine executing the CI workflow (`macos-latest`).
- **IPA**: iOS App Store Package — a zip archive containing a `Payload/Runner.app` bundle, the distributable format for iOS apps.
- **Podfile**: CocoaPods dependency manifest located at `ios/Podfile`; controls the iOS deployment target.
- **Info.plist**: iOS property list at `ios/Runner/Info.plist`; declares app metadata and privacy usage descriptions required by iOS.
- **flutter_launcher_icons**: Flutter dev-dependency that generates platform-specific app icons from a source image.
- **SystemUiMode**: Flutter API controlling system UI visibility; `edgeToEdge` is cross-platform, `immersiveSticky` is Android-only.
- **Sideloading**: Installing an unsigned IPA onto a physical iOS device using a tool such as AltStore or Sideloadly, valid for 7 days per free Apple ID signing.
- **Workflow_dispatch**: GitHub Actions trigger that allows manual execution of a workflow from the Actions UI.
- **NSCameraUsageDescription**: iOS Info.plist key that provides the user-facing reason the app requests camera access.
- **NSPhotoLibraryUsageDescription**: iOS Info.plist key that provides the user-facing reason the app requests photo library access.
- **CocoaPods**: iOS/macOS dependency manager; `pod install` resolves native dependencies declared in the Podfile.
- **Deployment Target**: Minimum iOS version the app supports; set to iOS 13.0 for this project.

---

## Requirements

### Requirement 1: iOS Platform Scaffolding

**User Story:** As a developer, I want the Flutter project to include a valid `ios/` directory, so that the app can be built and distributed for iOS devices.

#### Acceptance Criteria

1. WHEN `flutter create --platforms=ios . --project-name scout_ops_scan` is executed in the repository root, THE Flutter_Toolchain SHALL generate the `ios/` directory with all required Xcode project files.
2. THE `ios/Runner/Info.plist` SHALL exist after scaffolding and contain the default Flutter iOS configuration keys.
3. THE `ios/Podfile` SHALL exist after scaffolding and reference the project's native dependencies.

---

### Requirement 2: iOS Privacy Permissions

**User Story:** As a developer, I want the app to declare camera and photo library usage descriptions, so that iOS grants the app the permissions required for QR/barcode scanning.

#### Acceptance Criteria

1. THE `ios/Runner/Info.plist` SHALL contain an `NSCameraUsageDescription` key with a non-empty string describing why the app requires camera access.
2. THE `ios/Runner/Info.plist` SHALL contain an `NSPhotoLibraryUsageDescription` key with a non-empty string describing why the app requires photo library access.
3. IF `NSCameraUsageDescription` is absent from `Info.plist`, THEN THE CI_Workflow SHALL add the key before the build step executes.
4. IF `NSCameraUsageDescription` is already present in `Info.plist`, THEN THE CI_Workflow SHALL overwrite it with the canonical description string.

---

### Requirement 3: iOS Deployment Target

**User Story:** As a developer, I want the iOS deployment target set to iOS 13.0, so that the app is compatible with the minimum iOS version required by `mobile_scanner ^7.0.1` and other dependencies.

#### Acceptance Criteria

1. THE `ios/Podfile` SHALL declare `platform :ios, '13.0'` as the deployment target.
2. WHEN the Podfile contains a commented-out or lower-version platform declaration, THE CI_Workflow SHALL replace it with `platform :ios, '13.0'` before `pod install` runs.
3. WHILE `pod install` executes, THE CocoaPods_Resolver SHALL resolve all native pod dependencies against the iOS 13.0 deployment target without version conflicts.

---

### Requirement 4: App Icon — iOS Only

**User Story:** As a developer, I want `flutter_launcher_icons` configured for iOS only, so that the app icon is generated for the iOS target without producing Android or web icon artifacts.

#### Acceptance Criteria

1. THE `pubspec.yaml` `flutter_launcher_icons` section SHALL set `android: false`.
2. THE `pubspec.yaml` `flutter_launcher_icons` section SHALL set `ios: true`.
3. THE `pubspec.yaml` `flutter_launcher_icons` section SHALL reference `assets/logo.png` as the icon source image.
4. WHEN `flutter pub run flutter_launcher_icons` is executed, THE Flutter_Toolchain SHALL generate iOS icon assets in `ios/Runner/Assets.xcassets/AppIcon.appiconset/` without modifying any Android resource directories.

---

### Requirement 5: Cross-Platform System UI Mode

**User Story:** As a developer, I want all `SystemUiMode` calls to use `edgeToEdge` instead of `immersiveSticky`, so that the app compiles and runs correctly on iOS without Android-only API calls.

#### Acceptance Criteria

1. THE `lib/main.dart` SHALL NOT contain any call to `SystemUiMode.immersiveSticky`.
2. THE `lib/homepage.dart` SHALL NOT contain any call to `SystemUiMode.immersiveSticky`.
3. WHEN `SystemChrome.setEnabledSystemUIMode` is called at app startup, THE Flutter_Framework SHALL receive `SystemUiMode.edgeToEdge` as the argument.
4. WHEN `SystemChrome.setEnabledSystemUIMode` is called in `Homepage.initState`, THE Flutter_Framework SHALL receive `SystemUiMode.edgeToEdge` as the argument.

---

### Requirement 6: GitHub Actions CI Workflow

**User Story:** As a developer, I want a GitHub Actions workflow that builds an unsigned IPA on a macOS runner, so that I can download and sideload the app without a macOS machine or Apple Developer account.

#### Acceptance Criteria

1. THE `.github/workflows/ios_build.yml` SHALL define a workflow triggered exclusively by `workflow_dispatch`.
2. THE CI_Workflow SHALL execute on a `macos-latest` runner.
3. WHEN the workflow is triggered, THE CI_Workflow SHALL check out the repository using `actions/checkout@v4`.
4. WHEN the workflow is triggered, THE CI_Workflow SHALL set up Flutter stable channel using `subosito/flutter-action@v2`.
5. WHEN Flutter is set up, THE CI_Workflow SHALL run `flutter pub get` to install Dart dependencies.
6. WHEN `flutter pub get` succeeds, THE CI_Workflow SHALL run `flutter create --platforms=ios . --project-name scout_ops_scan` to generate the `ios/` directory if it does not already exist in the repository.
7. WHEN the `ios/` directory exists, THE CI_Workflow SHALL patch `ios/Runner/Info.plist` to add or overwrite `NSCameraUsageDescription` and `NSPhotoLibraryUsageDescription`.
8. WHEN `Info.plist` is patched, THE CI_Workflow SHALL set the Podfile deployment target to `platform :ios, '13.0'`.
9. WHEN the Podfile is updated, THE CI_Workflow SHALL run `pod install` in the `ios/` directory.
10. WHEN `pod install` succeeds, THE CI_Workflow SHALL run `flutter build ios --release --no-codesign`.
11. WHEN the build succeeds, THE CI_Workflow SHALL package the output into an IPA by creating a `Payload/` directory, copying `Runner.app` into it, and zipping the result as `app.ipa`.
12. WHEN the IPA is packaged, THE CI_Workflow SHALL upload `app.ipa` as a workflow artifact named `ios-ipa` with a retention period of 7 days using `actions/upload-artifact@v4`.

---

### Requirement 7: IPA Artifact Integrity

**User Story:** As a developer, I want the produced IPA to be structurally valid, so that sideloading tools (AltStore, Sideloadly) can install it on a physical iOS device.

#### Acceptance Criteria

1. THE IPA_Packager SHALL produce a zip archive whose root contains exactly one directory named `Payload`.
2. THE `Payload` directory SHALL contain exactly one `.app` bundle named `Runner.app`.
3. THE `Runner.app` bundle SHALL be the output of `flutter build ios --release --no-codesign` located at `build/ios/iphoneos/Runner.app`.
4. IF the `build/ios/iphoneos/Runner.app` path does not exist after the build step, THEN THE CI_Workflow SHALL fail with a non-zero exit code before the packaging step.

---

### Requirement 8: Sideloading Compatibility

**User Story:** As a developer, I want the IPA to be installable via AltStore or Sideloadly using a free Apple ID, so that the app can be tested on a physical iOS device without an Apple Developer account.

#### Acceptance Criteria

1. THE IPA SHALL be unsigned (built with `--no-codesign`) so that sideloading tools can apply their own signing certificate.
2. THE app bundle SHALL target a minimum iOS deployment version of 13.0, compatible with devices supported by AltStore and Sideloadly.
3. WHERE a free Apple ID is used for sideloading, THE Sideloading_Tool SHALL be able to sign and install the IPA with a validity period of 7 days.
