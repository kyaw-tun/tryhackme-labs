# Introduction to Phishing

## Overview

This was a scenario-based SOC exercise rather than a traditional walkthrough or CTF.

Instead of following a set of instructions to exploit a machine, I was given a series of security alerts and had to determine whether they were true positives or false positives.

The objective was to identify all of the true-positive alerts based on the evidence provided.

## Objective

- Review the alerts presented during the scenario.

- Distinguish true positives from legitimate activity.

- Explain the reasoning behind each classification.


## Alert Triage 

## Alert 1 - Inbound Email Containing Suspicious External Link 

- Severity: Medium

- sender: `onboarding@hrconnex.thm`

- recipient: `j.garcia@thetrydaily.thm`

- Content:

```email
Hi Ms. Garcia,\n\nWelcome to TheTryDaily!\n\nAs part of your onboarding, please complete your final profile setup so we can configure your access.\n\nKindly please click the link below:\n\n<a href="https://hrconnex.thm/onboarding/15400654060/j.garcia">Set Up My Profile</a>.\n\nIf you have questions, please reach out to the HR Onboarding Team.
```

## Analysis

At first glance, this alert could look suspicious simply because it contains an external link. However, looking at the actual content, there were no obvious phishing indicators.

The email was related to employee onboarding, the sender appeared to belong to the same organization, and the request itself was reasonable for a new employee.

There was also no obvious urgency, threat, impersonation, or suspicious-looking destination.

Classification: False Positive

This was a good reminder that an alert being triggered does not automatically mean that malicious activity has occurred. The alert still needs to be investigated and judged based on the available evidence.

## Alert 2 - Inbound Email Containing Suspicious External Link 

- Severity: Medium 

- sender: `urgents@amazon.biz`

- recipient: `h.harris@thetrydaily.thm`

- Content:

```email
Dear Customer,\n\nWe were unable to deliver your package due to an incomplete address.\n\nPlease confirm your shipping information by clicking the link below:\n\n`http://bit.ly/3sHkX3da12340`\n\nIf we don’t hear from you within 48 hours, your package will be returned to sender.\n\nThank you,\n\nAmazon Delivery
```

## Analysis

This alert contained several indicators that made the email suspicious.

First, the sender address did not look like an official Amazon address. The email also used a shortened URL rather than directing the recipient to Amazon's legitimate website.

The message was asking the recipient to provide or confirm shipping information through that link. It also used urgency by giving the recipient only 48 hours before the package would supposedly be returned.

These are common characteristics of phishing emails:

- Suspicious sender address

- Link that does not clearly lead to the legitimate organization

- Request for information

- Urgency and time pressure

- Impersonation of a trusted organization

Classification: True Positive

## Alert 3 - Access to Blacklisted External URL Blocked by Firewall

- Severity: High

- SourceIP: `10.20.2.17`

- DestinationIP: `67.199.248.11`

- URL: `http://bit.ly/3sHkX3da12340`

## Analysis

This alert was particularly interesting because it connected to the same URL seen in the previous phishing alert.

An internal host attempted to access an external URL that was already listed in the organization's blacklist or threat-intelligence feeds. The firewall successfully blocked the request.

The fact that the firewall blocked the connection does not make the event harmless. The important point is that an internal system attempted to communicate with a destination that was already known to be suspicious.

This could indicate that someone inside the organization interacted with the phishing email, or that a device had some other reason for attempting to reach the malicious destination.

The alert therefore warranted investigation.

Classification: True Positive

## Alert 4 - Inbound Email Containing Suspicious External Link 

- Severity: Medium

- Sender: `no-reply@m1crosoftsupport.co`

- recipient: `c.allen@thetrydaily.thm`

- content:

```email
Hi C.Allen,\n\nWe detected an unusual sign-in attempt on your Microsoft account.\n\nLocation: Lagos, Nigeria\n\nIP Address: 102.89.222.143\n\nDate: 2025-01-24 06:42\n\nIf this was not you, please secure your account immediately to avoid unauthorized access.\n\n<a href="https://m1crosoftsupport.co/login">Review Activity</a>\n\nThank you,\n\nMicrosoft Account Security Team
```

## Analysis

This email contained several strong phishing indicators.

The sender address was designed to look like Microsoft, but used `m1crosoft` instead of `microsoft`. The same technique appeared in the link destination.

The message also claimed that there had been an unusual login attempt and encouraged the recipient to act immediately. This combination of account-security impersonation and urgency is a common phishing technique.

The email was therefore suspicious even before considering the destination of the link.

Classification: True Positive

## Alert 5 - Inbound Email Containing Suspicious External Link 

- Severity: Medium

- sender: `onboarding@hrconnex.thm`

- recipient: `j.garcia@thetrydaily.thm`

- content: 

```email
Hi Ms. Garcia,\n\nWelcome to TheTryDaily!\n\nAs part of your onboarding, please complete your final profile setup so we can configure your access.\n\nKindly click the link below:\n\n<a href="https://hrconnex.thm/onboarding/15400654060/j.garcia">Set Up My Profile</a>.\n\nIf you have questions, please reach out to the HR Onboarding Team.
```

## Analysis

I did not independently reach this alert during my original run of the scenario. The scenario ended after I successfully identified the three required true-positive alerts.

I later booted up the scenario again to check Alert 5. It is the same alert as Alert 1, with the same details, so I classified it as a false positive.

Classification: False Positive

## Conclusion

When looking at these alerts, I mainly focused on the sender, the links, the context of the email, and whether there were any signs of urgency or impersonation. I also learned that just because an alert looks suspicious doesn't mean it's a true positive. I still need to look at the whole thing before making a decision.

I first started the SOC Level 1 path roughly 120 days ago. When I did this room back then, I didn't get all the answers right and wasn't always sure why an alert was a true or false positive.

Today, I went through the same scenario again and got all of them right. More importantly, I felt much more confident about why I was making each decision.

It's a simple thing, but it was nice to see that I can now recognize things that I struggled with when I first started.

### Key Takeaways

- Don't assume every alert is malicious.

- Look at the sender, links, and context together.

- Understand why an alert is a true or false positive before making a decision.

- Practice makes these decisions easier over time.
