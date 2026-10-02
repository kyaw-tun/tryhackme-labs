# Wireless Security

This write-up covers my understanding of this room and the key concepts I took away from it.

> **TryHackMe Room:** [Wireless Security](https://tryhackme.com/room/wirelesssecurity)

## Introduction

In the modern world, almost everyone has WiFi or at least mobile data. It makes convenient to the user who is using it because in the past, before mobile phones, internet meant "ethernet" cables, and wired connection. And now, that convenience comes with weaknesses too, and they are discussed below.  

## Wi-Fi Network Fundamentals

The two key components of Wi-Fi network are:

- Access Points (AP) - The device that broadcasts the signal, or that connects the phone, computer to the wired network: e.g. router
- Clients - These are the devices that use the internet: e.g. phone, computers, printers, smart devices, etc

### Identifiers in the Wi-Fi Network

- SSID (Service Set Identifier) - The name of the Wi-Fi network that you connect to or see when you browse available networks, e.g. `home_wifi`.
- BSSID (Basic Service Set Identifier) - The unique identifier of the Access Point (router) represented by the MAC address, e.g. `00:1A:2B:3C:4D:5E`.

### How Wireless Devices Communicate

The wireless devices communicate by sharing radio waves over a shared network airspace. And network devices communicate over different frequency bands. The most common ones are 2.4 GHz and 5 GHz. Think of it as roads, and the communication and the transfer of data as cars, motorcycles, and pedestrians.
2.4 GHz represents normal roads, where cars, motorcycles, pedestrians, everyone goes. So, it's congested, and there are more traffic there. 5 GHz represents highway roads, or toll roads, where only certain vehicles are allowed, and there are almost no traffic at all.

| 2.4 GHz | 5 GHz |
| --- | --- |
| Longer range (45 meters indoors) | Shorter range (15 meters indoors) |
| Can push through walls and floors easily | Struggles with obstacles |
| Only 3 non-overlapping channels (1,6,11) so congested | 23+ non overlapping channels significantly reducing congestion |
| More prone to interference from Bluetooth, Microwaves and other Wi-Fi | Less interference from other devices due to less crowded spectrum |
| Lower maximum throughput (600 Mbps) | Higher maximum throughput and speed (1300 Mbps) |
| Better for IoT devices and distant connection | Better for high bandwidth streaming, gaming, and video calls |

### Association Process

When a device (client) tries to connect to an AP, these processes happen:

1. Scan - Device scans for available networks
2. Select & Associate - Device select SSID and sends association request
3. Authenticate - Identify verification and between client and AP
4. Key Exchange - Encryption keys are established
5. Transit - Secure data transmission begins

## Wi-Fi Security

## Bluetooth Security

## RFID and NFC Security

## Other Wireless Technologies

## Test (maybe, don't include it?)

## Conclusion
