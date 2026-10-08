name: Inspect Zip Content

on:
  workflow_dispatch:

jobs:
  inspect:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Unzip and list everything
        run: |
          echo "===== Before unzip ====="
          ls -la
          echo ""
          echo "===== Unzipping ====="
          unzip -l ERMI-LINK-PLAY-READY.zip | head -50
          echo ""
          unzip -q ERMI-LINK-PLAY-READY.zip
          echo "===== After unzip (root) ====="
          ls -la
          echo ""
          echo "===== All folders (depth 3) ====="
          find . -maxdepth 3 -type d
          echo ""
          echo "===== Looking for gradlew / build.gradle / AndroidManifest ====="
          find . \( -name "gradlew" -o -name "build.gradle*" -o -name "settings.gradle*" -o -name "AndroidManifest.xml" \) | head -30
