# ChirpStack device-profiles

This repository contains ChirpStack device-profiles for LoRaWAN devices grouped by
vendor. A device-profile contains important information about the capabilities
of the device. For example which LoRaWAN mac-version has been implemented,
which regions are supported, if the device supports Class-B or Class-C, etc...
The aim is to build a complete list of LoRaWAN device-profiles that then can
be imported by ChirpStack or potentially any other LNS.

## Importing into ChirpStack

Example command:

```bash
# Clone the chirpstack-device-profiles repository.
git clone https://github.com/chirpstack/chirpstack-device-profiles.git

# Execute the import command.
chirpstack -c /etc/chirpstack import-device-profiles -d /path/to/chirpstack-device-profiles
```

See also: [Device profiles documentation](https://www.chirpstack.io/docs/chirpstack/use/device-profiles.html)

## Adding a new device-profile

### Fork and clone

* Create a fork of this repository (if not done already).
* Clone your local fork to your computer.

### Starting the web-interface

* Please make sure that you have Docker Compose installed.
* In the root of this repository, execute `make serve`.
* Once `Starting server, bind: 0.0.0.0:8090` appears, open the web-interface in your browser by navigating to [http://localhost:8090](http://localhost:8090).

### Add Vendor(s)

* In the left menu, click the _Add vendor_ button.
* Fill in the form and click _Submit_.

### Add profile(s)

* Select a vendor in the left menu (if no vendor is selected).
* Click _Profiles_ in the left menu.
* Click the _Add profile_ menu entry.
* Fill in the form and click _Submit_.

### Add codec(s)

* Select a vendor in the left menu (if no vendor is selected).
* Click _Codecs_ in the left menu.
* Click the _Add codec_ menu entry.
* Fill in the form, before clicking _Submit_ it is a good idea to click _Run codec tests_.

### Add device(s)

* Select a vendor in the left menu (if no vendor is selected).
* Click _Devices_ in the left menu.
* Click the _Add device_ menu entry.
* Fill in the form and add at least one firmware version by clicking the _Add firmware version_ button.
    * For each firmware version you can select one or multiple profiles and optionally a codec.

### Create pull-request

Once you have added the vendor(s), profile(s), codec(s) and device(s) you wish
to add to this repository you must commit the changes using `git`, push these
to your fork of this repository and create a pull-request in GitHub.

## Structure

Example structure for an `example-vendor` with an `example` device:

```
vendors/
└── example-vendor
    ├── codecs
    │   ├── example.js
    │   ├── test_decode_example.json
    │   └── test_encode_example.json
    ├── devices
    │   └── example.toml
    ├── profiles
    │   └── example-EU868.toml
    └── vendor.toml
```

Please take a look at the `vendors/example-vendor` example documented
configuration files.

### `vendors/example-vendor`

This is the root of the example vendor. It must contain a `vendor.toml`
file. This `vendor.toml`.

### `vendors/example-vendor/codecs`

This directory contains the payload codecs. Codecs can be used by one or
multiple devices. E.g. some vendors have a generic payload codec.

Each codec is expected to have tests for encoding and decoding. If the
codec filename is `example.js`, then you should create two test-files
called `test_decode_example.json` (thus + `test_decode_` prefix and `.json`
extension) and `test_encode_example.json`.

### `vendors/example-vendor/devices`

This directory contains the devices. Each device will have its own `.toml`
configuration.

### `vendors/example-vendor/profiles`

This directory contains the profiles. These profiles can be used by one
or multiple devices. The profile also defines the region.

## License

This repository is distributed under the MIT license. See also `LICENSE`.

# Nexelec device-profiles

This section is an internal guide for the Nexelec team on how to add new
device profiles to this repository.

## Vendor directories

We use two kinds of vendor directories:

| Directory | Purpose |
|---|---|
| `vendors/nexelec/` | Nexelec's own internal devices |
| `vendors/nexelec-{brand}/` | 3rd-party devices integrated into the Nexelec platform (e.g. `nexelec-rak`, `nexelec-kuando`) |

When adding a Nexelec product, always work under `vendors/nexelec/`.
When integrating a 3rd-party device, create or use `vendors/nexelec-{brand}/`.

## File structure

Files for internal Nexelec devices live under `vendors/nexelec/`:

```
vendors/nexelec/
├── vendor.toml                    # vendor metadata (edit rarely)
├── devices/
│   └── {product-ref}.toml        # one file per device model
├── profiles/
│   └── {REGION}_{MAC}_{RP}_{CLASS}.toml   # one file per region × capability
└── codecs/
    ├── {codec-name}.js            # payload decoder/encoder (if any)
    ├── test_decode_{codec-name}.json
    └── test_encode_{codec-name}.json
```

Profile files are shared across devices. If your new device uses the same
LoRaWAN parameters as an existing one, reuse the existing profile file.

## Profile naming convention

Profile filenames encode the key LoRaWAN parameters:

```
EU868_104_RP002-103_A.toml
│     │   │         └── Class suffix: A = Class A only
│     │   │                           B = supports Class B
│     │   │                           C = supports Class C
│     │   └── Reg params revision, dots removed: RP002-1.0.3 → RP002-103
│     └── MAC version, dots removed: 1.0.4 → 104
└── Region: EU868, US915, AS923, AU915, …
```

Example mapping for a typical Nexelec device:

| Parameter | Value | In filename |
|---|---|---|
| Region | EU868 | `EU868` |
| MAC version | 1.0.4 | `104` |
| Reg params revision | RP002-1.0.3 | `RP002-103` |
| Supports Class B/C | No | `A` |

→ filename: `EU868_104_RP002-103_A.toml`

## Device file naming

### File name

```
{product_ref}-{name}.toml
```

- `product_ref` is the base reference **without** the radio suffix (`LS` = LoRaSigFox, `LR` = LoRa).
- `name` is the product commercial name in lowercase kebab-case.

Examples:

| Product refs | File name |
|---|---|
| A300LS, A300LR | `a300-flow-core.toml` |
| X520LS, X520LR | `x520-rise.toml` |

### `name` field inside the TOML

```
{PRODUCT_REF} {NAME} ({ALL_PRODUCT_REFS})
```

- `PRODUCT_REF` — base ref without radio suffix, uppercase.
- `NAME` — commercial name, uppercase.
- `ALL_PRODUCT_REFS` — all refs including radio suffixes (LS, LR), space-separated.

Examples:

```toml
name = "A300 FLOW CORE (A300LS A300LR)"
name = "X520 RISE (X520LS X520LR)"
```

## Generating UUIDs

Every vendor, device, and profile requires a unique UUID in its `id` field.
Generate one with PowerShell:

```powershell
[System.Guid]::NewGuid().ToString()
```

Never reuse an existing UUID from another file.

## How to add a new device — manual TOML approach

This is the fastest path when the profiles already exist for the target region
and MAC version.

### 1. Check whether a profile already exists

```
vendors/nexelec/profiles/
```

If a file matching your device's region + MAC version + reg params + class
already exists, skip to step 3.

### 2. Create a new profile file

Copy an existing profile and update the fields. Example for EU868 Class A:

```toml
[profile]
id = "<new UUID>"
vendor_profile_id = 0
region = "EU868"
mac_version = "1.0.4"
reg_params_revision = "RP002-1.0.3"
supports_otaa = true
supports_class_b = false
supports_class_c = false
max_eirp = 0

[profile.abp]
rx1_delay = 0
rx1_dr_offset = 0
rx2_dr = 0
rx2_freq = 0

[profile.class_b]
timeout_secs = 0
ping_slot_nb_k = 0
ping_slot_dr = 0
ping_slot_freq = 0

[profile.class_c]
timeout_secs = 0
```

Save it as `vendors/nexelec/profiles/{REGION}_{MAC}_{RP}_{CLASS}.toml`.

### 3. Create the device file

Create `vendors/nexelec/devices/{product_ref}-{name}.toml`
(e.g. `a300-flow-core.toml` for product refs A300LS / A300LR):

```toml
[device]
id = "<new UUID>"
name = "A300 FLOW CORE (A300LS A300LR)"
description = "A300LS A300LR"

[[device.firmware]]
version = "1.0.0"
profiles = [
    "EU868_104_RP002-103_A.toml",
    "US915_104_RP002-103_A.toml",
]

[device.metadata]
documentation_url = "https://support.nexelec.fr/..."
```

Add one `profiles` entry per supported region.

### 4. Add a codec (if the device has a payload format)

Create `vendors/nexelec/codecs/{codec-name}.js` implementing
`decodeUplink(input)` and `encodeDownlink(input)` per the TS013-1.0.0 spec.

Create the two required test files:

- `test_decode_{codec-name}.json`
- `test_encode_{codec-name}.json`

Reference the codec from `[[device.firmware]]` by adding:

```toml
codec = "{codec-name}.js"
```

### 5. Run the tests

```bash
make test
```

All codec tests must pass before opening a PR.

## How to add a new device — web UI approach

Use this path if you prefer a guided form interface.

```bash
make serve
```

Open [http://localhost:8090](http://localhost:8090), select the **Nexelec**
vendor in the left menu, then use the **Profiles**, **Codecs**, and **Devices**
menus to add entries through the form. The tool writes the TOML files for you.

This requires Docker to be installed and running.
