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

The seven stages:

1. Reconnaissance
2. Weaponization
3. Delivery
4. Exploitation
5. Installation
6. Command & Control
7. Actions on Objectives

Short explanation of how defenders can use the stages to understand where an attack is occurring and where it can potentially be disrupted.

## Unified Kill Chain

Brief explanation of why the Unified Kill Chain exists and how it expands the traditional Kill Chain into a more detailed model.

[List the 18 phases here, without explaining every phase.]

## Comparing the Frameworks

| Framework | Main Purpose |
|---|---|
| Pyramid of Pain | Understand the value of different indicators and detection levels |
| Cyber Kill Chain | Describe an attack as a sequence of stages |
| Unified Kill Chain | Provide a more detailed model of adversary activity |

## My Experience

I was already familiar with many of these concepts from my previous cybersecurity studies. When completing these rooms, I relied heavily on pattern recognition and knowledge accumulated from studying networking, security concepts, and attack techniques.

The main value of these rooms for me was putting these concepts into established defensive frameworks rather than learning each concept individually.

## Conclusion

These frameworks provide different perspectives on the same problem: understanding adversary activity and identifying opportunities for detection and disruption.
