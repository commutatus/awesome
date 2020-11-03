---
title: Fastlane for React Native Android
nav_order: 1
parent: Efficient
grand_parent: Engineering
---

  

***React Native Fastlane deployment for Android***

*system prerequisites*

  

> Using RubyGems -> sudo gem install fastlane -NV

> Alternatively using -> Homebrew brew cask install fastlane

  

## Steps to generate json secret

1. Open the [Android fastlane](https://play.google.com/apps/publish/)

2. Click the **Settings** menu entry, followed by **API access**

[![Android fastlane](/assets/images/android-fastlane-1.png)](/assets/images/android-fastlane-1.png)

  

3. Click the **CREATE SERVICE ACCOUNT** button

[![Android fastlane](/assets/images/android-fastlane-2.png)](/assets/images/android-fastlane-2.png)

4. Follow the **Google Developers Console** link in the dialog, which opens a new tab/window:

1. Click the **CREATE SERVICE ACCOUNT** button at the top of the **Google Developers Console**

[![Android fastlane](/assets/images/android-fastlane-3.png)](/assets/images/android-fastlane-3.png)

2. Provide a `Service account name`

[![Android fastlane](/assets/images/android-fastlane-4.png)](/assets/images/android-fastlane-4.png)

3. Click **Select a role** and choose **Service Accounts > Service Account User**

[![Android fastlane](/assets/images/android-fastlane-5.png)](/assets/images/android-fastlane-5.png)

[![Android fastlane](/assets/images/android-fastlane-6.png)](/assets/images/android-fastlane-6.png)

4. Check the **Furnish a new private key** checkbox

[![Android fastlane](/assets/images/android-fastlane-7.png)](/assets/images/android-fastlane-7.png)

5. Make sure **JSON** is selected as the `Key type`

[![Android fastlane](/assets/images/android-fastlane-8.png)](/assets/images/android-fastlane-8.png)

[![Android fastlane](/assets/images/android-fastlane-9.png)](/assets/images/android-fastlane-9.png)

6. Click **SAVE** to close the dialog

7. Make a note of the file name of the JSON file downloaded to your computer

5. Back on the **Google Play Console**, click **DONE** to close the dialog

6. Click on **Grant Access** for the newly added service account

[![Android fastlane](/assets/images/android-fastlane-10.png)](/assets/images/android-fastlane-10.png)

7. Choose **Release Manager** (or alternatively **Project Lead**) from the `Role` dropdown. (Note that choosing **Release Manager** grants access to the production track and all other tracks. Choosing **Project Lead** grants access to update all tracks _except_ the production track.)

[![Android fastlane](/assets/images/android-fastlane-11.png)](/assets/images/android-fastlane-11.png)

8. Click **ADD USER** to close the dialog

## Setting up _fastlane_

Navigate your terminal to your project's directory and run

[![Android fastlane](/assets/images/android-fastlane-12.png)](/assets/images/android-fastlane-12.png)

  

```

fastlane init

```

  

You'll be asked to confirm that you're ready to begin, and then for a few pieces of information. To get started quickly:

  

1. Provide the package name for your application

  

2. Press enter when asked for the path to your json secret file

3. Answer 'n' when asked if you plan on uploading info to Google Play via fastlane (we can set this up later)

  

That's it! _fastlane_ will automatically generate a configuration for you based on the information provided.

  

You can see the newly created `./fastlane` directory, with the following files:

  

-  `Appfile` which defines configuration information that is global to your app

-  `Fastfile` which defines the "lanes" that drive the behavior of _fastlane_

  
  

```

default_platform(:android)

platform :android do

lane :beta do

path = '../app/build.gradle'

re = /versionCode\s+(\d+)/

s = File.read(path)

versionCode = s[re, 1].to_i

s[re, 1] = (versionCode + 1).to_s

f = File.new(path, 'w')
f.write(s)
f.close

releaseFilePath = File.join(Dir.pwd, "../app/", "YOUR_KEY_STORE_FILE_NAME")

gradle(task: 'clean')

gradle(

task: 'bundle',

build_type: 'Release',

print_command: false,

properties: {
"android.injected.signing.store.file" => releaseFilePath,

"android.injected.signing.store.password" => "STORE_PASSWORD",

"android.injected.signing.key.alias" => "MY_KEY_ASIAS",

"android.injected.signing.key.password" => "SIGNING_PASSWORD",
})
upload_to_play_store(
track: 'internal'
)
end
end
```
  

**Let's Deploy**

```

fastlane beta

```
*React native awesome boilerplate [Link](https://docs.fastlane.tools/getting-started/cross-platform/react-native/)*

*For more details visit [fastlane](https://docs.fastlane.tools/getting-started/cross-platform/react-native/)*