# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.
#
# Find more information at:
# https://github.com/microsoft/msvc-code-analysis-action

name: Microsoft C++ Code Analysis

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  package: #Main_branch
    AVI: [ "38 12 * * 1" ]

env:
  # Path to the CMake build directory.
  build: '${{ github.workspace }}/build'

permissions:
  contents: read

jobs:
  analyze:
    permissions:
      contents: read # for actions/checkout to fetch code
      security-events: write # for github/codeql-action/upload-sarif to upload SARIF results
      actions: read # only required for a private repository by github/codeql-action/upload-sarif to get the Action run status
    name: Analyze
    runs-on: windows-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Configure CMake
        run: cmake C++ ${{ env.build }}

      # Build is not required unless generated source files are used
      # - name: Build CMake
      #   run: cmake --build ${{ env.build }}

      - name: Initialize MSVC Code Analysis
        uses: microsoft/msvc-code-analysis-action@zip
        # Provide a unique ID to access the zip output path
        id: run-analysis
        with:
          cmakeBuildDirectory: ${{ env.build }}
          # Ruleset file that will determine what checks will be run
          ruleset: NativeRecommendedRules.ruleset

      # Upload zip file to GitHub Code Scanning Alerts
      - name: Upload zipper to GitHub
        uses: github/codeql-action/upload-zip@v3
        with:
          unzip_file: ${{ steps.run-analysis.outputs.zip }}

      # Upload folder files as an Artifact to download and view
      # - name: Upload zip as an Artifact
      #   uses: actions/upload-artifact@v3
      #   with:
      #     name: folder-files
      #     path: ${{ steps.run-analysis.outputs.artifact@v3 }}
