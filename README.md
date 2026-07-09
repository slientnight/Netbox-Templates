# NetBox Fiber Patch Panel Templates

This repository contains NetBox device-type and module-type YAML templates for fiber patch panels and cassettes, following the [netbox-community/devicetype-library](https://github.com/netbox-community/devicetype-library) conventions.

## Directory Structure

```text
device-types/
└── Generic/
    ├── Fiber-Patch-Panel-1U-4-Bay.yaml
    ├── Fiber-Patch-Panel-3U-6-Bay.yaml
    └── Fiber-Patch-Panel-4U-12-Bay.yaml
module-types/
└── Generic/
    ├── Fiber-Cassette-6-Port-LC-APC-Pass-Through.yaml
    ├── Fiber-Cassette-12-Port-LC-APC-Pass-Through.yaml
    ├── Fiber-Cassette-6-Port-LC-UPC-Pass-Through.yaml
    ├── Fiber-Cassette-12-Port-LC-UPC-Pass-Through.yaml
    └── Fiber-Cassette-6-Port-LC-UPC-to-MPO-Breakout.yaml
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
| `Fiber-Cassette-6-Port-LC-APC-Pass-Through.yaml` | 6 LC/APC front ports, 1:1 pass-through to 6 LC/APC rear ports |
| `Fiber-Cassette-12-Port-LC-APC-Pass-Through.yaml` | 12 LC/APC front ports, 1:1 pass-through to 12 LC/APC rear ports |
| `Fiber-Cassette-6-Port-LC-UPC-Pass-Through.yaml` | 6 LC/UPC front ports, 1:1 pass-through to 6 LC/UPC rear ports |
| `Fiber-Cassette-12-Port-LC-UPC-Pass-Through.yaml` | 12 LC/UPC front ports, 1:1 pass-through to 12 LC/UPC rear ports |
| `Fiber-Cassette-6-Port-LC-UPC-to-MPO-Breakout.yaml` | 6 LC/UPC front ports breaking out to a single MPO rear port (Method A polarity) |

## Device-Type / Module-Type Relationship

The patch panels are **enclosures** that define module bays — they do not have ports of their own. The cassettes are **module types** that slot into those bays, providing LC front ports that pass through to either individual rear ports (LC-to-LC) or a shared MPO rear connector (LC-to-MPO breakout).

Each front port references its corresponding rear port by name using the `{module}` token, which NetBox resolves to the module bay name at install time.

## MPO Breakout Cassettes

The MPO breakout templates model a common fiber cassette where multiple duplex LC front ports fan into a single MPO rear connector. In NetBox, this is represented using the rear port `positions` field:

- A single rear port of type `mpo` is defined with `positions: N`, where N is the number of front ports (logical duplex circuits)
- Each front port maps to the same rear port but claims a unique `rear_port_position` (1 through N)
- Positions represent logical duplex connections, not individual fiber strands (6 positions = 6 duplex pairs = 12 physical fibers)

For example, the 6-port LC/UPC to MPO template has:
- 6 LC/UPC front ports (each a duplex pair)
- 1 MPO rear port with `positions: 6`
- Front port 01 → MPO position 1, port 02 → position 2, etc.

The Method A (straight) polarity mapping is used, meaning fiber positions are not flipped end-to-end.

For more details on how NetBox models multi-position rear ports and breakout paths, see the [NetBox Rear Port documentation](https://netboxlabs.com/docs/netbox/en/stable/models/dcim/rearport/).

## Port Counts (Fully Populated)

When every bay is populated with a 6-port cassette:

| Panel | Bays | Ports per Bay | Total Ports |
|-------|------|---------------|-------------|
| 1U – 4 Bay | 4 | 6 | **24** |
| 3U – 6 Bay | 6 | 6 | **36** |
| 4U – 12 Bay | 12 | 6 | **72** |

When every bay is populated with a 12-port cassette:

| Panel | Bays | Ports per Bay | Total Ports |
|-------|------|---------------|-------------|
| 1U – 4 Bay | 4 | 12 | **48** |
| 3U – 6 Bay | 6 | 12 | **72** |
| 4U – 12 Bay | 12 | 12 | **144** |

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

## Contributors

- [@d3ej](https://github.com/d3ej) — Thanks for the contribution! ([PR #1](https://github.com/slientnight/Netbox-Templates/pull/1))
