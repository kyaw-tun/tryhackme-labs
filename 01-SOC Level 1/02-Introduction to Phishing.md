# Introduction to Phishing

## Introduction to the room, (like a real world scenario style)

## Objective

The objective of this room was to identify all the true positive alerts.

## Scenario 

### Alert 1 - Inbound Email Containing Suspicious External Link 

- Severity: Medium
- Description: This alert was triggered by an inbound email contains one or more external links due to potentially suspicious characteristics. As part of the investigation, check firewall or proxy logs to determine whether any endpoints have attempted to access the URLs in the email and whether those connections were allowed or blocked.
- sender: onboarding@hrconnex.thm
- recipient: j.garcia@thetrydaily.thm
- content: Hi Ms. Garcia,\n\nWelcome to TheTryDaily!\n\nAs part of your onboarding, please complete your final profile setup so we can configure your access.\n\nKindly please click the link below:\n\n<a href="https://hrconnex.thm/onboarding/15400654060/j.garcia">Set Up My Profile</a>.\n\nIf you have questions, please reach out to the HR Onboarding Team.

This actually looks like a normal onboarding email. And there is no suspicious signs to this email. So, it is identified as a false positive.

### Alert 2 - Inbound Email Containing Suspicious External Link 

- Severity: Medium 
- Description: This alert was triggered by an inbound email contains one or more external links due to potentially suspicious characteristics. As part of the investigation, check firewall or proxy logs to determine whether any endpoints have attempted to access the URLs in the email and whether those connections were allowed or blocked.
- sender: urgents@amazon.biz
- recipient: h.harris@thetrydaily.thm
- content: Dear Customer,\n\nWe were unable to deliver your package due to an incomplete address.\n\nPlease confirm your shipping information by clicking the link below:\n\nhttp://bit.ly/3sHkX3da12340\n\nIf we don’t hear from you within 48 hours, your package will be returned to sender.\n\nThank you,\n\nAmazon Delivery

The content of the email looks very suspicious because on the first point, they are asking for the shipping information that should be given to them by amazon if they are legitimate. On the second point, they are asking the recipient to give the information on a suspicious looking website, not the actual amazon website. And the tone and the words sound very urgent which is a common phishing tactics. And the sender's email look suspicious too. So, it is identified as true positive.

### Alert 3 - Access to Blacklisted External URL Blocked by Firewall

- Severity: High
- Description: This alert was triggered when a user attempted to access an external URL that is listed in the organization's blacklist or threat intelligence feeds. The firewall or proxy successfully blocked the outbound request, preventing the connection. Note: The blacklist only covers known threats. It does not guarantee protection against new or unknown malicious domains.
- SourceIP: 10.20.2.17
- DestinationIP: 67.199.248.11
- URL: http://bit.ly/3sHkX3da12340

This alert is true positive because an ip from within the firewall accessed to a blacklisted external URL. This activity is suspicious and it might mean an attacker get inside the firewall or a person or a device inside the firewall is contacting to that external URL. Therefore, it is identified as true positive.

### Alert 4 - Inbound Email Containing Suspicious External Link 

- Severity: Medium
- Description: This alert was triggered by an inbound email contains one or more external links due to potentially suspicious characteristics. As part of the investigation, check firewall or proxy logs to determine whether any endpoints have attempted to access the URLs in the email and whether those connections were allowed or blocked.
- Sender: no-reply@m1crosoftsupport.co
- recipient: c.allen@thetrydaily.thm
- content: Hi C.Allen,\n\nWe detected an unusual sign-in attempt on your Microsoft account.\n\nLocation: Lagos, Nigeria\n\nIP Address: 102.89.222.143\n\nDate: 2025-01-24 06:42\n\nIf this was not you, please secure your account immediately to avoid unauthorized access.\n\n<a href="https://m1crosoftsupport.co/login">Review Activity</a>\n\nThank you,\n\nMicrosoft Account Security Team

The content in the email seems suspicious and it matches several phishing email characteristics. First, the sender's email address is not a real "microsoft" email. It just looks like one. Second, the login URL they send looks fake too, same "m1crosoft". And then they are urging the recipient to act quickly which is a common phishing tactic. Therefore, it is identified as true positive.

### Alert 5 - Inbound Email Containing Suspicious External Link 

- Severity: Medium
- Description: This alert was triggered by an inbound email contains one or more external links due to potentially suspicious characteristics. As part of the investigation, check firewall or proxy logs to determine whether any endpoints have attempted to access the URLs in the email and whether those connections were allowed or blocked.
- sender: onboarding@hrconnex.thm
- recipient: j.garcia@thetrydaily.thm
- content: Hi Ms. Garcia,\n\nWelcome to TheTryDaily!\n\nAs part of your onboarding, please complete your final profile setup so we can configure your access.\n\nKindly click the link below:\n\n<a href="https://hrconnex.thm/onboarding/15400654060/j.garcia">Set Up My Profile</a>.\n\nIf you have questions, please reach out to the HR Onboarding Team.

This one, I did not get to do because the scenario only tells us to successfully identify all true positives and the scenario existed after getting the three true positives. And from what it seems, it looks exactly like the first alert. So, it is a normal onboarding email from an organization and is identified as false positive.

## Conclusion

I first started the SOC Level 1 path roughly 120 days ago (May 2026). When I previously encountered this type of alert-triage exercise, I struggled to confidently identify the true-positive alerts. Even when I made a correct classification, I wasn't confident that my reasoning was correct.

Revisiting the scenario today was a different experience. I was able to assess each alert independently, identify the initial alert as a false positive, and subsequently identify all three true-positive alerts as they appeared. I did not receive immediate confirmation after each decision; the final result was provided only after the scenario concluded.

Although this was a relatively simple scenario, the improvement in my confidence and ability to interpret the evidence was more meaningful to me than the difficulty of the exercise itself. It showed me that the concepts I've been studying over the past several months are becoming easier to apply during an investigation.
