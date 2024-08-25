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

(2) Observers can detect, at any given time, how many people are sending messages that reference a group's shared public key, and use that to estimate how many people are in a group

(3) Observers can detect when someone starts sending messages to a group and when they stop. For example, if they observe that there were 6 messages going to the group every 3 seconds, but then suddenly there are 7, that probably means a new person entered the group

(4) Observers can treat the time period during which someone is sending messages to a group as a "session." They can detect the duration of each session and track its start and stop times to identify the likeliest time zones that every group member lives in. So observers might not know who you are, but they might get a good idea of what time zone you live in

(5) Because each session sends a message every 3 seconds, and never varies from that, observers can get an upper bound on the number of messages you sent during your session. E.g. if your session lasted 1 hour, they know that there are 3600 seconds in 1 hour, meaning your client sent a total of 1200 messages during your session. They know that most of those messages were probably junk messages, but some of them could have been real. So the maximum number of real messages you could have sent during your session is 1200 -- that's the "upper bound." Note that this would only give them an upper bound on how many real messages you *could* have sent during your session. It would not tell them the *actual* number of real messages you sent

(6) Observers who control one of the relays users connect to can additionally see when websocket connections are opened and closed, and treat those as additional data points about when people log in and log out

(7) If the tor network is compromised, e.g. if feds are running most of the nodes and logging all ip traffic, there is a meaningful chance they can identify your ip address and, through that, your real identity. Global surveillance of this sort can probably also figure out what relays you are talking to, and map that info to what group chats are ongoing, so that they can perhaps learn what group chats you are in
