# TapSDK

## 前提条件

* 安装Unity **Unity 2018.3**或更高版本

* iOS **10**或更高版本

* Android 目标为**API21**或更高版本

## 1.添加TapSDK

* 使用Unity Pacakge Manager

```json
//在YourProjectPath/Packages/manifest.json中添加以下代码
"dependencies":{
        "com.tds.sdk":"https://github.com/xindong/TAPSDK_UPM.git#{version_name}"
    }
```

* 本地导入

在工程目录下（与Assets目录同级）创建TapSDK文件夹，导入TapSDK工程。
```json
//在YourProjectPath/Packages/manifest.json中添加以下代码
"dependencies":{
        "com.tds.sdk":"file:..//TapSDK"
    }
```

## 2.配置TapSDK

### 2.1 Android 配置

* 编辑Assets/Plugins/Android/AndroidManifest.xml文件,在Application Tag下添加以下代码。
```xml
    <activity
        android:name="com.taptap.sdk.TapTapActivity"
        android:configChanges="keyboard|keyboardHidden|screenLayout|screenSize|orientation"
        android:exported="false"
        android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen" />
```

### 2.2 iOS 配置

在 **Assets/Plugins/iOS/Resource** 目录下创建 **TDS-Info.plist** 文件,复制以下代码并且替换其中的 **client_Id** 以及申请权限时的文案。

* 注意事项：文件路径和文件名请确认正确且大小写敏感，如果错误会导致iOS编译失败。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>taptap</key>
    <dict>
        <key>client_id</key>
        <string>client-id</string>
    </dict>
    <key>NSPhotoLibraryUsageDescription</key>
    <string>App需要您的同意,才能访问相册</string>
    <key>NSCameraUsageDescription</key>
    <string>App需要您的同意,才能访问相机</string>
    <key>NSMicrophoneUsageDescription</key>
    <string>App需要您的同意,才能访问麦克风</string>
    <key>NSUserTrackingUsageDescription</key>
    <string>App需要追踪你的信息</string>
</dict>
</plist>
```

### 2.3 编译流程

#### 2.3.1 IOS 编译流程 

编译脚本( **Plugins/Script/Editor/TDSIOSBuildPostProcessor.cs** ) 采用默认标签 **[PostProcessBuild]** 来自动执行。如有其他需要，则使用 **[PostProcessBuildAttribute(order)]** 来决定脚本执行顺序。

```c#
    [PostProcessBuild]
    public static void OnPostprocessBuild(BuildTarget BuildTarget, string path)
    {
        ...
    }
```

##### 1.添加所需要的framework以及 Build Setting 

TapSDK 会给iOS工程自动添加以下配置

```c#
    // 编译配置
    proj.AddBuildProperty(target, "OTHER_LDFLAGS", "-ObjC");
    proj.AddBuildProperty(unityFrameworkTarget, "OTHER_LDFLAGS", "-ObjC");
    // Swift编译选项
    proj.SetBuildProperty(target, "ENABLE_BITCODE", "NO"); //bitcode  NO
    proj.SetBuildProperty(target,"ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES","YES");
    proj.SetBuildProperty(target, "SWIFT_VERSION", "5.0");
    proj.SetBuildProperty(target, "CLANG_ENABLE_MODULES", "YES");
    proj.SetBuildProperty(unityFrameworkTarget, "ENABLE_BITCODE", "NO"); //bitcode  NO
    proj.SetBuildProperty(unityFrameworkTarget,"ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES","NO");
    proj.SetBuildProperty(unityFrameworkTarget, "SWIFT_VERSION", "5.0");
    proj.SetBuildProperty(unityFrameworkTarget, "CLANG_ENABLE_MODULES", "YES");
    // add extra framework(s)
    // 参数: 目标targetGUID, framework,是否required:fasle->required,true->optional
    proj.AddFrameworkToProject(unityFrameworkTarget, "CoreTelephony.framework", false);
    proj.AddFrameworkToProject(unityFrameworkTarget, "QuartzCore.framework", false);
    proj.AddFrameworkToProject(unityFrameworkTarget, "Security.framework", false);
    proj.AddFrameworkToProject(unityFrameworkTarget, "WebKit.framework", false);
    proj.AddFrameworkToProject(unityFrameworkTarget, "Photos.framework", false);
    proj.AddFrameworkToProject(unityFrameworkTarget, "AssetsLibrary.framework", false);
    proj.AddFrameworkToProject(unityFrameworkTarget, "AVKit.framework", false);
    proj.AddFrameworkToProject(unityFrameworkTarget, "AuthenticationServices.framework", true);
    proj.AddFrameworkToProject(unityFrameworkTarget, "LocalAuthentication.framework", false);
    proj.AddFrameworkToProject(unityFrameworkTarget, "SystemConfiguration.framework", false);
    proj.AddFrameworkToProject(unityFrameworkTarget, "Accelerate.framework", false);
    proj.AddFrameworkToProject(unityFrameworkTarget, "SafariServices.framework", false);
    proj.AddFrameworkToProject(unityFrameworkTarget, "AVFoundation.framework", false);
    proj.AddFrameworkToProject(unityFrameworkTarget, "MobileCoreServices.framework", false);
    proj.AddFrameworkToProject(unityFrameworkTarget, "AppTrackingTransparency.framework", true);
    proj.AddFrameworkToProject(unityFrameworkTarget, "AdSupport.framework", false);
    proj.AddFrameworkToProject(unityFrameworkTarget, "iAd.framework", true);
    proj.AddFrameworkToProject(unityFrameworkTarget, "AdServices.framework", true);
```

##### 2. 资源文件依赖

* 使用upm方式导入TapSDK,会将 **Library/PacakgeCache/com.tds.sdk@{CommitHashCode}/Plugins/iOS/Resource**
目录中Copy资源文件。

* 使用本地方式导入TapSDK,则会将 **YourProject/TapSDK/Plugins/iOS/Resource** 目录中Copy资源文件。

```c#
    //从UPM缓存目录中拷贝
    string remotePackagePath = TDSFileHelper.FilterFile(parentFolder + "/Library/PackageCache/","com.tds.sdk@");
    //从本地目录中拷贝
    string localPackagePath = TDSFileHelper.FilterFile(parentFolder,"TapSDK");
```

iOS资源文件Copy到XCode工程目录下的 **TDSResource** 文件夹中，再添加依赖到指定Target。

##### 3. Plist文件以及UnityAppController.mm文件配置

* 根据上文iOS配置，用户需要在Assets/Plugins/iOS/Resource 文件夹中创建TDS-Info.plist文件。TapSDK会自动配置 **TapTap** 所需要的 **URL Types** 以及 **UnityAppController.mm** 文件中添加相应代码。

```c#
    // 配置TDS-Info.plist 文件
    SetPlist(path,resourcePath + "/TDS-Info.plist");
    ...
    // 修改 UnityAppController 文件
    string unityAppControllerPath = pathToBuildProject + "/Classes/UnityAppController.mm";
    TDSEditor.TDSScriptStreamWriterHelper UnityAppController = new TDSEditor.TDSScriptStreamWriterHelper(unityAppControllerPath);
    UnityAppController.WriteBelow(@"#import <OpenGLES/ES2/glext.h>", @"#import <TapSDK/TapLoginHelper.h>");
    UnityAppController.WriteBelow(@"id sourceApplication = options[UIApplicationOpenURLOptionsSourceApplicationKey], annotation = options[UIApplicationOpenURLOptionsAnnotationKey];",@"if(url){[TapLoginHelper handleTapTapOpenURL:url];}");
```



