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
        uses: android-actions/setup-android@v3

      - name: Make Gradle executable
        run: chmod +x ./gradlew

      - name: Build Release AAB
        run: ./gradlew bundleRelease --no-daemon

      - name: Upload ERMI LINK AAB
        uses: actions/upload-artifact@v4
        with:
          name: ERMI-LINK-release-AAB
          path: app/build/outputs/bundle/release/*.aab
