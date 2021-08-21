需要工具


bundletool-all-1.6.1.jar
bundletool.jar 是google提供生成&测试aab的工具，gradle打包里面也是使用的这个工具。
获取方式github: github.com/google/bund…
详细文档&使用方法: developer.android.com/studio/comm…


aapt2
aapt全称Android Asset Packaging Tool是Android资源打包工具。
获取方式ANDROID SDK: $ANDROID_SDK/build-tools/30.0.3/aapt2
获取方式google maven: dl.google.com/dl/android/…
详细文档&使用方法:developer.android.com/studio/comm…


apktool_2.5.0.jar
反编译安卓apk工具。
获取方式github: github.com/iBotPeaches…


android.jar
android framework,提供了系统的资源和api。
获取方式ANDROID SDK:  $ANDROID_SDK/platforms/android-30/android.jar


apk生成aab
Android Studio打包可选Android App Bundle(aab)，并提供详细教程，本文不再说明。
解压apk
通过apktool去解压apk包
java -jar apktool_2.5.0.jar d test.apk -s -o decode_apk_dir
复制代码
解压apk后 decode_apk_dir 目录结构：
./decode_apk_dir
├── AndroidManifest.xml
├── apktool.yml
├── assets
├── classes2.dex
├── classes.dex
├── lib
├── original
├── res
└── unknown
复制代码
编译资源
编译资源使用aapt2编译生成 *.flat文件集合
aapt2 compile --dir decode_apk_dir\res -o compiled_resources.zip
复制代码
生成compiled_resources.zip文件
为什么要加.zip的后缀，不和谷歌官方文档一样直接生成compiled_resources文件，或者compiled_resources文件夹。此处为了windows能正常的编译打包，linux和mac随意~
关联资源
aapt2 link --proto-format -o base.apk -I android_30.jar \
--min-sdk-version 19 --target-sdk-version 29 \
--version-code 1 --version-name 1.0 \
--manifest decode_apk_dir\AndroidManifest.xml \
-R compiled_resources.zip --auto-add-overlay
复制代码
生成base.apk
解压base.apk
通过unzip解压到base文件夹，目录结构：
./base
├── AndroidManifest.xml
├── res
└── resources.pb
复制代码
拷贝资源
以base文件夹为根目录
创建 base/manifest 将 base/AndroidManifest.xml 剪切过来
拷贝assets , 将 ./temp/decode_apk_dir/assets 拷贝到 ./temp/base/assets
拷贝lib， 将 ./temp/decode_apk_dir/lib 拷贝到 ./temp/base/lib
拷贝unknown， 将 ./temp/decode_apk_dir/unknown 拷贝到 ./temp/base/root
拷贝kotlin， 将 ./temp/decode_apk_dir/kotlin拷贝到 ./temp/base/root/kotlin
拷贝META-INF，将./temp/decode_apk_dir/original/META-INF 拷贝到 ./temp/base/root/META-INF (删除签名信息***.RSA**、.SF、.MF)
创建./base/dex 文件夹，将 ./decode_apk_dir/*.dex（多个dex都要一起拷贝过来）
base/manifest                        ============> base/AndroidManifest.xml
decode_apk_dir/assets                ============> base/assets
decode_apk_dir/lib                   ============> base/lib
decode_apk_dir/unknown               ============> base/root
decode_apk_dir/kotlin                ============> base/root/kotlin
decode_apk_dir/original/META-INF     ============> base/root/META-INF
decode_apk_dir/*.dex                 ============> base/dex/*.dex
复制代码
最终的目录结构
base/
├── assets
├── dex
├── lib
├── manifest
├── res
├── resources.pb
└── root
复制代码
压缩资源
将base文件夹，压缩成base.zip 一定要zip格式
编译aab
打包app bundle需要使用bundletool
java -jar bundletool-all-1.6.1.jar build-bundle \
--modules=base.zip --output=base.aab
复制代码
aab签名
jarsigner -digestalg SHA1 -sigalg SHA1withRSA \
-keystore luojian37.jks \
-storepass ****** \
-keypass ****** \
base.aab \
******
复制代码
注意：您不能使用 apksigner 为 aab 签名。签名aab的时候不需要使用v2签名，使用JDK的普通签名就行。
测试
此时我们已经拿到了一个aab的包，符合Google Play的上架要求，那么我们要确保这个aab的包是否正常呢？作为一个严谨的程序员还是得自己测一下。
上传Google Play
上传Google Play的内部测试，通过添加测试用户从Google Play去下载到手机测试。更加能模拟真实的用户环境。
bundletool安装aab(推荐)
每次都上传到Google Play上面去测试，成本太高了，程序员一般没上传权限，运营也不在就没法测试了。此时我们可以使用bundletool模拟aab的安装。
连接好手机，调好adb，执行bundletool命令进行安装
1.从 aab 生成一组 APK
java -jar bundletool-all-1.6.1.jar build-apks \
--bundle=base.aab \
--output=base.apks \
--ks=luojian37.jks \
--ks-pass=pass:****** \
--ks-key-alias=****** \
--key-pass=pass:******
复制代码
2.将 APK 部署到连接的设备
java -jar bundletool-all-1.6.1.jar install-apks --apks=base.apks
复制代码
还原成apk
竟然apk可以转化成aab，同样aab也可以生成apk，而且更加简单
java -jar bundletool-all-1.6.1.jar build-apks \
--mode=universal \
--bundle=base.aab \
--output=test.apks \
--ks=luojian37.jks \
--ks-pass=pass:****** \
--ks-key-alias=****** \
--key-pass=pass:******
复制代码
此时就可以或得一个test.apks的压缩包，解压这个压缩包就有一个universal.apk，和开始转化的apk几乎一样。
获取工具&源码
github.com/37sy/build_…
结束语
过程中有问题或者需要交流的同学，可以扫描二维码加好友，然后进群进行问题和技术的交流等；
