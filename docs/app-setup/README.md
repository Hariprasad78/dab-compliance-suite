# Application and App Store Setup Guide

## Why app setup is required

Some DAB tests install, launch, and uninstall an application on the DUT. DUT
means Device Under Test. These tests need an application package that can
actually be installed on that device.

## Why the suite does not provide or hardcode a default app

The DAB Compliance Test Suite is generic. It is used across different products,
operating systems, app stores, and package formats. Because of that, the suite
cannot know which app, APK, app store, or store URL is valid for every device.

Users must provide an app that is safe to install and uninstall during testing.
Do not upload random apps. Use a known test app that your DUT can install,
launch, and remove safely.

## Choosing an application for your DUT

Choose an application package based on the DUT platform. The package must match
the install format supported by that platform.

The app should:

- Be supported by the DUT platform.
- Be safe to install and uninstall during repeated test runs.
- Not require user accounts, payments, or manual setup to complete the test.
- Be reachable by the DUT when served by the test tool.

## Android TV / Google TV example

For Android TV and Google TV devices, the application package can be a valid APK
or APKS file.

Run:

```bash
python3 main.py --init
```

When prompted, provide the full path to the APK or APKS file. The tool copies
the configured package under `config/apps/` for later install tests.

## Other platform example

For non-Android platforms, use the package format supported by that platform.
For example, if your platform expects a vendor-specific package type, provide a
valid package in that format during `--init`.

The test suite does not convert packages between platforms. The package you
provide must already be installable by the DUT.

## App store URL setup

Some tests use `applications/install-from-app-store`. For those tests, provide
an app store URL that is valid for the DUT platform and reachable from the DUT.

The app store URL should point to an app supported by that platform and store.
Use `python3 main.py --init` to configure the URL when prompted.

## How the script recognizes the configured app

The setup flow stores the configured application package under `config/apps/`.
Install tests use that configured artifact when `applications/install` is
applicable for the DUT.

Run:

```bash
python3 main.py --init
```

The setup flow collects:

1. Full path to the DUT-compatible application package.
2. App store URL for install-from-app-store tests, if needed.

## Expected behavior for DAB 2.0 and unsupported operations

`applications/install` is a DAB 2.1 app-related test. If `--dab-version 2.0` is
used, DAB 2.1-only install tests should be marked `OPTIONAL_FAILED` before app
payload setup.

If `operations/list` does not report `applications/install` support, the install
test should also be marked `OPTIONAL_FAILED` before app payload setup.

In both cases, the tool should not wait for an app artifact in `config/apps/`.

If the DUT is DAB 2.1, `applications/install` is reported in `operations/list`,
and the configured app artifact is missing, the install test should be skipped.
Run `python3 main.py --init` or place the correct DUT-compatible app package
under `config/apps/`.

## Troubleshooting

### Warning: Waiting for app upload

You may see:

```text
[WARN] [RUNTIME INSTALL] Waiting for 'Sample_App' to be uploaded into config/apps.
```

This warning means the install artifact was not available.

After PR #162, this should not happen for DAB 2.0 or unsupported
`applications/install` paths. Those tests should be marked `OPTIONAL_FAILED`
before app payload setup.

If the install test is applicable, run:

```bash
python3 main.py --init
```

Then configure a valid app package for your DUT. You can also place the correct
DUT-compatible package under `config/apps/`.

If the DUT does not support `applications/install`, verify the DUT response for
`operations/list` and verify the `--dab-version` value used for the run.

### App package cannot be installed

Make sure the package format matches the DUT platform. Android TV and Google TV
devices can use APK or APKS packages. Other devices must use the package format
supported by that platform.

### App store install test fails

Make sure the configured app store URL is valid, reachable from the DUT, and
points to an app supported by that platform and store.
