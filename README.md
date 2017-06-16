# Fastfile

Configuaration file for Fastlane

### Configure Custom Setting

1. create .env.default file
2. Set `TEAM_ID`, `APP_IDENTIFIER`, `APPLE_ID`, `SLACK_URL`, `GIT_URL`, `MAIN_PROJECT`, `SCHEME`.

### Option

* `build` - to set Info.plist build number
* `version` - to set Info.plist version number 
* `appversion` - to set app version that should be edited or created in itunesconnect

### Lanes

* `adhoc` - to make Ad-Hoc ipa file
* `metadata` - to deliver metadata to Itunesconnect
* `submit_review` - to submit application for review
* `screenshot` - to take screenshots automatically
* `beta` - to upload application for testflight
* `release` - to upload application for release

### Usage Example

```shell
fastlane beta
fastlane addhoc version:1.1.5 build:2
fastlane metadata appversion:2.1.1
fastlane release build:5
```

Commands above are not related with each other.


