---
title: "Backups: 2019 Update"
description: "How I backup my computers and applications in 2019"
date: 2019-02-09T14:00:00-05:00
tags: ["backup", "guide"]
categories: ["guide"]

resizeImages: false
---

A lot has changed in the three years since [my last post on backups](/post/backups-2016/).

This post is an updated version with all the changes and updates that I have done in three years. Backups are still a _very_ important thing for me, and I'm still pretty extensive with them.

<!--more-->

## Foundations

My current setup that needs to be covered by backups is:

* 2012 Retina MacBook Pro (same computer from 2016)
* iPhone XS Max
* iPad Pro 10.5-inch (2017)
* Synology DS413j Network Attached Storage (NAS)

## Whole-Device Backups

For my onsite backups of my laptop I still use [SuperDuper!](http://www.shirt-pocket.com/SuperDuper/SuperDuperDescription.html), which I use every week to make a bootable clone of my laptop. I have stopped using TimeMachine because I was having way too many problems with the backups to the NAS.

Offsite backups are now split between [Backblaze](https://secure.backblaze.com/r/01upmv) for my laptop and [Backblaze B2](https://www.backblaze.com/b2/cloud-storage.html) for my NAS. CrashPlan switched to only selling to businesses, but I had switched over even before that. Backblaze (both normal and B2) have been rock-solid for me.

My iOS devices (iPhone and iPad) are still using [iCloud Backup](https://www.apple.com/support/icloud/backup-storage/). My [encrypted backups](https://support.apple.com/en-us/HT205220) to iTunes are now only done before iOS updates and switching devices.

### A Note on Dropbox

[Dropbox](https://dropbox.com) is still pretty good, but I have also moved a lot of my data out of Dropbox and into [DEVONthink](https://www.devontechnologies.com/products/devonthink/overview.html). I have DEVONthink syncing through a mix of personal WebDAV and iCloud.

However: **Neither DEVONthink nor Dropbox are backup.** Though it can be used for it in a pinch, both are just storage and synchronization systems.

## Secondary Backups: Apps and Services

So many changes to apps and services in just three years. The ones I still use from 2016 without any changes are Pinboard (bookmarks), Overcast (podcasts), and Paprika (recipes).

### Old Apps, New Details

I still use some of the apps from 2016, both there are updates to how I handle backups for them.

#### Journaling: Day One

Day One is a lot easier to backup now. From the menu select Export → JSON and download any media necessary. That will backup all of your posts, images, and other content into a single zip file.

#### Budgeting: YNAB

I have stuck with YNAB because I haven't been annoyed by it enough to change. I have accepted that my content is on their servers and cope. Every month I do backup my data from YNAB.

#### Passwords: 1Password

I now use [1Password for Families](https://1password.com/families/) and I have my entire family on it. I accept that I cannot easily backup my passwords and associated documents, but I have invested _heavily_ in 1Password and the risk I take from using them is (I feel) less than the risk I would have from trying to handle everything myself.

Also, a large part of that is the risk that is mitigated by having an easy-to-use system for all of my family members to use. The aggregated risk mitigation is worth the cost to me many, many times over.

#### Source Code: GitLab

I still run my personal GitLab instance, but I am in the process of moving a lot of my code to the commercial [GitLab.com](https://gitlab.com/) system. Some of my code still lives in [GitHub](https://github.com/gubler) and I want to move a lot of that to GitLab as well. I do plan on syncing my code from GitLab.com to my local GitLab, but only as a backup.

I no longer run a git server on my NAS - it was not worth the hassle.

#### Task Management: OmniFocus 3

I switched to [Things 3](https://culturedcode.com/things/) for awhile, but when OmniFocus 3 (OF3) was released, I returned to it. I have a bunch of tools built to push data into OF3 (via [ShortCuts](https://itunes.apple.com/us/app/shortcuts/id915249334) or [Drafts](https://itunes.apple.com/us/app/drafts-5-capture-act/id1236254471)), but getting data _out_ is not something I worry about much.

I do however do a backup of my [OmniSync Server](https://manage.sync.omnigroup.com/) data every month and that include my OF3 data.

### New Apps

And here's the new stuff.

#### Email: FastMail

All my email now goes to [FastMail](https://www.fastmail.com/?STKI=16733801) and I use [MailRoute](https://www.mailroute.net/) for spam/virus filtering. just use Apple's built in mail client (on both macOS and iOS) and backup from the macOS Mail app into DEVONthink.

#### RSS: FeedBin

I switched over to [FeedBin](https://feedbin.com/) and I do a monthly [export of my subscriptions](https://feedbin.com/settings/import_export).

#### Writing: Drafts and SublimeText

All my writing is now started in either [Drafts](https://itunes.apple.com/us/app/drafts-5-capture-act/id1236254471) on the iPad or SublimeText on macOS. No matter which app it starts in, all of my writing ends up in DEVONthink.

#### DEVONthink

DEVONthink is a surrogate filesystem for me. I have seven different databases in it currently and every scan, text document, image, note, idea, or record goes in. Its better to say what _doesn't_ end up in DEVONthink for me: music (either Plex on the NAS or Apple Music), video (Plex on the NAS), ebooks (Calibre), photos (iCloud Photo library/Photos app) and source code (GitLab/GitHub).
