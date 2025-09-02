# firebase-app-distribution

This action uploads artifacts (.apk,.aab or .ipa) to Firebase App Distribution.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Usage

### macOS Apple Silicon (M1/M2/M4) Support

To use this Action on **Apple Silicon macOS runners**, you must perform the following pre-setup:

```yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20.x'
      
      - name: Install Firebase CLI (ARM64 compatible)
        run: npm install -g firebase-tools
      
      - name: Authenticate Firebase using Service Account
        env:
          FIREBASE_SERVICE_ACCOUNT_KEY: ${{ secrets.FIREBASE_SERVICE_ACCOUNT_KEY }}
        run: |
          echo "$FIREBASE_SERVICE_ACCOUNT_KEY" > /tmp/firebase-key.json
          export GOOGLE_APPLICATION_CREDENTIALS=/tmp/firebase-key.json
          echo "GOOGLE_APPLICATION_CREDENTIALS=/tmp/firebase-key.json" >> $GITHUB_ENV
```

Add the following step to your workflow.

```yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: gowthamraj281/firebase-app-distribution@v1
        with:
          file: app.apk or app.IPA
          app: ${{ secrets.FIREBASE_APP_ID }}
          credentials: ${{ secrets.FIREBASE_SERVICE_ACCOUNT_KEY }}
          credentials-file: ${{ secrets.FIREBASE_CREDENTIALS_FILE }}  # Optional: Required only if you don't use serviceCredentialsFileContent.
          release-notes: ""                    # Optional
          release-notes-file: ""               # Optional
          testers: ""                          # Optional
          testers-file: ""                     # Optional
          groups: ""                           # Optional
          groups-file: ""                      # Optional
          debug: false                         # Optional: Default false. A flag you can include to print verbose log output.
          cache: true                          # Optional: Default true. Whether to cache the firebase tools for next job, keeping this "true" will speed up the build.
          upgrade: true                        # Optional: Default true. Whether to attempt to upgrade firebase tools when cache is "true", turning this "false" will speed up the build.
          token: ${{ secrets.FIREBASE_TOKEN }} # Deprecated
```

## Google service account

According to Google, we have to start using [service account for authenticating with Firebase](https://firebase.google.com/docs/app-distribution/authenticate-service-account?platform=ios). 

### Download credentials from Firebase

To get the credentials you can go to your Firebase project -> Project settings -> Service account -> Generate new private key.

![google-service-account-credentials](https://github.com/gowthamraj281/firebase-app-distribution/blob/main/.docs/assests/google-service-account-credentials.png)

And then make sure in Google Cloud IAM, the user "firebase-adminsdk" has the permission "Firebase App Distribution Admin SDK Service Agent".

![google-service-account-permission](https://github.com/gowthamraj281/firebase-app-distribution/blob/main/.docs/assests/google-service-account-permission.png))

### Use credentials in this action

#### 1. Define `credentials` in this action (recommended)

Required Content of Service Credentials private key JSON file. This action will set up the environment variable `GOOGLE_APPLICATION_CREDENTIALS` for you.

```yml
credentials: ${{ secrets.FIREBASE_SERVICE_ACCOUNT_KEY }}
```

#### 2. Define `credentials-file` to use credentials in project file

Check in the credentials file in the project and configure it's path as action secret `FIREBASE_CREDENTIALS_FILE`, use it on the parameter `credentials-file`. This action will set up the environment variable `GOOGLE_APPLICATION_CREDENTIALS` for you.

```yml
credentials-file: ${{ secrets.FIREBASE_CREDENTIALS_FILE }}
```

## Limitations and improvements

1. Currently only works on linux (including macos)
