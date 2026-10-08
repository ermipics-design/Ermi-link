name: Build ERMI LINK Android AAB

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Setup Android SDK
        run: |
          mkdir -p $HOME/android-sdk/cmdline-tools
          curl -o cmdline-tools.zip https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip
          unzip -q cmdline-tools.zip -d $HOME/android-sdk/cmdline-tools
          mv $HOME/android-sdk/cmdline-tools/cmdline-tools $HOME/android-sdk/cmdline-tools/latest
          echo "ANDROID_HOME=$HOME/android-sdk" >> $GITHUB_ENV
          echo "ANDROID_SDK_ROOT=$HOME/android-sdk" >> $GITHUB_ENV
          echo "$HOME/android-sdk/cmdline-tools/latest/bin" >> $GITHUB_PATH
          echo "$HOME/android-sdk/platform-tools" >> $GITHUB_PATH

      - name: Accept licenses and install packages
        run: |
          yes | sdkmanager --licenses || true
          sdkmanager "platform-tools" "platforms;android-36" "build-tools;36.0.0"

      - name: Install Gradle
        run: |
          curl -sL https://services.gradle.org/distributions/gradle-8.7-bin.zip -o gradle.zip
          unzip -q gradle.zip
          echo "$PWD/gradle-8.7/bin" >> $GITHUB_PATH

      - name: Generate Gradle Wrapper
        run: gradle wrapper --gradle-version 8.7

      - name: Make Gradle executable
        run: chmod +x ./gradlew

      - name: Build Release AAB
        run: ./gradlew bundleRelease --no-daemon --stacktrace

      - name: Upload ERMI LINK AAB
        uses: actions/upload-artifact@v4
        with:
          name: ERMI-LINK-release-AAB
          path: app/build/outputs/bundle/release/*.aab
          if-no-files-found: error
        
