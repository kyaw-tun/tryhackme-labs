# Cyber Defense Frameworks

## Introduction

This is a collection of my understanding of the Pyramid of Pain, Cyber Kill Chain, and Unified Kill Chain rooms from the TryHackMe platform. I combined all three of them because, while the tasks in these rooms are fairly straightforward, the concepts and steps behind each framework are important for understanding how cyber attacks can be detected and disrupted.

Each framework looks at an attack from a different perspective. The Pyramid of Pain focuses on how difficult different indicators are for an attacker to change, while the Cyber Kill Chain and Unified Kill Chain provide different ways of looking at the stages and activities involved in an attack.

This write-up focuses on what I understood from completing the rooms and how I see these frameworks fitting into a broader defensive security approach.

## TryHackMe Rooms:

- [Pyramid of Pain](https://tryhackme.com/room/pyramidofpainax)
    
- [Cyber Kill Chain](https://tryhackme.com/room/cyberkillchainzmt)

- [Unified Kill Chain](https://tryhackme.com/room/unifiedkillchain) 

This write-up covers my understanding of these three rooms and the key concepts I took away from them.

## Pyramid of Pain

Pyramid of Pain is, as per my understanding, the level of pain it would cause an attacker to change their approach when their advances are detected. They are classified by different levels and colors, which makes it easier to identify how difficult it would be for an attacker to change each part.

Here are the levels:

### Hash Values - Trivial (Blue)

These are the easiest to change if they are detected because the hash value of a malicious file changes if you modify the file even slightly. For example, if there is a malicious `.exe` file and you append some extra data to it, such as a text file or other content, the hash value of the `.exe` will be completely different even though the actual functionality of the file might not have changed. This makes hash values very easy for an attacker to change.

### IP Addresses - Easy (Green)

These are pretty easy to change too. There are several tools and ways an attacker can change or use different public IP addresses, making this another relatively easy indicator to replace.

### Domain Names - Simple (Teal)

If the domain name is detected, an attacker can buy or use another domain. This sounds simple if they have the means to do it, but it is still more painful than changing IP addresses or hash values because there is more effort involved in replacing the domain and continuing the attack.

### Network/Host Artifacts - Annoying (Yellow)

At this stage, if the artifacts are detected, the attacker will feel more annoyed and frustrated because they would have to change their methodology, tools, or other artifacts to avoid being detected again.

### Tools - challenging (Yellow)

This stage is similar to the Network/Host Artifacts stage, and the color that represents both stages is the same too. At this stage, if the attacker's tools are detected, they might have to modify or write a different tool, or even change their target.

### Tactics, Techniques and Procedures (TTPs) - hardest (Red)

This is the hardest level to change if detected. The attacker would be left with two choices: go back and learn more ways to avoid detection, or give up and change their target.

The structure is constructed as a pyramid because the higher you go in the pyramid, the more difficult it generally is for an attacker to change their behavior. Detecting something higher up the pyramid can therefore cause more disruption to the attacker because it can force them to change the way they operate, rather than simply replacing one indicator.

## Cyber Kill Chain

The Cyber Kill Chain is a model developed by Lockheed Martin that describes a cyber attack as a sequence of stages. It provides defenders with a high-level view of an attacker's progression, from the initial preparation and targeting of a victim to the eventual achievement of their objective.

The seven stages:

1. Reconnaissance
2. Weaponization
3. Delivery
4. Exploitation
5. Installation
6. Command & Control
7. Actions on Objectives

The idea is not necessarily that every real-world attack will follow these stages perfectly or in exactly this order. Rather, the model provides a way to understand an attacker's progression and identify opportunities where defenders may be able to detect, prevent, or disrupt the attack.

When I started doing cyber security about six months ago and encountered Cyber Kill Chain, I didn't think much of it. I just learned it as steps that attackers take to infiltrate a system or organization. And I learned these stages by heart (it is pretty easy). And MITRE ATT&CK was a subsequent lesson, and the steps on that would be very overwhelming. And i would think why shouldn't Cyber Kill Chain be enough, why would attackers need more steps (as if attackers follow these steps and not the other way around).

But now, after 4 months of TryHackMe and multiple CTF rooms, and redoing it again, I now understand that it isn't enough. It merely covers the basics or just an overview of the steps. But it is helpful to know nonetheless, it gives you a glimpse on where the attacker is on his path and what is his next step is going to be.

## Unified Kill Chain

...


## Comparing the Frameworks

| Framework | Main Purpose |
|---|---|
| Pyramid of Pain | Understand the value of different indicators and detection levels |
| Cyber Kill Chain | Describe an attack as a sequence of stages |
| Unified Kill Chain | Provide a more detailed model of adversary activity |

## My Experience

...

## Conclusion

...
