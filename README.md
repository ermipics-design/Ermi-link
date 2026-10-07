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
          cache: 'gradle'

      - name: Setup Android SDK
        uses: android-actions/setup-android@v4.0.4
        with:
          packages: 'platform-tools'

      - name: Install Android packages
        run: |
          sdkmanager "platforms;android-36" "build-tools;36.0.0"

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
          
