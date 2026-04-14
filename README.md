# NetBox Fiber Patch Panel Templates

This repository contains NetBox device-type and module-type YAML templates for fiber patch panels and cassettes, following the [netbox-community/devicetype-library](https://github.com/netbox-community/devicetype-library) conventions.

## Directory Structure

```text
device-types/
└── Generic/
    ├── Fiber-Patch-Panel-3U-6-Bay.yaml
    └── Fiber-Patch-Panel-4U-12-Bay.yaml
module-types/
└── Generic/
    └── Fiber-Cassette-6-Port-LC-APC-Pass-Through.yaml
```

## Templates

### Device Types

| Template | Description |
|----------|-------------|
| `Fiber-Patch-Panel-3U-6-Bay.yaml` | 3U fiber patch panel enclosure with 6 module bays |
| `Fiber-Patch-Panel-4U-12-Bay.yaml` | 4U fiber patch panel enclosure with 12 module bays |

### Module Types

| Template | Description |
|----------|-------------|
| `Fiber-Cassette-6-Port-LC-APC-Pass-Through.yaml` | Fiber cassette module with 6 LC/APC front ports passing through to bare fiber on the rear, 1:1 mapping |

## Device-Type / Module-Type Relationship

The patch panels are **enclosures** that define module bays — they do not have ports of their own. The cassette is a **module type** that slots into those bays, providing LC/APC front ports that pass through to bare fiber stubs on the rear of the cassette.

Each front port references its corresponding rear port by name using the `{module}` token, which NetBox resolves to the module bay name at install time.

## Port Counts (Fully Populated)

When every bay is populated with a 6-port cassette:

| Panel | Bays | Ports per Bay | Total Ports |
|-------|------|---------------|-------------|
| 3U – 6 Bay | 6 | 6 | **36** |
| 4U – 12 Bay | 12 | 6 | **72** |

## Importing into NetBox

### Via the Web UI

1. Navigate to **Devices → Device Types** (or **Devices → Module Types**).
2. Click the **Import** button in the top-right corner.
3. Paste the contents of the YAML file into the text area, or upload the file directly.
4. Click **Submit** to create the device type or module type.

Import the device types first, then the module type. Once both exist you can assign the cassette module type to any module bay on a patch panel instance.

### Via the REST API

Use the NetBox REST API to create device types and module types programmatically.

```bash
# Import a device type
curl -X POST \
  -H "Authorization: Token $NETBOX_TOKEN" \
  -H "Content-Type: application/json" \
  https://<netbox-host>/api/dcim/device-types/ \
  -d @- <<'EOF'
{
  "manufacturer": {"slug": "generic"},
  "model": "Fiber Patch Panel - 3U - 6 Bay",
  "slug": "generic-fiber-patch-panel-3u-6-bay",
  "u_height": 3.0,
  "is_full_depth": false
}
EOF

# Import a module type
curl -X POST \
  -H "Authorization: Token $NETBOX_TOKEN" \
  -H "Content-Type: application/json" \
  https://<netbox-host>/api/dcim/module-types/ \
  -d @- <<'EOF'
{
  "manufacturer": {"slug": "generic"},
  "model": "Fiber Cassette - 6 Port LC/APC Pass-Through",
  "slug": "generic-fiber-cassette-6-port-lc-apc-pass-through"
}
EOF
```

Alternatively, you can use the [netbox-community/devicetype-library](https://github.com/netbox-community/devicetype-library) import scripts to bulk-load YAML templates from a repository that follows this directory layout.
