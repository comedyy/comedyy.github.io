IOS打包问题



 1. 打包需要有 team。 有两种方式可以拿到team。 一个是登入账号，然后从服务器上下载证书。另外一种方式是从别人的p12导入。导入开发证书跟发布证书。

 2. 还需要有个  privisioning profile, 记录着当前的发包， 包名，证书类型，以及所有的设备。

    这些设置统统有一个plist文件来管理。

    adhoc

    ```xml
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
    	<key>method</key>
    	<string>ad-hoc</string>
    	<key>teamID</key>
    	<string>M8VLC75MUF</string>
    	<key>provisioningProfiles</key>
    	<dict>
    		<key>com.wan.wartuneusa</key>
    		<string>wartuneusa_adhoc</string>
    	</dict>
    	<key>compileBitcode</key>
    	<false/>
    	<key>uploadSymbols</key>
    	<false/>
    </dict>
    </plist>
    ```

    appstore

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
    	<key>method</key>
    	<string>app-store</string>
    	<key>teamID</key>
    	<string>M8VLC75MUF</string>
    	<key>provisioningProfiles</key>
    	<dict>
    		<key>com.wan.wartuneusa</key>
    		<string>wartuneusa_dis</string>
    	</dict>
    	<key>compileBitcode</key>
    	<false/>
    	<key>uploadSymbols</key>
    	<false/>
    </dict>
    </plist>
    
    ```

    



 1. 需要在buildsetting中设置当前使用的证书类型。

    

    Unity中配置：

    ```c#
    		proj.SetTeamId(target, "QT4K5KS365");
    
    		proj.SetBuildProperty(target, "CODE_SIGN_IDENTITY[sdk=iphoneos*]", "iPhone Developer");
    		proj.SetBuildProperty(target, "PROVISIONING_PROFILE_SPECIFIER", "newproject6_dev");
    ```
    

打包脚本配置：使用到了plist文件。

```python
xcodePath = os.path.join(workspace, "xcode")
xcodeProjPath = os.path.join(xcodePath, "Unity-iPhone.xcodeproj")
achivePath = os.path.join(xcodePath, "build/x.xcarchive")
ipaExportPath = os.path.join(xcodePath, "build/export")
plistPath = os.path.join("/Users/nt_build_mac/workspace", "export-ipa-7road-adhoc.plist")
ipaPath = os.path.join(ipaExportPath, "newproject6.ipa")

errorCodeAchive = os.system("xcodebuild -project {xcodeProjPath} -scheme Unity-iPhone archive -archivePath {achivePath}".format(xcodeProjPath = xcodeProjPath, achivePath = achivePath))
if errorCodeAchive != 0:
	raise SystemExit("XcodeAchiveError exit Code = {errorCodeAchive}".format(errorCodeAchive = errorCodeAchive))

errorExport = os.system("xcodebuild -exportArchive -archivePath {achivePath} -exportPath {ipaExportPath} -allowProvisioningUpdates -exportOptionsPlist {plistPath}"
	.format(achivePath = achivePath, ipaExportPath = ipaExportPath, plistPath = plistPath))
if errorExport != 0:
	raise SystemExit("XcodeExportError exit Code = {errorExport}".format(errorExport = errorExport))
```


