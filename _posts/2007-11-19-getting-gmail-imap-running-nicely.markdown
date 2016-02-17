---
layout: post
title: Getting Gmail IMAP Running Nicely
tags:
- geek
- gmail
- google
- os-x
---

I've just managed to get my Gmail service upgraded to the new version with IMAP support, (If you're a
user in the UK like me and are still waiting for your Gmail service to upgrade, simply navigate into your
Gmail settings and change your language settings from 'English (UK)' to 'English (US)'. This should make
your Gmail switch to version 2 instantly!) and as such i've been trawling the net for information on how
to get this all set-up nicely. By this I mean moving my mail filtering rules onto Gmail itself and
getting IMAP to play a bit more nicely with Apple Mail...


Here's how to get Apple Mail to work as expected, thanks to [5ThirtyOne](http://5thirtyone.com/):

> Similar steps must be taken to ensure that any emails sent, saved as drafts, or deleted are properly identified by Gmail's servers. After completing the IMAP setup steps for Apple Mail, instructing Mail is a few simple clicks away. Once your Gmail IMAP account is added to Mail, you'll notice your [Gmail account] in the left sidebar.
>
> 1. Highlight '[Gmail] Sent Mail' in the sidebar and select 'Mailbox' (menu bar) > 'Use This Mailbox For' > 'Sent'.
> 2. Highlight '[Gmail] Drafts' in the sidebar and select 'Mailbox' (menu bar) > 'Use This Mailbox For' > 'Drafts'
> 3. Highlight '[Gmail] Trash' in the sidebar and select 'Mailbox' (menu bar) > 'Use This Mailbox For' > 'Trash'
> 4. Highlight '[Gmail] Spam' in the sidebar and select 'Mailbox' (menu bar) > 'Use This Mailbox For' > 'Junk'
>
> Once properly configured, managing email from Apple Mail or the iPhone will be no different from managing emails within the Gmail web client - sent, drafts, trash, and junk properly sorted between your various email clients and web interface.

Read the [full article](http://5thirtyone.com/archives/862) for even more information (including iPhone
set-up).

And the best hints for setting up slightly more complex filters in Gmail came from
[Lifehacker](http://lifehacker.com/):

> I needed to set up a filter that would apply label 'work' to any
> email that came from 'xyz@workplace.com' OR had the word 'workoholic'
> in it. Unfortunately, Gmail's inbuilt 'Create a filter' feature's
> dialog boxes are connected with an AND operator. Example: if I wrote
> ' xyz@workplace.com' in the 'from:' field of the 'Create a filter'
> page, and 'workoholic' in the 'Has the words:' field, then only emails
> that came from 'xyz@workplace' AND had the word 'workoholic' would be
> picked up by this filter. So, to get the desired result, I needed TWO
> filters. Unless...
>
> [...]
>
> Incorporate both the conditions in the 'Has the words:' field
> itself! So my 'Has the words' field reads as: (from:(xyz@workplace.com)
> OR (workoholic)). Voila! c'est fini! But the possibilities are endless!

Read the [full article](http://lifehacker.com/software/gmail/supercharge-your-gmail-filters-204781.php)
for more information.
