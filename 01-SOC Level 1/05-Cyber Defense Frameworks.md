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

The Cyber Kill Chain provides a high-level view of how an attack can progress, from the attacker gathering information about a target to eventually achieving their objective. For defenders, the value is not necessarily in predicting exactly what an attacker will do next, but in understanding where an attack is taking place and identifying opportunities to detect or disrupt it.

When I started learning cybersecurity about six months ago and first encountered the Cyber Kill Chain, I didn't think much of it. I just learned it as a series of steps that attackers take to infiltrate a system or organization. The seven stages were also fairly easy to memorize, so at the time I didn't think there was much more to it.

Later, when I encountered MITRE ATT&CK, I found the number of tactics and techniques much more overwhelming. I remember wondering why the Cyber Kill Chain wasn't enough. If an attack could be represented by seven stages, why would we need a framework with so many more?

After several months of TryHackMe and working through different CTF rooms, I understand the distinction much better.

The Cyber Kill Chain is useful as a high-level overview of an attack. It gives defenders a way to understand the general progression of an intrusion and think about where it could be detected or disrupted.

But it doesn't describe all the different ways an attacker can operate within those stages. Real attacks aren't necessarily neat, linear processes where an attacker moves through seven steps one after another. Attackers can use different techniques, repeat activities, skip stages, or perform multiple activities at once.

That's where more detailed frameworks such as MITRE ATT&CK become useful. I now see the Cyber Kill Chain less as a complete description of an attack and more as a big-picture map that helps put the more detailed techniques into context.

## Unified Kill Chain

The Unified Kill Chain expands on the traditional Cyber Kill Chain by breaking an attack into more detailed stages. While the Cyber Kill Chain provides a high-level overview of how an attack progresses, the Unified Kill Chain adds more steps to provide a more complete view of the different activities an attacker may perform during an intrusion.

1. Reconnaissance
2. Weaponization
3. Delivery
4. Social Engineering
5. Exploitation
6. Persistence
7. Defense Evasion
8. Command & Control
9. Pivoting
10. Discovery
11. Privilege Escalation
12. Execution
13. Credential Access
14. Lateral Movement
15. Collection
16. Exfiltration
17. Impact
18. Objectives

These steps are grouped into three broader sections: In, Through, and Out.

## In — Initial Foothold

The first nine steps are classified as "In", referring to the attacker gaining an initial foothold in the target environment. During this stage, the attacker gathers information, prepares their attack, attempts to gain access, and establishes the access needed to continue the attack.

## Through — Network Propagation

Steps 9 through 14 are classified as "Through", referring to network propagation. At this stage, the attacker attempts to move through the environment, discover other systems, obtain higher privileges, execute actions, access credentials, and move laterally across the network.

The exact activities depend on what the attacker is trying to accomplish and what access they have managed to obtain.

## Out — Action on Objectives

The final four steps are classified as "Out", referring to the attacker's actions against their objectives. At this point, the attacker may collect and exfiltrate sensitive information, cause an impact to the organization, or carry out whatever objective they originally intended.

For example, an attacker might corrupt an organization's data, steal confidential files and sell them, or use stolen information to blackmail the organization. In a ransomware attack, their objective could be to encrypt data and demand payment in exchange for restoring access.

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
