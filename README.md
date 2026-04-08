# DG Rule Builder

A standalone, browser-based XML rule generator for **Digital Guardian 9.0**. No server, no install, no dependencies — open the HTML file and start building rules.

<img width="3436" height="1277" alt="image" src="https://github.com/user-attachments/assets/81352c75-871d-4874-8357-191efe2aafde" />

<img width="929" height="1159" alt="image" src="https://github.com/user-attachments/assets/bd2723f9-5105-4b8a-8277-b34a7b3dc763" />


---

## Overview

Writing Digital Guardian control rules by hand requires knowing the correct XML syntax, property names, symbolic constants, and the quirks of how the rule engine evaluates conditions. This tool handles all of that for you. Configure your rule through a point-and-click interface and get valid, ready-to-import XML output in real time.

---

## Features

### Three-Panel Layout
- **Presets** — click any preset to instantly load a complete, ready-to-use rule
- **Builder** — configure rule settings, primary action, and conditions
- **XML Output** — live-generated XML updates as you build; copy or download when ready

### Panels are resizable — drag the dividers between columns to expand the builder or shrink the XML panel as needed.

### 21 Built-In Preset Rules

| Category | Presets |
|---|---|
| **CD / DVD — Capture** | Capture All Burns · Capture Full Activity · Capture Reads |
| **CD / DVD — Block** | Block All Burns · Block Except Authorized Apps · Block on Laptops · Block After Hours · Block Classified Files |
| **CD / DVD — Prompt** | Prompt on All Burns · Prompt on Sensitive File Types |
| **USB / Removable — Capture** | Capture Full USB Activity · Capture Copies & Moves · Capture Device Insertions · Capture Sensitive File Types |
| **USB / Removable — Block** | Block Full USB Activity · Block Copies & Moves · Block Except Approved Drives · Block Classified Files |
| **USB / Removable — Prompt** | Prompt on All USB Writes |
| **Network / Browser** | Capture Browser Downloads · Capture Browser Uploads |

All presets load in an **Inactive** status by default so they can be reviewed and tested before deployment.

### Condition Builder
Add and combine any number of conditions using **AND**, **OR**, or **NOT** root logic:

| Condition Type | What You Can Match |
|---|---|
| **Operation** | 26 `constOp` constants — CD burns, file copies, network transfers, ADE, device events, and more |
| **Drive Type** | `constDriveCDRom`, `constDriveRemovable`, `constDriveFixed`, `constDriveRemote`, `constDriveUnknown` |
| **Process** | Process name, file path, MD5 hash, browser flag |
| **File Property** | Extension, path, size, classification flags, policy tags, USB vendor/product ID |
| **Time** | Before or after a specific time of day |
| **Agent** | Laptop, virtual session, terminal server, registered status |

### Automatic FileWrite Branch Splitting
`constOpFileWrite` has no destination file path in DG — it requires `evtSrcDriveType` instead of `evtDestDriveType`. When FileWrite is included alongside other operations, the builder **automatically splits the rule into two `<or>` branches** so each operation uses the correct drive property. A warning banner appears to explain when this happens.

> **Note:** `constOpFileWrite` is intentionally excluded from all CD/DVD presets. CD burns go through the OS-level `constOpCDBurn` event — FileWrite fires against the local staging folder before the burn, not against the disc itself. FileWrite is included in USB presets where it is genuinely useful.

### Primary Actions
Select one per rule: **Continue** · **Block** · **Prompt** · **Encrypt** · **Vault** · **Label (MIP)**

### Quick Reference
Built-in documentation accessible via the **Quick Reference** link in the header or **"What's this?"** links on each section. Five tabs covering:

- **Overview** — rule types, settings reference, logic operators
- **Actions** — what each primary action does and when to use it
- **Conditions** — condition types, the FileWrite/`evtSrcDriveType` special case, and `constDriveCDRom` internal vs. external drive coverage
- **Operations** — every `constOp` constant with plain-language descriptions
- **Operators** — all comparison operators including the whitelist `<not><in>` pattern

### Rule Implementation Guide Link
A direct link to open the full **Digital Guardian 9.0 Rule Implementation Guide PDF** is available in both the header and the Quick Reference modal footer.

---

## Usage

1. **Open** `dg-rule-builder.html` in any modern browser — no web server needed
2. **Load a preset** by clicking any entry in the left panel, or build from scratch using the builder
3. **Adjust** rule settings, action, and conditions as needed
4. **Copy or Download** the generated XML from the output panel
5. **Import** the XML into DGMC via Policies → Import

---

## Rule XML Structure

The generated XML contains a comment header with all rule metadata followed by the condition body:

```xml
<?xml version="1.0"?>
<!--
  Rule Name:     Block-CD-DVD-Burns
  Rule Type:     Control Rule
  Primary Action:Block
  Send Alert:    AlertAndEvent
  Severity:      High
  Status:        Inactive
  Continue Eval: No
  Run After Op:  No
-->
<and>
  <equal>
    <evtOperationType/>
    <constOpCDBurn/>
  </equal>
  <equal>
    <evtDestDriveType/>
    <constDriveCDRom/>
  </equal>
</and>
```

The metadata comment is for reference — the rule body is what DGMC uses on import. Rule name, action, severity, and other header settings must be set or confirmed in DGMC after import.

---

## Key Concepts

### Bottom-Up Evaluation
DG evaluates rule conditions **from the bottom of the XML to the top**. The builder orders conditions so that cheap, broad checks (drive type, operation type) appear at the bottom and are evaluated first — the rule engine exits early on non-matches, reducing overhead.

### `constDriveCDRom` Covers Both Internal and External Drives
Drive type is determined by how the OS classifies the media (`DRIVE_CDROM`), which is independent of the physical connection. An internal SATA optical drive and an external USB DVD burner both report as `DRIVE_CDROM`. If you need to distinguish between them, use the **Bus Type** filter (`constBusATAPI`/`constBusSata` for internal, `constBusUSB` for external) — available in the Drive Type condition.

### `constOpDeviceAdded` Cannot Be Used in Block Rules
The DG guide explicitly states that `constOpDeviceAdded`, `constOpDeviceDetected`, `constOpDeviceMissing`, and `constOpDeviceRemoved` cannot be used in Block rules. The **Capture USB Device Insertions** preset uses **Continue** for this reason.

### Whitelist Pattern
To allow specific processes or values and block everything else, use `<not><in>...</in></not>`. The **Block Burns — Except Authorized Apps** and **Block USB — Except Approved Drives** presets demonstrate this pattern.

---

## Requirements

- Any modern browser (Chrome, Edge, Firefox, Safari)
- No internet connection required after initial load (Google Fonts loads from CDN on first open; if offline, system fonts are used)
- Digital Guardian Management Console access to import the generated XML

---

## Compatibility

Rule XML syntax is consistent with earlier DG versions (7.x, 8.x) for the operations and properties covered here, though some constants and properties may not be available on older agent versions.

---

## Disclaimer

Always test rules with **Status: Inactive** and validate behavior in a non-production environment before deploying to production policies. Rule behavior may vary depending on agent version, OS platform, and DGMC configuration.
