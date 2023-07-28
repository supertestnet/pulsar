# Pulsar
My submission for the HRF's encrypted group chat bounty

# What is this?
Pulsar is an encrypted chat app that masks all your metadata except your ip address. It works by sending fixed-length, fixed-frequency messages over the nostr messaging network.

# How does it work?
A group creator gives a "shared secret" to the other members, allowing anyone who knows it to encrypt and decrypt group messages. Every user transmits an encrypted message every 3 seconds, using a fresh, new nostr key, even when they haven't typed anything. Most of these messages contain junk data. Each client decrypts this data, detects that it is junk, and does not display it. All of these "junk" messages are the same length -- a little over 1000 characters. When a user *really* types something, their real message is queued so that it goes out only when the next every-3-second-message is scheduled. It, too, is limited in size, and if it is less than the maximum size it is padded to 1000 characters. That way it then "blends in" with the "fake" messages that are transmitted every 3 seconds. Once the message is decrypted, clients can detect whether a message is fake or real, and they only display the real ones. Each user's real pubkey and real signature is also embedded within their messages so that people who know the shared secret can know who is saying what. 

# What data does this protect?
Because messages are encrypted, observers cannot surveil message contents.

Because of fixed-length, fixed-frequency messaging, observers cannot gather message frequency.

Because of ephemeral keys, observers cannot gather user identities, except by learning a user's ip address, which means you still need to use tor or a vpn.

Thus this app fulfills the terms of the bounty.
