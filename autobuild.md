## 自动打包
1. 使用unityapi 生成打包代码
2. 使用xcodebuild命令生成achive包
3. 使用xcodebuild命令生成ipa包。
4. 对应分发。
    4.1 如果是appstore包，使用`xcrun altool`命令上传appstore。
        这个命令需要一个key，需要appstore的账号管理者才能去appstore后台申请的。

    4.2 如果是development包的话，放到一个http服务器上，然后手机通过点击网页来下载。
        这个ipa放在http服务器上就可以，plist文件需要放在https服务器上，自签名的证书就可以。
        ```
        <html>
        <head>
                <meta http-equiv="content-type" content="text/html; charset=UTF-8" />
                <META HTTP-EQUIV="Pragma" CONTENT="no-cache">
            <META HTTP-EQUIV="Cache-Control" CONTENT="no-cache">
            <META HTTP-EQUIV="Expires" CONTENT="0">
        </head>
            <body>
                <center>
                <a href="itms-services://?action=download-manifest&url=https://10.191.38.169/test180.plist"><font style='color:#ff0000;font-size:40px'>点此安装180测试包</h1></a>
                </center>

            </body>
        </html>
        ```

        plist文件：
        ```
        <plist version="1.0">
        <dict>
            <key>items</key>
            <array>
                <dict>
                    <key>assets</key>
                    <array>
                        <dict>
                            <key>kind</key>
                            <string>software-package</string>
                            <key>url</key>
                            <string>https://10.191.38.169/test180.ipa</string>
                        </dict>
                        <dict>
                            <key>kind</key>
                            <string>display-image</string>
                            <key>needs-shine</key>
                            <true/>
                            <key>url</key>
                            <string>https://10.191.38.169/57.png</string>
                        </dict>
                        <dict>
                            <key>kind</key>
                            <string>full-size-image</string>
                            <key>needs-shine</key>
                            <true/>
                            <key>url</key>
                            <string>https://10.191.38.169/512.png</string>
                        </dict>
                    </array>
                    <key>metadata</key>
                    <dict>
                        <key>bundle-identifier</key>
                                    <string>com.open.tongxue</string>
                        <key>bundle-version</key>
                        <string>2.4.0</string>
                        <key>kind</key>
                        <string>software</string>
                        <key>title</key>
                        <string>同学+</string>
                    </dict>
                </dict>
            </array>
        </dict>
        </plist>
        ```
