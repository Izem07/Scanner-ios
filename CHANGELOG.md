# ScoutOps Scan — Changelog

## v1.3.3
- Native iOS battery accuracy: replaced `battery_plus` on iOS with a UIKit `MethodChannel` (`UIDevice.batteryLevel` with proper rounding) to match the iOS status bar exactly
- Station dots (R1/R2/R3, B1/B2/B3) now correctly highlight on scan — fixed column index bug in `filledStationsForCurrentMatch` (was reading `columns[7]` for match number, correct is `columns[2]`)
- Button loading states: Reset shows spinner for 600ms, Sync spins until network returns, Export stays active while share sheet is open
- History and Export buttons highlight while their sheet/share dialog is open, clear on dismiss
- Settings screen accessible on web (was silently broken) — opens as bottom sheet on web, page push on mobile

## v1.3.2
- Rebased to v1.2.3, then bumped to v1.3.2 for release
- Match info card repositioned to sit directly above the action buttons (removed hardcoded `bottom: 115` offset)

## v1.2.2 / v1.2.1
- Battery icon shows lightning bolt when charging with percentage display
- Various workflow and build fixes

## v1.2
- Initial Neon database sync support
- CSV export via share sheet
- Scan history bottom sheet
- Match card with alliance color and station dots
- Animated scan line viewfinder
- Real battery tracking via `battery_plus`
- Haptic feedback on scan
- Settings screen for Neon connection string

## v1.0
- First working build
- Basic QR/barcode scanning with `mobile_scanner`
- Flutter project scaffold
