# Pulsar
My submission for the HRF's encrypted group chat bounty

# What is this?
Pulsar is an encrypted chat app that masks all your metadata except your ip address. It works by sending fixed-length, fixed-frequency messages over the nostr messaging network.

# Why did you make this?
The human rights foundation [posted](https://bitcoinmagazine.com/business/human-rights-foundation-announces-20-btc-bounty-challenge-for-bitcoin-development) the following bounty:

> The third bounty is another 2 BTC reward for the creation of a Nostr client implementation of end to end encrypted group chats which is incapable of leaking metadata to potentially malicious third parties. In order to be eligible, the group chat must enable three or more users to communicate, with no ability for outside adversaries to gather the content of the messages, nor the identity or frequency of the users messaging.

The goals are:

(1) let 3 or more users communicate

(2) outside observers cannot see message contents

(2) outside observers cannot see user identities

(3) outside observers cannot see message frequency

# How does it work?
A group creator gives a "shared secret" to the other members, allowing anyone who knows it to encrypt and decrypt group messages. Every user transmits an encrypted message every 3 seconds, using a fresh, new nostr key, even when they haven't typed anything. Most of these messages contain junk data. Each client decrypts this data, detects that it is junk, and does not display it. All of these junk messages are the same length -- a little over 1000 characters. When a user *really* types something, their real message is queued so that it goes out only when the next junk message is scheduled. The real message, too, is limited in size, and if it is less than the maximum size it is padded to 1000 characters. That way it "blends in" with the junk messages that are transmitted every 3 seconds. Once the message is decrypted by the other clients, they can detect whether a message is junk or real, and they only display the real ones. Each user's real pubkey and real signature is also embedded within their message so that all who know the shared secret also know who said what.

# What data does this protect?
Because all messages are encrypted, observers cannot surveil message contents.

Because real messages blend in with fixed-frequency junk messages, observers cannot surveil message frequency.

Because of ephemeral keys, observers cannot gather user identities, except by learning a user's ip address, which means you still need to use tor or a vpn.

Also, any number of users can use Pulsar to communicate, 3 or 10 or 2,000 or whatever. Thus Pulsar fulfills the terms of the bounty.
