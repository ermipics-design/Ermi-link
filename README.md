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
          distribution: temurin
          java-version: '17'
          cache: gradle

      - name: Set up Android SDK
  uses: android-actions/setup-android@v4
  with:
    packages: 'platform-tools'
    accept-android-sdk-licenses: true
    log-accepted-android-sdk-licenses: true

      - name: Accept Android licenses
        run: |
          yes | sdkmanager --licenses || true
          sdkmanager "platform-tools" "platforms;android-36" "build-tools;35.0.0"

      - name: Extract ERMI LINK project
        run: |
          echo "===== ZIP FILES ====="
          find . -maxdepth 2 -type f -name "*.zip" -print

          ZIP_FILE=$(find . -maxdepth 2 -type f -name "ERMI-LINK-PLAY-READY.zip" | head -1)

          if [ -z "$ZIP_FILE" ]; then
            echo "ERROR: ERMI-LINK-PLAY-READY.zip not found"
            exit 1
          fi

          echo "Using: $ZIP_FILE"

          mkdir -p extracted
          unzip -q "$ZIP_FILE" -d extracted

          echo "===== Extracted files ====="
          find extracted -maxdepth 4 -type f | head -100

      - name: Locate Android project
        run: |
          PROJECT_DIR=$(find extracted -type f -name "settings.gradle" -o -name "settings.gradle.kts" \
            | head -1 | xargs -r dirname)

          if [ -z "$PROJECT_DIR" ]; then
            echo "ERROR: Android project settings.gradle not found"
            exit 1
          fi

          echo "Android project found at: $PROJECT_DIR"

          rm -rf android-project
          mkdir -p android-project

          cp -a "$PROJECT_DIR"/. android-project/

          echo "===== Android project ====="
          find android-project -maxdepth 3 -type f | sort | head -150

      - name: Verify Android project
        run: |
          cd android-project

          echo "===== Project files ====="
          ls -la

          echo "===== App module ====="
          ls -la app || true

          test -f settings.gradle || test -f settings.gradle.kts
          test -d app

      - name: Install Gradle 8.13
        run: |
          curl -sL https://services.gradle.org/distributions/gradle-8.13-bin.zip -o gradle.zip
          unzip -q gradle.zip
          echo "$PWD/gradle-8.13/bin" >> "$GITHUB_PATH"

      - name: Generate Gradle Wrapper
        working-directory: android-project
        run: |
          gradle wrapper --gradle-version 8.13

      - name: Make Gradle executable
        working-directory: android-project
        run: chmod +x gradlew

      - name: Build Release AAB
        working-directory: android-project
        run: ./gradlew bundleRelease --no-daemon --stacktrace

      - name: Find AAB
        run: |
          echo "===== AAB FILES ====="
          find android-project -type f -name "*.aab" -print

      - name: Upload ERMI LINK AAB
        uses: actions/upload-artifact@v4
        with:
          name: ERMI-LINK-release-AAB
          path: "android-project/**/build/outputs/bundle/**/*.aab"
          if-no-files-found: error
