---
layout: post
title:  "IDA Dynamic Debugging"
tags: IDA Reversing binary
---

on mac

debugserver 추출

```swift
$ hdiutil attach /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/12.1\ \(16B5059d\)/DeveloperDiskImage.dmg
$ cp /Volumes/DeveloperDiskImage/usr/bin/debugserver ./

$ lipo -thin arm64 debugserver  -output debugserver_arm64/
```

debugserver → iphone

on iphone

ldid를 이용한 debugserver 자격 수정 및 /usr/bin 복사 스크립트

```swift
#!/bin/sh
set -e

if [[ -x "/usr/bin/debugserver" ]]; then
  set +e
  /usr/bin/debugserver > /dev/null 2>&1
  if [[ $? -ne 1 ]]; then
    rm -f /usr/bin/debugserver
  fi
  set -e
fi

if [[ ! -x "/usr/bin/debugserver" && -x "/Developer/usr/bin/debugserver" ]]; then
    cp /Developer/usr/bin/debugserver /private/var/tmp/debugserver
    ldid -S/usr/share/entitlements/debugserver.xml /private/var/tmp/debugserver
    mv /private/var/tmp/debugserver /usr/bin
fi
if [[ -x "/usr/bin/debugserver" ]]; then
    exec /usr/bin/debugserver "$@"
else
    echo "Please mount developer disk image"
fi
```