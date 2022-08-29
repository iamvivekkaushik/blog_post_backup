## Publishing Flutter apps to Play Store using Github Actions and Fastlane

Recently I was trying to figure out how I can publish flutter apps to the Play Store using Github Actions and publish them I did. I started reading some documentation available at [flutter's official site](https://docs.flutter.dev/deployment/cd) and followed the instructions to set up Fastlane locally on my machine.  First, we will get the Fastlane working on your local machine.

#### Installing Fastlane

First, we will install the Fastlane tool on our local machine. Use homebrew to install it on a mac

```
brew install fastlane
```

on a Linux or Windows machine, we'll use ruby's gem package manager to install Fastlane.

```
gem install fastlane
```

#### Setting up app signing

Go to `android/app/build.gradle` file and define add the following lines:

```
...
// Add these lines
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

// ^ Above this
apply plugin: 'com.android.application'
apply plugin: 'com.google.gms.google-services'

...

 defaultConfig {
    ...

    // Add this as well
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }

    buildTypes {
        release {
            // TODO: Add your own signing config for the release build.
            // Signing with the debug keys for now, so `flutter run --release` works.
            signingConfig signingConfigs.release
        }

    }

...
```

You also need to create a new file under the android folder called `key.properties`, and add the following configurations:

```
storePassword=store-password
keyPassword=key-password
keyAlias=key-alias
storeFile=path-to-jks
```

make sure to change these with your Keystore configuration. Run the `flutter build appbundle` command to build a signed app bundle.

#### Creating the service account

To upload the app bundle on the play console we need a service account that will have the required permissions to perform actions on our behalf. Before doing this make sure you have already registered an app on the play console with the correct package name.

- Head over to [Play Console](play.google.com/console).
- On the left sidebar under **Setup**, you'll see **API Access**.
- Scroll Down until you see the **Service Accounts** section.
- Click on **Learn how to create service accounts**.
- On the popup that shows up click on the link that says **Google Cloud Platform**
- Once you're in the google cloud console, click on **Create service account**
- This will ask you a couple of questions, just fill those in and create a new service account.
- Once a new service account is created click on the **three dots** for the context menu and then **Manage Keys**
- Now you need to create a JSON key for this service account.
- Once all that is done return back to the play console and click on the **Done** button in the Dialog Box.
- You should see the service account listed on the play console. Click on the **View Play Console Permissions**.
- Under app permission, you need to give admin access to this account for the app you want it to manage.

#### Setting up Fastlane for the project

 Now that app signing is set up you need to configure Fastlane for the android project. Make sure you are in the android folder and run the following command:

```
fastlane init
```

Fastlane will ask you a couple of questions, like what's the package name for your app and path to the json secret (this is the file that we downloaded from the last step).

This will create a new folder called `fastlane`, which will contain two files `Appfile` and `Fastfile`. Make sure the package name and path to JSON secret are correct in the Appfile. 

Open the `Fastfile` and replace it with the following lines:

```
# This file contains the fastlane.tools configuration
# You can find the documentation at https://docs.fastlane.tools
#
# For a list of all available actions, check out
#
#     https://docs.fastlane.tools/actions
#
# For a list of all available plugins, check out
#
#     https://docs.fastlane.tools/plugins/available-plugins
#

# Uncomment the line if you want fastlane to automatically update itself
# update_fastlane

default_platform(:android)

platform :android do
  desc "Submit a new Google Play [Beta Testing]"
  lane :beta do
    upload_to_play_store(
      track: 'internal',
      release_status: 'draft',
      aab: '../build/app/outputs/bundle/release/app-release.aab'
    )
  
    # sh "your_script.sh"
    # You can also use other beta testing services here
  end

  desc "Deploy a new version to the Google Play [Production]"
  lane :deploy do
    upload_to_play_store(track: 'production', aab: '../build/app/outputs/bundle/release/app-release.aab')
  end
end
```

Now you've created two lanes one that will push the build to the internal testing `beta` track and another one `deploy` to production.

Run the `fastlane supply init` to fetch information from the play console about the app (like screenshots, app descriptions, etc).

You can test if your setup works by running:

```
fastlane beta
```

#### Setting up Github Action

Now that we know our configuration works locally, let's set up a GitHub action that will create a beta release every time we push our app into the development branch.

First, we need to add some repository secrets by going to **Repository Settings** > **Secrets** >> **Actions** and add the following variables:

- `PLAY_STORE_UPLOAD_KEY` this is the base64 encoded Keystore file (.jks)
- `KEYSTORE_KEY_ALIAS` this is the upload Keystore key alias
- `KEYSTORE_KEY_PASSWORD` this is our upload key password
- `KEYSTORE_STORE_PASSWORD` this is our store password
- `PLAY_STORE_CONFIG_JSON` this is the google service account's config file

> use the `cat keystore.jks | base64 | pbcopy` to copy the base64 version of jks file to the clipboard.


```
name: Deploy Beta build to Play Store

on:
  push:
    branches:
      - development

# Declare default permissions as read only.
permissions: read-all

jobs:
  fastlane-deploy:
    runs-on: ubuntu-20.04
    steps:
      # Set up Flutter.
      - name: Checkout App Repo
        uses: actions/checkout@v3
        with:
          ref: 'development'

      - name: Setup Java
        uses: actions/setup-java@v1
        with:
          java-version: '12.x'

      - name: Configure Keystore
        run: |
          echo "$PLAY_STORE_UPLOAD_KEY" | base64 --decode > /home/runner/work/repo-name/repo-name/upload-keystore.jks
          echo "storeFile=/home/runner/work/repo-name/repo-name/upload-keystore.jks" >> key.properties
          echo "keyAlias=$KEYSTORE_KEY_ALIAS" >> key.properties
          echo "storePassword=$KEYSTORE_STORE_PASSWORD" >> key.properties
          echo "keyPassword=$KEYSTORE_KEY_PASSWORD" >> key.properties
          echo "$KEYSTORE_KEY_ALIAS"
          echo "$PLAY_STORE_CONFIG_JSON" > /home/runner/work/repo-name/repo-name/service_key.json
          echo "json_key_file(\"/home/runner/work/repo-name/repo-name/service_key.json\")" > /home/runner/work/repo-name/repo-name/android/fastlane/Appfile
          echo "package_name(\"package-name\")" >> /home/runner/work/repo-name/repo-name/android/fastlane/Appfile
          cat /home/runner/work/repo-name/repo-name/android/fastlane/Appfile
        env:
          PLAY_STORE_UPLOAD_KEY: ${{ secrets.PLAY_STORE_UPLOAD_KEY }}
          KEYSTORE_KEY_ALIAS: ${{ secrets.KEYSTORE_KEY_ALIAS }}
          KEYSTORE_KEY_PASSWORD: ${{ secrets.KEYSTORE_KEY_PASSWORD }}
          KEYSTORE_STORE_PASSWORD: ${{ secrets.KEYSTORE_STORE_PASSWORD }}
          PLAY_STORE_CONFIG_JSON: ${{ secrets.PLAY_STORE_CONFIG_JSON }}
        working-directory: android

      - name: Install Flutter & Build App Bundle
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.x'
          channel: 'stable'
      - run: flutter --version && flutter pub get && flutter build appbundle --release
      
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '2.7.2'

      - name: Fastlane Action
        uses: maierj/fastlane-action@v2.2.1
        with:
          lane: "beta"
          subdirectory: "android"
```

> make sure to replace the `repo-name` and `package-name` in the `Configure Keystore` step.

That's all, everytime you push to the development branch a new draft release will be automatically created in the play console.