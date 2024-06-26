---
title: "Phishing and Digital Hygiene"
author: jerryyf
date: 2024-05-07
toc: true
tags: ["security"]
---

Too many people around me have been falling victim to phishing scams recently, so in this post I want to cover some basic ways to spot phishing, as well a deeper dive into maintaining online hygiene and personal security practices to minimise attack surface in the first place.

<!--more-->

I had hoped most of this was common knowledge, but perhaps not, so hope you may learn something new and useful.

## tl;dr

- **Always check the link** before you open it. Whether that's hovering over it to see a link preview, or just examining it carefully by eye in an SMS or email.
    - If it's a link ending with `.top`, `.info`, `.life` - don't open it. These are cheap top level domains (TLDs) that can be mass purchased for phishing purposes.
- Take the [Google Phishing Quiz](https://phishingquiz.withgoogle.com/) - quick and helpful for learning how to identify email phishing, but shows how scammers attempt to cleverly obfuscate malicious URLS with legitimate looking ones. Don't forget to check that link before opening it.
- **Use a password manager.**
	- [Bitwarden](https://bitwarden.com/) - probably the most widely used and trusted open source password manager. Great browser integrations and secure E2EE cloud sync.
	- [KeePassXC](https://keepassxc.org/) - good offline option for the extra paranoid. Features browser integration, password generation, and strong database encryption.
- Check if your email address has been in a data breach on [haveibeenpwned](https://haveibeenpwned.com). If it has, your password for whatever service was breached is now in database dumps on publicly accessible forums. Therefore,
	- Change your passwords. Ideally to something resilient to brute-force and dictionary attacks 
		- This is why a password manager is perfect - generate different strong passwords for every service.
	- Never reuse passwords - one account password being leaked means all your accounts become at risk
- **Enable TOTP MFA for as many services as you can (must for password manager).**
	- Cannot be stressed enough - even if your login details are compromised, an extra layer of authentication is required. Avoid SMS and email MFA, go for TOTP using an authenticator app.
	- [Aegis Authenticator](https://getaegis.app/) - a nice and secure, feature-rich open source MFA token manager app for Android

## Phishing

There are mainly 2 types of scams I see people falling victim to:

- SMS
- Email
- Robocalls

Last but certainly not least, spearphishing, but for the majority of us with a low profile and low threat model there shouldn't be a need to worry about this.

### SMS phishing

Usually the primary means of general phishing (as opposed to spearphishing), though this can vary by country. Mass sending of overdue urgent bill notices with a malicious payment link is quite a common one. Others to watch out for are missed parcel deliveries, and fake government messages.

### Email phishing

Email phishing, can be hard to spot due to the sheer abundance of it. We all receive multiple emails a day; they are long and mundane. So when the occasional alarming one like `Someone has just accessed your Google account` jumps out, it plays into our *emotional bias* and creates a sense of urgency to act. Just calm down and think before entering and passwords, verification codes or financial details.

### Robocalls

These are simply annoying. Robot voice press 1 for English 2 for Chinese, just hang up, or better yet press 1 or 2 and waste their time for fun.

### Spearphishing

A more sophisticated form of phishing where a malicious actor gathers, or already knows enough information about you to launch a targeted social engineering attack. Typically with the aim to exploit your emotional bias, these targeted attacks can make you feel vulnerable and in imminent danger, causing you to react faster and fire away your card details into that scam site. Not as common as general mass phishing attempts over SMS, phone call and email.

## Digital hygiene

In today's world it is simply no longer acceptable to be careless about personal privacy and security as it was when the internet was first around. Cybercrime is on the rise more than ever before. Our dependency on the internet has us, really, forming a third sense of self - the digital identity, in addition to our physical self and mental self. This should be protected with as much care as we would to protect our physical and mental self image, just like how we eat to stay alive and live by our values to uphold our own perceived identity.

### Complacency

Passwords used to be stored in plaintext - some companies even still do that because they believe their database is secure. This false sense of security - complacency - is dangerous on both a business and personal level.

As mentioned before, one of the best things one can do for their own digital footprint is to update all your account passwords to something randomly generated by a password manager. I would avoid using a builtin browser one such as Google Password Manager for Chrome, and Firefox Passwords. Control your own data, especially sensitive information like passwords. Try to not rely on any cloud service, regardless of how reputable they are. Your data should be in your own hands - no one should be able to access it, and it should not rely on the infrastructure of a single third party.

### Concept of zero trust

This ties back into not relying on third party services for your data. What would happen if something happened to your Google account, that which everything is stored on? [This incident](https://www.theguardian.com/technology/2022/aug/22/google-csam-account-blocked) is a perfect case study demonstrating this concept. To this day nothing has been resolved.

**So where should my huge amounts of data be stored then?**

It doesn't have to move anywhere. Just make copies of it on storage that **you control**. Some ideas:

- Make backups on local hard disks
	- It's important to still locally backup passwords when using a cloud provider like Bitwarden
- If you're more privacy conscious, encrypt your data before uploading to a cloud service provider, use an verifiable open source E2EE solution, or better yet host your own. Some great self-hosting options:
	- [Nextcloud](https://nextcloud.com/)
	- [EteSync](https://www.etesync.com/)
	- [Syncthing](https://syncthing.net/)

### Blocking ads and tracking

A discussion with coworkers about what ad-blockers they used popped up, and the following what was said is what inspired me to write this section:

*"I just use the one everyone uses, AdBlock Plus"*

Unfortunately it seems true that a large majority use an ad-blocker that receives monetary benefit from advertisers (including Google, meaning YouTube ads aren't blocked) to whitelist their ads. There is nothing *inherently* wrong with supporting the internet business model and such, but [the way in which the company goes about it has caused controversy](https://www.theguardian.com/technology/2013/oct/14/the-tiny-german-company-threatening-the-internets-business-model) in the last decade. From an advertiser's perspective, your ads get blocked, then you are forced to pay money to get them whitelisted. From a user's perspective, it's not a fully functioning ad-blocker. [uBlock Origin](https://ublockorigin.com/) is the best open source ad-blocker available. In actual fact it is more than an ad-blocker if you get deeper into its config. Independent, maintained by community and yes, will block YouTube ads.

## Closing words

Feel free to reach out to me if you would like to know more about anything covered in this post. I have only scratched the surface on privacy and security here, and I am always happy to share my insights.
