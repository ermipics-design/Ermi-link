name: Build ERMI LINK Android AAB

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Unzip project
        run: |
          unzip -q ERMI-LINK-PLAY-READY.zip
          ls -la

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

      - name: Show project build files
        run: |
          echo "===== settings.gradle ====="
          cat ERMI-LINK-PLAY-READY/settings.gradle || true
          echo ""
          echo "===== root build.gradle ====="
          cat ERMI-LINK-PLAY-READY/build.gradle || true
          echo ""
          echo "===== app/build.gradle ====="
          cat ERMI-LINK-PLAY-READY/app/build.gradle || true
          echo ""
          echo "===== gradle.properties ====="
          cat ERMI-LINK-PLAY-READY/gradle.properties || true

      - name: Install Gradle 8.4 and generate wrapper
        run: |
          cd ERMI-LINK-PLAY-READY
          curl -sL https://services.gradle.org/distributions/gradle-8.4-bin.zip -o /tmp/gradle.zip
          unzip -q /tmp/gradle.zip -d /tmp
          export PATH="/tmp/gradle-8.4/bin:$PATH"
          gradle wrapper --gradle-version 8.4
          chmod +x ./gradlew

      - name: Build Release AAB
        run: |
          cd ERMI-LINK-PLAY-READY
          ./gradlew bundleRelease --no-daemon --stacktrace

      - name: Find artifacts
        run: |
          find . -name "*.aab" -o -name "*.apk" | tee /tmp/arts.txt
          cat /tmp/arts.txt

      - name: Upload AAB/APK
        uses: actions/upload-artifact@v4
        with:
          name: ERMI-LINK-release
          path: |
            **/*.aab
            **/*.apk
          if-no-files-found: warn
