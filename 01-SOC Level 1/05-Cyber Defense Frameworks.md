# Cyber Defense Frameworks

## Introduction

This is a collection of my understanding of the rooms Pyramid of Pain, Cyber Kill Chain, and Unified Kill Chain from TryHackMe platform. I combined all three of them because the tasks there are straight forward but the steps to each of these methods are essential to detect a cyber attack.

## Pyramid of Pain

Pyramid of Pain is, as per my understanding, the level of pain it would have to the attackers to change when their advances are detected. And they are classified by levels and colors so that it would be easier to identify. 

Here are the levels:

### Hash Values - Trivial (Blue)

These are the easiest to change if they are detected because hash values of malicious files change if you change a little bit of the file. And you can do that easily. Like adding a hash file attached to the file. Or changing the variable of a code. 

### IP Addresses - Easy (Green)

These are pretty easy to change too. There are several tools and ways you can change the public ip addresses.

### Domain Names - Simple (Teal)

If the domain name is detected, you can buy another domain. Which sound simple if you have the means, but it is much more painful than changing IP addresses or hash values. 

### Network/Host Artifacts - Annoying (Yellow)

At this stage, if detected, the attacker will feel more annoyed and frustrated, because they would have to change their methodology, and tools, and artifacts.

### Tools - challenging (Yellow)

This stage is similar to Network/Host Artifacts stage. And the color that represents both stages are same too. At this stage, if detected, the attacker might have to write a different tool, or even change the target.

### Tactics, Techniques and Procedures (TTPs) - hardest (Red)

This is the hardest to change if detected. The attacker would be left with two choices, go back and learn more ways to not get detected, or give up and change the target. 

The structure is constructed in a way of a pyramid because the higher you detect in the pyramid, the more difficult it generally is for an attacker to change their behavior. (I might need to add some words here).

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
