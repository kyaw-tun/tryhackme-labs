# Complimentary

> **TryHackMe Room:** [Complimentary](https://tryhackme.com/room/hh-complimentary-05e0b604)

This write-up covers how I approached the room, step by step. Flags and specific key and password values are not included.

## Introduction

This is a cloud security room. When I first did it about four months ago, it was my first encounter with cloud security. Since then I have completed two or three other rooms on the subject, but I still consider myself a beginner in this area.

The room is fairly easy, and even easier for anyone familiar with cloud environments.

## Initial Access

The room provides a link to the lab, which is an `s3-website` endpoint ending in `amazonaws.com`. The site itself is not very interesting, but its source code is. The `app.js` file contains an identity pool ID, a region, and a table name.

Next, I opened the network tab in the browser's developer tools and refreshed the page. It listed many requests made by the site, and the interesting one was the request to `https://cognito-identity.us-east-1.amazonaws.com/`. Selecting it and opening the response tab revealed credentials for a guest identity, including `AccessKeyId`, `SecretKey`, and `SessionToken`.

## Getting the Flag

To get the flag, I needed to access the table named in `app.js`. This requires the AWS CLI to be installed. There are other ways to reach the table using only the credentials from the website, but they require writing code, which I did not do here.

I added the credentials with the `aws configure` command, which prompts for each value:

```bash
AWS Access Key ID [None]:
AWS Secret Access Key [None]:
AWS Session Token [None]:
Default region name [None]:
Default output format [None]:
```

The AWS CLI saves these values in a file called `credentials` in the `~/.aws/` folder. I then checked that they worked with this command:

```bash
aws sts get-caller-identity
```

It returned the unique account and identity tied to the guest credentials, so I knew they were valid and could start testing what they had access to in the database table. I could not run `describe` on the table, as it returned a "permission denied" error, but I could run `scan`:

```bash
aws dynamodb scan --table-name complimentary-GuestWellnessProfiles --region us-east-1
```

The output listed usernames, passwords, full names, notes, and other fields in `json` format in the terminal, and the flag was in there.

## Conclusion

This room is fairly easy even without prior cloud experience, especially given the clues in the `concierge briefing` and `@0xMia's STORY`. It is also not the only way to solve it. I solved it with another method as well, but I preferred this one, which is why I chose to document it.

I did run into dead ends and errors, notably the permission denied error and finding the credentials in the first place, but those came from my inexperience rather than from the difficulty of the room. Overall, it gave me a useful overview of cloud security and a glimpse of what the field might look like if I decide to pursue it in the future.
