---
layout: post
comments: true
title:  "macOS 升级后，无法打开Xcode"
date:   2020-01-26 12:00:00 +0800
categories: experience
---

升级到Catalina后，在打开Xcode时提示无法打开，看了crash log发现如下信息

```log
Package: PKLeopardPackage 
<id=com.apple.pkg.MobileDeviceDevelopment, version=13.0.0.9000000001.1.1565773556, url=file:///Applications/Xcode.app/Contents/Resources/Packages/MobileDeviceDevelopment.pkg>
Failed to verify with error: Error Domain=PKInstallErrorDomain Code=102 
"The package “MobileDeviceDevelopment.pkg” is untrusted." 
UserInfo={
  NSLocalizedDescription=The package “MobileDeviceDevelopment.pkg” is untrusted., NSURL=MobileDeviceDevelopment.pkg -- file:///Applications/Xcode.app/Contents/Resources/Packages/, PKInstallPackageIdentifier=com.apple.pkg.
  MobileDeviceDevelopment, NSUnderlyingError=0x7fa31feafac0 
    {
      Error Domain=NSOSStatusErrorDomain Code=-2147409654 "CSSMERR_TP_CERT_EXPIRED" UserInfo={
        SecTrustResult=5, PKTrustLevel=PKTrustLevelExpiredCertificate, NSLocalizedFailureReason=CSSMERR_TP_CERT_EXPIRED
      }
    }
  }
```

解决方法是把系统时间修改为2019/10/01，之后重新打开即可

参考:
<a href="https://stackoverflow.com/questions/58550284/mobiledevice-pkg-untrusted-cannot-open-xcode-after-os-x-update" target="_blank">https://stackoverflow.com/questions/58550284/mobiledevice-pkg-untrusted-cannot-open-xcode-after-os-x-update</a>
