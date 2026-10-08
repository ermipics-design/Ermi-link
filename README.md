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
          echo "===== After unzip ====="
          ls -la
          find . -maxdepth 3 -type d

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

      - name: Find project root and setup Gradle
        id: project
        run: |
          # Find the real Android project root
          PROJECT_ROOT=$(find . -name "settings.gradle*" -o -name "settings.gradle.kts" | head -1 | xargs dirname 2>/dev/null || echo ".")
          if [ "$PROJECT_ROOT" = "." ] || [ -z "$PROJECT_ROOT" ]; then
            PROJECT_ROOT=$(find . -name "build.gradle*" | head -1 | xargs dirname 2>/dev/null || echo ".")
          fi
          echo "PROJECT_ROOT=$PROJECT_ROOT"
          echo "root=$PROJECT_ROOT" >> $GITHUB_OUTPUT
          cd "$PROJECT_ROOT"
          ls -la

          # Install Gradle if needed
          if [ ! -f "./gradlew" ]; then
            echo "No gradlew found, installing Gradle and generating wrapper..."
            curl -sL https://services.gradle.org/distributions/gradle-8.7-bin.zip -o /tmp/gradle.zip
            unzip -q /tmp/gradle.zip -d /tmp
            export PATH="/tmp/gradle-8.7/bin:$PATH"
            gradle wrapper --gradle-version 8.7
          fi
          chmod +x ./gradlew
          echo "Gradle ready at $PROJECT_ROOT"

      - name: Build Release AAB
        run: |
          cd "${{ steps.project.outputs.root }}"
          ./gradlew bundleRelease --no-daemon --stacktrace || ./gradlew assembleRelease --no-daemon --stacktrace

      - name: Find and show artifacts
        run: |
          echo "===== Searching for AAB / APK ====="
          find . -name "*.aab" -o -name "*.apk" 2>/dev/null | tee /tmp/artifacts.txt
          cat /tmp/artifacts.txt

      - name: Upload AAB/APK
        uses: actions/upload-artifact@v4
        with:
          name: ERMI-LINK-release
          path: |
            **/*.aab
            **/*.apk
          if-no-files-found: warn
          retention-days: 30
