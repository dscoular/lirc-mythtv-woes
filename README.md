# lirc-mythtv-woes

Since upgrading from Mythbuntu 16.04.5 to 18.04, my mythtv config files have two issues:

1. Any remote key press loops until I hit a key on the keyboard, then the remote works perfectly.
2. Hitting the KEY_SLEEP on the remote sends my server to sleep! This is a 24x7 mythtv server so that is bad news.

The 16.04.5 config worked perfectly. I see lots of advice about removing lirc but then I read that the reason people think this is due to the fact that lirc config files have changed radically since 0.9.4 and that you really need to manually rewrite the configs. I followed what I could of the guide at https://lirc.org to update to 0.10.0 (the version on mythbuntu 18.04).

This repo is just to let people browse my config files and give their 2 cents.

Doug Scoular

