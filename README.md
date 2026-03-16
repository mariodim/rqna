
# Example NGAP Trace for UE Registration and Session Setup

This repository contains an **example NGAP packet trace** collected from an **Open5GS-based experimental setup**. The trace is provided as a lightweight illustrative companion to the analytical results discussed in the paper.

The purpose of the file is to show a **real control-plane message sequence** associated with:

- **UE registration** on the N2 interface
- subsequent **session setup signaling** visible in NGAP

The trace can be used to extract an **observable proxy for the control-plane service time**, namely the delay between the **Registration Request** and the subsequent **PDU Session Resource Setup Request**.

---

# File included

`new_test_open5gs_private.pcapng`

---

# Anonymization

The original trace contained public IP addresses. For privacy and sharing purposes they were replaced with private addresses while preserving packet structure and timestamps.

In the anonymized trace:

- `10.0.0.1` = **AMF**
- `10.0.0.2` = **gNB**

The anonymization only affects addressing information and **does not alter message ordering or timing**.

---

# NGAP signaling sequence

A typical signaling sequence visible in the trace is:

1. `InitialUEMessage, Registration request`
2. `DownlinkNASTransport, Authentication request`
3. `UplinkNASTransport, Authentication response`
4. `DownlinkNASTransport, Security mode command`
5. `UplinkNASTransport`
6. `InitialContextSetupRequest`
7. `InitialContextSetupResponse`
8. `PDUSessionResourceSetupRequest`
9. `PDUSessionResourceSetupResponse`

This sequence illustrates the **5G control-plane setup procedure at the NGAP level**.

---

# Simplified signaling diagram

The sequence above can be visualized as follows:

```
UE           gNB (10.0.0.2)           AMF (10.0.0.1)
 |                |                        |
 |  NAS Reg Req   |                        |
 |--------------->|                        |
 |                | InitialUEMessage       |
 |                |----------------------->|
 |                |                        |
 |                |<-----------------------|
 |                |  Authentication Req    |
 |                |----------------------->|
 |                |  Authentication Resp   |
 |                |<-----------------------|
 |                |                        |
 |                |<-----------------------|
 |                | Security Mode Command  |
 |                |----------------------->|
 |                | Uplink NAS             |
 |                |----------------------->|
 |                |                        |
 |                |<-----------------------|
 |                | InitialContextSetupReq |
 |                |----------------------->|
 |                | InitialContextSetupRsp |
 |                |                        |
 |                |<-----------------------|
 |                | PDU Session Setup Req  |
 |                |----------------------->|
 |                | PDU Session Setup Resp |
 |                |                        |
```

---

# What is measured

The paper uses service-time statistics derived from a previous Open5GS measurement campaign.
This trace **does not replace that dataset**. Instead, it provides an **independent sanity check** showing that the observed control-plane timing is consistent with the order of magnitude used in the analytical model.

The trace can be used to measure the delay between:

- **Registration Request** (inside `InitialUEMessage`)
- **PDU Session Resource Setup Request**

This quantity is used as an **observable proxy for the control-plane service time**.

---

# Extracting the service-time proxy in Wireshark

## 1. Open the trace

Open

`open5gs_private.pcapng`

in **Wireshark**.

---

## 2. Apply an NGAP filter

Use the following display filter:

```
ip.addr == 10.0.0.1 && ip.addr == 10.0.0.2 && ngap
```

This restricts the view to NGAP packets exchanged between the AMF and the gNB.

---

## 3. Identify node directions

In this trace:

- `10.0.0.2 -> 10.0.0.1` = **gNB → AMF**
- `10.0.0.1 -> 10.0.0.2` = **AMF → gNB**

Example:

- `InitialUEMessage` is sent **gNB → AMF**
- `PDUSessionResourceSetupRequest` is sent **AMF → gNB**

---

## 4. Locate the start event

Find the packet labeled:

```
InitialUEMessage, Registration request
```

This packet marks the **start of the measured interval**.

---

## 5. Set the time reference

Right-click that packet and select:

```
Set Time Reference
```

Wireshark will reset its displayed time to **0.000 s**.

---

## 6. Locate the end event

Scroll forward until you find:

```
PDUSessionResourceSetupRequest
```

This packet marks the **end of the measured interval**.

---

## 7. Read the time difference

The value shown in the **Time column** for `PDUSessionResourceSetupRequest` corresponds to the extracted service-time proxy.

In the example trace the value is typically around:

**0.2 – 0.3 seconds**

which is consistent with the service-time parameters used in the analytical model.


---

# Important note

The quantity extracted from the trace should be interpreted as an **observable proxy for the control-plane service time**, not as the internal processing time of a single network function.
It captures the portion of the control-plane setup visible on the **N2 interface**.


