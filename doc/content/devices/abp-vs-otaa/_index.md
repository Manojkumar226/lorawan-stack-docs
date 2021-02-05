---
title: "ABP vs OTAA"
description: ""
---

This section can help you understand the differences between ABP and OTAA activation modes for LORaWAN end devices, and comprehend why using OTAA is recommended. 

<!--more-->

A `DevEUI` is a 64-bit unique ID assigned to an end device by the manufacturer. This value is linked to the hardware and it cannot be altered.

Unlike `DevEUI`, which identifies an end device globally, a 32-bit `DevAddr` identifies the end device within the current network and all communication after joining the network is done with it. A `DevAddr` value consists of `NwkID` and `NwkAddr`. `NwkID` is a 7-bit network identifier, used to distinguish (overlapping) LoRaWAN networks, while `NwkAddr` is a 25-bit network address of the end device.

{{< note >}} `DevAddr` are not unique identifiers. If there are more than one device with identical `DevAddr`, {{% tts %}} Network Server has to perform **matching**. See more in the [Network Server reference]({{< ref "/reference/components/network-server#uplink-handling" >}}) section. {{</ note >}}

A `DevAddr` and security keys are assigned to an end device during a procedure called **activation**. LoRaWAN supports two modes of activating an end device:

- ABP (Activation By Personalization)
- OTAA (Over-The-Air Activation)

## ABP

In ABP activation, a **fixed** `DevAddr` and **session keys** for a pre-selected network are hardcoded into the end device, and they remain the same throughout the lifetime of an ABP end device. With this mode, an end device skips the join procedure which seems to be simpler, but there are quite a few limitations of using ABP, which are explained below. 

## OTAA

OTAA end devices are provisioned with **root keys**. In OTAA activation, an end device performs a join procedure with a LoRaWAN network, during which a **dynamic** `DevAddr` is assigned to an end device, and root keys are utilized to derive **session keys**. Hence, `DevAddr` and session keys change as each new session is established.

## Why is OTAA better than ABP?

ABP's drawbacks are arising as consequences of its main characteristics.

1. **ABP end devices use a fixed `DevAddr`.**

  Taking the structure of `DevAddr` into the account and the fact that, with ABP, `NwkID` is fixed, we come to the conclusion that this device can work correctly only in its predefined network. Even if a network or a cluster allows registering end devices with different `DevAddr` values than the ones that were assigned to that network/cluster, the [Packet Broker]({{< ref "/reference/peering#packet-broker" >}}) will not route the traffic from those end devices to the right network/cluster. 

  Also, even if the network/cluster is assigned with additional address blocks, the Network Server will not be able to perform optimization of `DevAddr` allocation, hence the device matching procedure for ABP end devices will not be improved.

  OTAA end devices are assigned a new `DevAddr` at establishing each new session.

2. **ABP end devices use a fixed security session.**

  ABP end devices must not reset frame counters during their lifetime, otherwise the messages from those end devices may end up being dropped. Not being able to reset frame counters means that the end of life for the ABP end device is when it runs out of frame counters.

  Session keys are hardcoded into an ABP device, meaning that even if they leak, they cannot be rotated, which is a serious security threat.

  OTAA end devices re-negotiate frame counters and session keys at establishing each new session.

3. **ABP end devices use fixed network parameters.**

  When registering an ABP device on {{% tts %}}, you will need to provide the **RX1 Delay**, **RX1 Data Rate Offset**, **RX2 Data Rate Index**, **RX2 Frequency** and a list of **Factory Preset Frequencies**. If these values are not correctly configured, uplinks and/or downlinks might not work.

  OTAA end devices re-negotiate network parameters at establishing each new session.
