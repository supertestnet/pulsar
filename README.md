# Pulsar
My submission for the HRF's encrypted group chat bounty

# What is this?
Pulsar is an encrypted chat app that masks all your metadata except your ip address. It works by sending fixed-length, fixed-frequency messages over the nostr messaging network.

# Why did you make this?
The human rights foundation [posted](https://bitcoinmagazine.com/business/human-rights-foundation-announces-20-btc-bounty-challenge-for-bitcoin-development) the following bounty:

> The third bounty is another 2 BTC reward for the creation of a Nostr client implementation of end to end encrypted group chats which is incapable of leaking metadata to potentially malicious third parties. In order to be eligible, the group chat must enable three or more users to communicate, with no ability for outside adversaries to gather the content of the messages, nor the identity or frequency of the users messaging.

Consequently, my goals are:

(1) three or more users can communicate

(2) outside observers cannot see message contents

(3) outside observers cannot see user identities

(4) outside observers cannot see message frequency

# How does it work?
A group creator creates a "chat string" -- i.e. a shared secret -- and sends it to the other members. Anyone who knows the chat string can encrypt and decrypt group messages. Every user transmits an encrypted message every 3 seconds, using a fresh, new nostr key, even when they haven't typed anything. Most of these messages contain junk data. Each client decrypts this data, detects if it is junk, and discards it if so. All of these junk messages are the same length -- a little over 1000 characters. When a user *really* types something, their real message is queued so that it goes out only when the next junk message is scheduled. The real message, too, is limited in size, and if it is less than the maximum size it is padded to a litle over 1000 characters. That way it "blends in" with the junk messages that are transmitted every 3 seconds. When a real message is decrypted by the other clients, they detect that it isn't junk, so they discard the padding (if any) and display the message. Each user's real pubkey and real signature is also embedded within each of their messages so that everyone who knows the shared secret also knows who said what.

# What data does this protect?
Because all messages are encrypted, observers cannot surveil message contents.

Because real messages blend in with fixed-frequency junk messages, observers cannot surveil message frequency.

Because of ephemeral keys, observers cannot gather user identities, except by learning a user's ip address, which means you still need to use tor or a vpn.

Also, any number of users can use Pulsar to communicate with one another, be they 2, 3, 10, 2,000 or whatever. Thus Pulsar fulfills the terms of the bounty.
