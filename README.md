# Pulsar
My submission for the HRF's encrypted group chat bounty

# What is this?
Pulsar is an encrypted chat app that masks your metadata. It works by sending fixed-length, fixed-frequency messages over the nostr messaging network.

# How to try it

Our development version, on clearnet, **doxes your ip address**: https://supertestnet.github.io/pulsar/

Our production version, on tor, **protects your ip address**: http://kzthpkengwzo7tjo7xh36xmjpxdyxlhky76lwxsiop2zogt44udidsqd.onion/

# Why did you make this?
The human rights foundation [posted](https://bitcoinmagazine.com/business/human-rights-foundation-announces-20-btc-bounty-challenge-for-bitcoin-development) the following bounty:

> The third bounty is another 2 BTC reward for the creation of a Nostr client implementation of end to end encrypted group chats which is incapable of leaking metadata to potentially malicious third parties. In order to be eligible, the group chat must enable three or more users to communicate, with no ability for outside adversaries to gather the content of the messages, nor the identity or frequency of the users messaging.

Consequently, my goals are:

(1) outside observers cannot see message contents

(2) outside observers cannot see user identities

(3) outside observers cannot see message frequency

(4) three or more users can communicate

# How does it work?
A group creator creates a "chat string" -- i.e. a shared secret -- and sends it to the other members. Anyone who knows the chat string can encrypt and decrypt group messages. Every user transmits an encrypted message every 3 seconds, using a fresh, new nostr key, even when they haven't typed anything. Most of these messages contain junk data. Each client decrypts this data, detects if it is junk, and discards it if so. All of these junk messages are the same length -- a little over 1000 characters. When a user *really* types something, their real message is queued so that it replaces the next junk message in the queue, and goes out when that junk message *would have* gone out. The real message, too, is limited in size, and if it is less than the maximum size it is padded to a little over 1000 characters. That way it "blends in" with the junk messages that are transmitted every 3 seconds. When a real message is decrypted by the other clients, they detect that it isn't junk, so they discard the padding (if any) and display the message. Each user's real pubkey and real signature are also embedded within each of their encrypted messages so that everyone who knows the shared secret also knows who said what.

# What data does this protect?
(1) Message contents -- all messages are encrypted

(2) User identities -- all keys rotate constantly and users connect via tor

(3) Message frequency -- real messages are queued with junk messages and padded to the same size, then sent out with fixed frequency so they blend in (but see #6 in the section below this one -- attackers can currently exploit the encryption method to discover messsage frequency and message length, and that must be fixed)

Also, the shared secret allows any number of people to use Pulsar to communicate with one another, be they 2, 3, 10, 2,000 or whatever. Thus Pulsar fulfills the terms of the bounty.

# What metadata can outside observers still detect?
(1) Each group has a "shared public key" which all in-group messages reference in each of their messages. Observers can detect all references to this shared public key

(2) Observers can detect, at any given time, how many people are sending fixed-frequency messages that reference a group's shared public key, and treat these as messages to everyone in the group

(3) Observers can treat the moment you start sending messages to a group as a "log in" moment and the moment you stop as a "log out" moment

(4) Observers can combine the previous data points and use heuristics to guess how many people are probably in a group and their possible time zones. For example: "5 people logged in to the same group during primetime in the Eastern time zone. 2 people logged into the same group and talked during primetime in the Western European time zone. So there are probably 7 people in the group in two different time zones."

(5) Observers can see that the maximum size of any message is about 1000 characters

(6) The AES-CBC encryption standard, as implemented by nip4 and therefore Pulsar, uses an "initialization vector" to encrypt messages in blocks. One attack against this encryption standard involves guessing the contents of an encrypted block, then using the initialization vector as a kind of checksum to see if your guess was correct. Guessing correctly is often easy to do if the messages follow a preknown format. For example, Pulsar currently uses 0-value byte vectors to pad real messages, and junk messages are basically just 1000 0-value byte vectors. Since the way Pulsar produces padding and junk has a known pattern, an attacker can take each encrypted block, "guess" that it's a bunch of zeroes, and use the intiialization vector to check if their guess is correct. If it is, they can discard that padding. This attack allows an observer to identify and discard all "junk" messages, and also identify and discard almost all of the padding in a real message. This attack thus reveals the frequency of real messages as well as their length. There are two easy ways to fix it, though: (1) use random byte vectors, not 0-value byte vectors, as padding (2) use a different encryption scheme. We intend to shortly mitigate this attack using option (1).
