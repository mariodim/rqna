# Example NGAP Trace for UE Registration and Session Setup

This repository contains an **example NGAP packet trace** collected from an **Open5GS-based experimental setup**. The trace is provided as a lightweight illustrative companion to the analytical results discussed in the paper on the application of the Robust Queueing framework to 5G service chains.

The purpose of the file is to show a **real control-plane message sequence** associated with:

- **UE registration** on the N2 interface  
- subsequent **session setup signaling** visible in NGAP  

The trace can be used to extract an **observable proxy for the control-plane service time**, namely the delay between the **Registration Request** and the subsequent **PDU Session Resource Setup Request**.

---

# System setup and interpretation

In this trace, communication occurs between:

- a **gNB** (Next Generation Node B), i.e., the 5G base station to which the User Equipment (UE) attaches  
- the **core network**, represented by a single IP address  

In the Open5GS deployment used to generate this trace, the core network functions:

- **AMF (Access and Mobility Management Function)**  
- **SMF (Session Management Function)**  
- **UPF (User Plane Function)**  

are **co-located on the same host and share the same IP address**.

As a result:

- the NGAP trace exposes only **gNB ↔ AMF signaling**
- internal interactions involving SMF and UPF are **not directly visible at the packet level**

This is consistent with the modeling assumptions used in the paper, where:

- the **AMF represents the ingress node of the service chain**
- SMF and UPF are traversed subsequently, but their processing is not directly observable from NGAP traces

Importantly, the **AMF is the dominant control-plane entity during the registration procedure**, making it the most relevant observation point for timing analysis.

---

# Files included

`open5gs_private.pcapng`  
`open5gs_private2.pcapng`

---

# Anonymization

The original trace contained public IP addresses. For privacy and sharing purposes they were replaced with private addresses while preserving packet structure and timestamps.

In the anonymized trace:

- `10.0.0.1` = **Core network (AMF/SMF/UPF co-located)**  
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

This sequence illustrates the **5G control-plane setup procedure at the NGAP level**, primarily handled by the AMF, with subsequent involvement of SMF and UPF at the core-network level.

---

# Simplified signaling diagram

The sequence above can be visualized as follows:


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
# Open5GS Test environment 

The test environment scheme includes UERANSIM (UE and gNB emulator) and Open5GS (AMF, SMF, UPF emulator). The two platforms have been installed on two VMs (Linux Ubuntu 20.04), both equipped with 1 vCPU (Intel Xeon E5- 1620 v2 @ 3.70 GHz); The RAM amounts to 10 GB for Open5GS VM and to 2.21 GB for UERANSIM VM. AMF, SMF, and UPF nodes run as native Linux processes.

# Important note

The quantity extracted from the trace should be interpreted as an **observable proxy for the control-plane service time**, not as the internal processing time of a single network function.
It captures the portion of the control-plane setup visible on the **N2 interface**.


