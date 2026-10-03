# Wireless Security

This write-up covers my understanding of this room and the key concepts I took away from it.

> **TryHackMe Room:** [Wireless Security](https://tryhackme.com/room/wirelesssecurity)

## Introduction

In the modern world, almost everyone has Wi-Fi or at least mobile data. It makes things more convenient for users because in the past, before mobile phones and wireless connections became common, internet access usually meant using an Ethernet cable or another wired connection. However, that convenience also comes with weaknesses, which are discussed below. 

## Wi-Fi Network Fundamentals

The two key components of Wi-Fi network are:

- **Access Points (AP)** -  The device that broadcasts the signal or connects phones and computers to the wired network, such as a router.
- **Clients** - These are the devices that connect to the network, such as phones, computers, printers, and other smart devices.

### Identifiers in the Wi-Fi Network

- **SSID (Service Set Identifier)** - The name of the Wi-Fi network that you connect to or see when browsing available networks, e.g. `home_wifi`.
- **BSSID (Basic Service Set Identifier)** - The unique identifier of an access point, usually represented by its MAC address, e.g. `00:1A:2B:3C:4D:5E`.

### How Wireless Devices Communicate

Wireless devices communicate by sharing radio waves over the air. Network devices communicate over different frequency bands, with 2.4 GHz and 5 GHz being two of the most common ones.

I like to think of them as roads, with communication and data transfer being the cars, motorcycles, and pedestrians.

2.4 GHz is like a normal road where everyone goes, so it can become congested and has more interference. 5 GHz is more like a highway, where there is generally less traffic and more room for communication.

| 2.4 GHz | 5 GHz |
| --- | --- |
| Longer range | Shorter range |
| Can push through walls and floors easily | Struggles more with obstacles |
| Only 3 non-overlapping channels (1,6,11) | 23+ non overlapping channels significantly reducing congestion |
| More prone to interference from Bluetooth, Microwaves and other Wi-Fi | Less interference from other devices due to less crowded spectrum |
| Lower maximum throughput (600 Mbps) | Higher maximum throughput and speed (1300 Mbps) |
| Better for IoT devices and distant connection | Better for high bandwidth streaming, gaming, and video calls |

### Association Process

When a device (client) tries to connect to an AP, these processes happen:

1. Scan - The device scans for available networks.
2. Select & Associate - The device selects an SSID and sends an association request.
3. Authenticate - Authentication takes place between the client and AP.
4. Key Exchange - Encryption keys are established.
5. Transit - Secure data transmission begins.

## Wi-Fi Security

For this section, the different Wi-Fi versions over the years, security protocols, common Wi-Fi misconfigurations, common Wi-Fi attacks, and ways to protect a Wi-Fi network are covered.

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

There are four major Wi-Fi security protocols covered here. Two of them are old protocols that are deprecated and no longer considered secure, while WPA2 and WPA3 are the commonly used modern protocols:

- WEP - This security protocol can be broken easily, and it is advised not to use it, even though some old routers may still support it.
- WPA - This protocol is deprecated and is much less secure than its successors.
- WPA2 - This is a widely used protocol in both consumer and enterprise networks.
- WPA3 - This is a newer recommended protocol that provides stronger security than previous generations and is supported by many modern devices.

### Common Wi-Fi Misconfiguration

- Using outdated protocol such as WEP or WPA
- Using a weak password
- Using unnecessary features such as WPS
- Using default credentials

### Common Wi-Fi Attack concept

- Password attack - This can involve attempting to recover a Wi-Fi password through methods such as offline dictionary attacks.
- Rogue Access Point - An unauthorized access point connected to or operating within a network.
- Evil Twin Attack - Faking the SSID of a legitimate network to trick users into connecting and potentially capturing credentials or traffic.
- Deauthentication Attacks - Forcing devices to disconnect from the network.
- Traffic Interceptions - Capturing traffic when encryption is weak, missing, or improperly configured.

### Protecting Wi-Fi Networks

- Use WPA2 or WPA3 with strong passphrases.
- Turn off outdated protocols such as WEP.
- Turn off WPS if not required.
- Change default administrative credentials.
- Implement network segmentation.
- Regularly update firmware on access points.

## Bluetooth Security

There are two main types of Bluetooth: Classic Bluetooth and Bluetooth Low Energy (BLE). Classic Bluetooth is commonly used for things such as streaming audio and transferring files, while BLE is used in smart devices such as smart watches, fitness trackers, and smart locks. 

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

1. Disable Bluetooth when not in use.
2. Avoid leaving devices in discoverable mode.
3. Use modern Bluetooth versions.
4. Require user approval for pairing.
5. Avoid pairing in public places.
6. Remove unused paired devices.
7. Keep devices updated.
8. Monitor BLE advertising data.
9. Avoid relying on "Just Works" pairing for sensitive environments when stronger authentication is available.

## RFID and NFC Security

RFID (Radio Frequency Identification) is a technology that uses electromagnetic fields to identify and track tags attached to objects.

NFC (Near Field Communication) is a short-range wireless technology that enables devices or tags to exchange data when brought into close proximity.

Here are their differences:

|     | NFC | RFID |
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
- Limit sensitive data stored on cards or use tokenisation
- Immediately deactivate lost or stolen cards
- Use protective sleeves such as Faraday pouches to protect against unauthorised scanning
- Update access controls regularly
- Implement secondary biometric authentication where possible

## Other Wireless Security

In this modern environment, there are many other wireless technologies apart from the ones mentioned above. They support smart devices, automation systems, and connected infrastructure.

The ones mentioned in this room are Zigbee, Z-Wave, LoRa (Long Range), cellular technologies (e.g., LTE-M, NB-IoT), and infrared. I am not going to go through them one by one. They all pose different security risks in emerging wireless ecosystems.

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

This room was very fun to do even though it was theory-heavy. I already knew most of the facts presented in the room, but I decided to write about it because there were still things I had trouble differentiating, especially RFID and NFC.

I had heard of and was very familiar with NFC, but I had never used it hands-on, as even my phone does not come with it. On top of that, this room introduced RFID, and after learning about both, it initially made me think they were basically the same thing.

That is why I decided to learn a bit more about them so that I could properly distinguish between the two. This was probably the most useful part of the room for me, because it helped me understand that although RFID and NFC are related wireless technologies, they are not the same thing.
