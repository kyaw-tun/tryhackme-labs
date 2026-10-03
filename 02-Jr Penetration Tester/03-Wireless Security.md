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

For this section, Wi-Fi versions over the years, security protocols that it has been using, Wi-Fi misconfiguration, and common Wi-Fi attacks, and how to protect the Wi-Fi network are covered.

### Different Versions of Wi-Fi

| Standard | Generation | Year | Frequency | Key Improvement |
| --- | --- | --- | --- | --- |
| `802.11` | - | 1997 | `2.4 GHz` | 2 Mbps |
| `802.11b` | - | 1999 | `2.4 GHz` | 11 Mbps |
| `802.11g` | - | 2003 | `2.4 GHz` | 54 Mbps |
| `802.11n` | Wi-Fi 4 | 2009 | `2.4 GHz`,`5 GHz` | 600 Mbps |
| `802.11ac` | Wi-Fi 5 `common` | 2013 | `5 GHz` | up to 6.9 Gbps |
| `802.11ax` | Wi-Fi 6/6E `common` | 2021 | `2.4 GHz`, `5 GHz`, `6 GHz` | up to 9.6 Gbps |
| `802.11be` | Wi-Fi 7 | 2024 | `2.4 GHz`, `5GHz`, `6 GHz` | up to 46 Gbps |

### Wi-Fi Security Protocols

There are 4 Wi-Fi security protocols. Two of them are old protocols that are deprecated now, and are not secure at all, and 2 common protocols that are widely used:

- WEP - This security protocol can be broken, and it is advised to not use this protocol even though some old routers still do.
- WPA - This protocol is deprecated, and very less secure than its successors.
- WPA2 - This is widely used protocol in both consumer and enterprise networks.
- WPA3 - This is a new protocol, it is present in modern mobile phones but for routers, they are still very less common. And it uses the strongest encryption of all.

### Common Wi-Fi Misconfiguration

- Using outdated protocol like WEP or WPA
- Using weak password
- Using unnecessary features like WPS
- Using default credentials

### Common Wi-Fi Attack concept

- Password attack - This means brute-forcing Wi-Fi network
- Rogue Access Point - Connecting an access point (router) that doesn't belong to the network intentionally or unintentionally
- Evil Twin Attack - Faking the SSID as a legitimate network and intercepting traffic and capturing credentials 
- Deauthentication Attacks - Forcing devices to disconnect from the network
- Traffic Interceptions - Capturing the traffic if the encryption is weak or improperly configured

### Protecting Wi-Fi Networks

- Using WPA2 or WPA3 with strong passphrases
- Turning off outdated protocols such as WEP
- Turning off WPS if not required
- Changing default administrative credentials
- Implementing network segmentation
- Regularly updating firmware on access points

## Bluetooth Security

There are two types of Bluetooth: classic Bluetooth and Bluetooth Low Energy (BLE). Classic Bluetooth is used in streaming audio and transferring files. And BLE is used in smart devices, like smart watches, fitness trackers, smart locks. 

### Common Bluetooth Security Risks

- Unnecessary Discoverability 
- Weak Pairing Mechanisms
- Unauthorized Pairing
- Bluejacking
- Bluesnarfing
- Bluebugging
- Lack of Device Updates

### Security Consideration for Bluetooth

Here are the recommendations:

1. Disable Bluetooth when not in use
2. Avoid leaving devices in discoverable mode.
3. Use modern Bluetooth versions.
4. Require user approval for pairing.
5. Avoid pairing in public places.
6. Remove unused paired devices.
7. Keep devices updated.
8. Monitor BLE advising data.
9. Avoid "just works" only devices in sensitive environments

## RFID and NFC Security

RFID (Radio frequency identification) is a technology that uses electromagnetic fields to identify and track tags attached to objects.
NFC (Near Field Communication) is a technology that enables two devices to exchange data when brought into close proximity. 

Here are their differences:

|     | RFID | NFC |
| --- | --- | --- |
| Frequency | 13.56 MHz | 125 kHz - 960 MHz |
| Range | Up to 5 cm | Up to 100+ m |
| Communication | Two-way | One-way (typical) |
| Common Uses | Contactless payments, Phone pairing, Transit cards | Access badges, Inventory tracking, Asset management |

### Common RFID and NFC Security Risks

- Eavesdropping
- Cloning
- Relay attacks
- Unauthorized scanning (skimming)
- Lost or stolen cards

### Security Considerations for RFID and NFC

- Use cards that support encryption
- Limit sensitive data stored on cards or use tokenization
- Immediately deactivate lost or stolen cards
- Use protective sleeves such as Faraday pouches to protect against unauthorized scanning
- Update access controls regularly
- Implement secondary biometric authentication where possible

## Other Wireless Security

In this modern environment, there are many other wireless technology devices apart from the ones mentioned above. They support smart devices, automation systems, and connected infrastructure.

The ones that are mentioned in this room are Zigbee, Z-Wave, LoRa (Long Range), cellular (e.g., LTE-M, NB-IoT) and infrared. I am not going to mention one by one. They all pose security risks in emerging wireless ecosystems. 

## Knowledge Test

### Matching the attack

| Attack Scenario | Wireless Technology |
| --- | --- |
| An attacker within range connects to a victim's phone without authorization and downloads their contacts and messages  | Bluetooth (Short Range Pairing) |
| An attacker sniffs the network during device pairing and recovers the encryption key, which was only protected by a well-known default value shared across all devices. | Zigbee (IoT Mesh Protocol) |
| Two attackers use relay devices to forward the communication between a victim’s contactless payment card and a shop’s payment terminal in real time. | NFC (13.56 MHz contactless) |
| An attacker captures the four-way handshake between a client and an access point, then runs an offline dictionary attack to recover the network password | Wi-Fi (802.11) |
| An attacker records a signal from a remote control and replays it to operate the target device from outside a window, without needing any authentication. | Infrared (Line of sight) |
| An attacker reads data from an employee’s access badge and writes it to a blank card, producing a working copy that opens the office door. | RFID (Tag Identification) |

## Conclusion

This room is very fun to do even if it's theory heavy. And I already knew most of the facts presented in the room. But i decided to write it because there were things I still had trouble differentiating about, like RFID and NFC. I have heard and was very familiar with NFC, but i have never used it hands on as not even my phone comes with it. And on top of that this room introduced RFID, and after learning them, it made me think they are the same thing. That is why i decided to learn a bit more so that i would be able to distinguish them.
