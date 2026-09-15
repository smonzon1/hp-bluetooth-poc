# ICA Bluetooth Telemetry Interface for HP
## Proof-of-Concept Event Exposure

## 1. Introduction

As part of the ICA proof-of-concept, the sample application can expose a set of Bluetooth telemetry events to HP. These events are intended to provide a simple and practical interface for consuming Bluetooth-related call, device, radio, and firmware telemetry.

The interface includes:

- **Parsed ICA events** for common Bluetooth scenarios
- **Raw firmware telemetry statistics** used as inputs for Bluetooth audio quality scoring

This combination allows HP to integrate at the level most appropriate for its use case: either through normalized application-level events or through access to the underlying firmware telemetry inputs.

### Scope of This POC

This proof-of-concept targets users participating in **Microsoft Teams calls** using **Classic Bluetooth audio headsets (SCO/eSCO)**. Bluetooth LE Audio is not included in this initial implementation.

The telemetry is designed to help identify scenarios such as:
- Audio breakups ("you are breaking up")
- Choppy or distorted voice quality
- Temporary loss of audio
- Complete inability to hear the remote party

---

## 2. Summary of Exposed Events

The proof-of-concept interface exposes the following event types:

1. **Bluetooth Audio Call Started**
2. **Bluetooth Audio Call Stopped**
3. **Bluetooth Audio Call Quality**
4. **Bluetooth Device Updated**
5. **Bluetooth Radio State Changed**
6. **Bluetooth Classic Audio Statistics**

Together, these events cover call lifecycle, call quality, device state changes, Bluetooth radio state changes, and firmware telemetry inputs for quality scoring.

---

## 3. Event Descriptions

### 3.1 Bluetooth Audio Call Started

**Event name:** `BluetoothAudioCallStarted`

This event indicates that a Bluetooth audio call has started. In practical terms, it represents the point at which the Bluetooth audio path for the call is established.

**Typical usage**
- Detect the beginning of a Bluetooth voice call
- Start call-related timers or logging
- Trigger user interface state changes
- Correlate the call with subsequent quality events

**Expected frequency**
- Emitted once when a call starts
- Event-driven, not periodic

**Payload**

| Field | Type | Description |
|---|---|---|
| `TimestampMs` | `long` | Timestamp of the event in milliseconds since Unix epoch |
| `SessionId` | `int` | Identifier for the call session |
| `DeviceAddress` | `string` | Bluetooth address of the device associated with the call |
| `StartedAtMs` | `long` | Timestamp representing when the call started, in milliseconds since Unix epoch |

---

### 3.2 Bluetooth Audio Call Stopped

**Event name:** `BluetoothAudioCallStopped`

This event indicates that a Bluetooth audio call has ended.

**Typical usage**
- Detect the end of a Bluetooth voice call
- Stop call timers or active call processing
- Calculate call duration
- Finalize call-related reporting

**Expected frequency**
- Emitted once when a call ends
- Event-driven, not periodic

**Payload**

| Field | Type | Description |
|---|---|---|
| `TimestampMs` | `long` | Timestamp of the event in milliseconds since Unix epoch |
| `SessionId` | `int` | Identifier for the call session |
| `DeviceAddress` | `string` | Bluetooth address of the device associated with the call |
| `DurationMs` | `long` | Duration of the call in milliseconds |

---

### 3.3 Bluetooth Audio Call Quality

**Event name:** `BluetoothAudioCallQuality`

This event provides quality-related telemetry for an active Bluetooth audio call during a Microsoft Teams voice call. It is intended to expose call-quality measurements during the call rather than only after the fact.

**Typical usage**
- Monitor call quality over time
- Detect degraded call experience
- Support quality dashboards or indicators
- Identify periods of poor Bluetooth link quality that may correlate with user experience issues
- Correlate call quality with device or platform behavior

**Expected frequency**
- Emitted as quality snapshots during an active Teams call
- Published approximately every 5 seconds while a call is active
- Event-driven based on sampling intervals and call state

**Payload**

| Field | Type | Description |
|---|---|---|
| `TimestampMs` | `long` | Timestamp of the event in milliseconds since Unix epoch |
| `DeviceAddress` | `string` | Anonymized device identifier |
| `RxScore` | `double` | Quality score for the receive audio path (0-5 scale, where higher indicates worse quality) |
| `TxScore` | `double` | Quality score for the transmit audio path (0-5 scale, where higher indicates worse quality) |
| `RxStatus` | `string` | Status indicator for the receive audio path (Green/Yellow/Red) |
| `TxStatus` | `string` | Status indicator for the transmit audio path (Green/Yellow/Red) |

---

### 3.4 Bluetooth Device Updated

**Event name:** `BluetoothDeviceUpdated`

This event indicates that the state or metadata of a Bluetooth device has changed. It is a consolidated device event intended to cover multiple types of device changes through a single payload.

Examples of changes that may trigger this event include:
- Device connection
- Device disconnection
- Battery information update
- Device removal or unpairing
- LE Audio state changes

**Typical usage**
- Maintain a current device inventory or cache
- Update displayed device information
- Detect device connection and disconnection activity
- Track battery and device capability changes
- Available regardless of whether a Microsoft Teams session is active
- Applies to any Bluetooth type, including BLE

**Expected frequency**
- Emitted whenever a tracked Bluetooth device changes
- Event-driven, not periodic
- May occur many times over the lifecycle of a device

**Payload**

| Field | Type | Description |
|---|---|---|
| `TimestampMs` | `long` | Timestamp of the event in milliseconds since Unix epoch |
| `DeviceAddress` | `string` | Bluetooth address of the device |
| `AdapterAddress` | `string` | Bluetooth address of the local adapter |
| `FriendlyName` | `string` | Human-readable device name |
| `DeviceType` | `string` | Device type classification |
| `IsLEDevice` | `bool` | Indicates whether the device is a Bluetooth Low Energy device |
| `IsLeAudio` | `bool` | Indicates whether the device supports LE Audio |
| `Batteries` | `ComponentBattery[]` | Battery information for one or more device components, if available |
| `IsConnected` | `bool` | Indicates whether the device is currently connected |
| `LastConnectedTimestampMs` | `long` | Timestamp of the most recent connection, in milliseconds since Unix epoch |
| `ContainerId` | `string` | Device container identifier |
| `IsLeAudioActive` | `bool` | Indicates whether LE Audio is currently active |
| `VendorId` | `uint` | Bluetooth vendor identifier, if available |
| `ProductId` | `uint` | Bluetooth product identifier, if available |
| `VendorIdSource` | `uint` | Source authority for the vendor identifier |
| `IsRemoved` | `bool` | Indicates that the device was removed or unpaired rather than simply disconnected |

**Note on battery data**  
The `Batteries` field contains component-level battery information when available. For earbuds and similar devices, this typically includes battery data for the left earbud, right earbud, and charging case.

---

### 3.5 Bluetooth Radio State Changed

**Event name:** `BluetoothRadioStateChanged`

This event indicates that the state of the Bluetooth radio has changed.

Supported state values are:
- `On`
- `Off`
- `Disabled`

**Typical usage**
- Detect Bluetooth availability changes
- Update platform or application status indicators
- Explain downstream device disconnect behavior
- Support troubleshooting and diagnostics
- Available regardless of whether a Microsoft Teams session is active
- Applies to any Bluetooth type, including BLE

**Expected frequency**
- Emitted when the radio changes state
- Event-driven, not periodic

**Payload**

| Field | Type | Description |
|---|---|---|
| `TimestampMs` | `long` | Timestamp of the event in milliseconds since Unix epoch |
| `PreviousState` | `string` | Previous Bluetooth radio state |
| `NewState` | `string` | New Bluetooth radio state |

---

### 3.6 Bluetooth Classic Audio Statistics

**Event name:** `BluetoothClassicAudioStatistics`

This event exposes raw firmware telemetry counters for a single Event 402 reporting interval. It is published by the provider process every sampling period during an active call and carries the raw inputs used for Bluetooth audio quality analysis.

**Typical usage**
- Compute Bluetooth audio quality scores
- Monitor raw firmware counters during active calls
- Correlate telemetry inputs with quality outcomes
- Support diagnostics and analysis of classic Bluetooth audio behavior

**Expected frequency**
- Emitted every sampling period during an active call
- Event-driven, not periodic outside active call telemetry collection

**Payload**

| Field | Type | Description |
|---|---|---|
| `TimestampMs` | `long` | Unix ms when Event 402 was received (ETW header timestamp) |
| `ConnectionHandle` | `int` | SCO/eSCO connection handle from Event 402 TLV `0x6A` |
| `RxLostPackets` | `int` | Received payloads with errors in this interval (TLV `0x6D`) |
| `TxLostPackets` | `int` | Transmitted payloads lost or unacknowledged in this interval (TLV `0x6E`) |
| `SamplingPeriodMs` | `int` | Configured firmware reporting period sent over IOCTL (`1000..10000`, default `5000`) |

**Note**  
This event provides raw telemetry inputs only. Scores and status are computed and published on `BluetoothAudioCallQuality`.

---

## 4. POC Scope and Limitations

### Supported Scenarios
- **Microsoft Teams Calls**: Call lifecycle and call-quality telemetry events are generated during active Microsoft Teams voice calls
- **Classic Bluetooth Audio (SCO/eSCO)**: This POC targets classic Bluetooth headsets using Synchronous Connection-Oriented (SCO) or Enhanced Synchronous Connection-Oriented (eSCO) links for call-audio scenarios
- **Device and radio state coverage**: `BluetoothDeviceUpdated` and `BluetoothRadioStateChanged` are active regardless of Teams call state and apply across Bluetooth types, including BLE

### Not Supported in This Implementation
- **Bluetooth LE Audio**: LE Audio devices and profiles are not included in this initial POC
- **Non-Teams Calls**: Events are not generated for calls outside of Microsoft Teams (e.g., Skype, telephony, other applications)
- **Call Quality Issues**: While quality events support general call quality monitoring, this POC is specifically focused on Bluetooth link quality degradation and the resulting user experience impact

### Quality Degradation Scenarios
The telemetry is designed to identify periods where Bluetooth connection quality issues manifest as:
- Audio breakups (user reports "you are breaking up")
- Choppy or distorted voice quality
- Temporary loss of audio
- Complete inability to hear the remote party

---

## 5. Frequency and Consumption Model

From an integration perspective, the events can be grouped into two categories:

### Lifecycle or state-transition events
These are emitted when a state change occurs:
- `BluetoothAudioCallStarted`
- `BluetoothAudioCallStopped`
- `BluetoothDeviceUpdated`
- `BluetoothRadioStateChanged`

These are **event-driven** and occur only when the corresponding condition changes.

### Snapshot or telemetry events
These may occur repeatedly over time:
- `BluetoothAudioCallQuality`
- `BluetoothClassicAudioStatistics`

These are suited for:
- continuous monitoring
- telemetry dashboards
- diagnostics
- quality score input analysis

---

## 6. Integration Value for HP

The proof-of-concept interface is designed to give HP a practical starting point for Bluetooth telemetry integration.

### Benefits of the parsed ICA events
- Easier to consume than raw system events
- Stable event naming
- Clear mapping to Bluetooth scenarios of interest
- Reduced need for low-level Windows event expertise

### Benefits of the raw firmware telemetry inputs
- Exposes the counters used to compute call quality scores
- Keeps quality calculations aligned with the provider telemetry model
- Supports diagnostics and validation of quality outcomes
- Provides a stable input contract for the quality pipeline

---

## 7. Recommended Positioning for Customer Communication

A concise way to describe the interface to HP is:

> The ICA proof-of-concept exposes a set of application-ready Bluetooth telemetry events for Microsoft Teams calls using Classic Bluetooth audio headsets (SCO/eSCO). Events cover call start, call end, call quality, device changes, Bluetooth radio state, and the raw firmware counters used for quality scoring.

---

## 8. Qualification

This document describes the **proof-of-concept event interface** currently intended for HP evaluation.  
As with any proof-of-concept integration surface, event definitions and detailed payload semantics may be refined during productization or based on partner feedback.
