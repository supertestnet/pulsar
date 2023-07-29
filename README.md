# Pulsar
My submission for the HRF's encrypted group chat bounty

# What is this?
Pulsar is an encrypted chat app that masks your metadata. It works by sending fixed-length, fixed-frequency messages over the nostr messaging network.

# How to try it

Our development version, on clearnet, **doxes your ip address**: https://supertestnet.github.io/pulsar/

Our production version, on tor, **protects your ip address**: http://kzthpkengwzo7tjo7xh36xmjpxdyxlhky76lwxsiop2zogt44udidsqd.onion/

# Why did you make this?
The human rights foundation [posted](https://bitcoinmagazine.com/business/human-rights-foundation-announces-20-btc-bounty-challenge-for-bitcoin-development) the following bounty:

> 2 BTC for the creation of end to end encrypted group chats powered by any popular Nostr client that does not leak metadata to third parties. Users must be able to chat with at least two other Nostr users. Outside observers must not be able to see the content of messages, the sender/recipient of messages, or the total number of messages between recipients.

Consequently, my goals are:

(1) outside observers cannot see message contents

(2) outside observers cannot see user identities

(3) outside observers cannot see number of messages

(4) three or more users can communicate

# How does it work?
A group creator creates a "chat string" -- i.e. a shared secret -- and sends it to the other members. Anyone who knows the chat string can encrypt and decrypt group messages. Every user transmits an encrypted message every 3 seconds, using a fresh, new nostr key, even when they haven't typed anything. Most of these messages contain junk data. Each client decrypts this data, detects if it is junk, and discards it if so. All of these junk messages are the same length -- a little over 1000 characters. When a user *really* types something, their real message is queued so that it replaces the next junk message in the queue, and goes out when that junk message *would have* gone out. The real message, too, is limited in size, and if it is less than the maximum size it is padded to a little over 1000 characters. That way it "blends in" with the junk messages that are transmitted every 3 seconds. When a real message is decrypted by the other clients, they detect that it isn't junk, so they discard the padding (if any) and display the message. Each user's real pubkey and real signature are also embedded within each of their encrypted messages so that everyone who knows the shared secret also knows who said what.

# What data does this protect?
(1) Message contents -- all messages are encrypted

(2) User identities -- all keys rotate constantly and users connect via tor

(3) Number of messages -- real messages are queued with junk messages and padded to the same size, then sent out with fixed frequency so they blend in

Also, the shared secret allows any number of people to use Pulsar to communicate with one another, be they 2, 3, 10, 2,000 or whatever. Thus Pulsar fulfills the terms of the bounty.

# What metadata can outside observers still detect?
(1) Each group has a "shared public key" which all in-group messages reference in each of their messages. Observers can detect all references to this shared public key

(2) Observers can detect, at any given time, how many people are sending messages that reference a group's shared public key, and treat these as messages to everyone in the group

(3) Observers can treat the moment you start sending messages to a group as a "log in" moment and the moment you stop as a "log out" moment, thus getting an upper bound on the number of messages you sent, though this would not tell them the *actual* number of messages you sent

(4) Observers who control one of the relays users connect to can additionally see when websocket connections are opened and closed, and treat those as additional data points about when people log in and log out

(5) Observers can combine the previous data points and use heuristics to guess how many people are probably in a group and their possible time zones. For example: "5 people logged in to the same group during primetime in the Eastern USA time zone. 2 people logged into the same group during primetime in the Western European time zone. So there are probably 7 people in the group in two different time zones."

(6) Observers know in advance that the maximum size of any message is about 1000 characters
