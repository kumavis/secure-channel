researching what makes a good connection protocol for secure-scuttlebutt, am investigating different protocols to see what level of privacy & simplicity is achievable.

One thing that is fairly obvious is the TLS is not very good, and unsuitable for a p2p protocol.
In a p2p protocol we can usually regard key management as a solved problem - with the web, there is the whole CA system to certify that a given peer (ip address) is actually the owner of the given domain, but in a p2p system peers are identified by their public key (which is stable) and not their ip address, which is unstable. Usually there is a some sort of look-up system to get the ip addresses,  
so it may not be necessary to send the key inside the connection.

## private-stream

[private-stream](https://github.com/dominictarr/private-stream) is a very simple private protocol that I wrote. It provides encryption, but not authentication. That is, you cannot be sure exactly who you are talking to, but you may be sure that no one else is listening. It is a symmetrical protocol, in that client and server handshake are identical. Each peer sends a Diffie Helman public key + an Initialization Vector (8 byte random number). The DH keys are combined to produce a shared key, and a cipher streams (salsa20) are constructed using the shared key and the two ivs so that each channel (A->B, and B->A) have the same key but different IVs (which is sufficient for security)

I was considering putting authentication inside of this protocol but I am now re-evaluating that.

### dramatization of private stream

alice and bob meet in a dark alleyway

Alice & Bob (simultaniously) passes each other a secret note, also with random number written on outside.

> Alice and Bob now combine their secret, with the secret they received and the random numbers,
> to communicate with each other.

Alice does not know who Bob is and vice versa, but no one can tell what they are saying either.

### Further Reading

* [readme](https://github.com/dominictarr/private-stream)

## bittorrent obfuscation

BT encryption is not well regarded, but because it chooses insecure algorithms, not because it's abstract design is bad. It provides the same assurances (or it would, if it's ciphers where secure) as private-stream, except it's a little uglier because it's not symmetrical. (the stream has to know whether it's the caller (client) or answerer (server), so it can derive a distinct key)

### Dramatization of Bittorrent Obfuscation

Alice and Bob meet in an alleyway wearing silly costumes.
They do not realize there is a security camera.
Luckily nobody really cares what they are up to.
Alice and Bob exchange notes and feel like they are spies.
etc, etc.

### Further Reading

* [bittorrent creator on why obfuscation is silly](http://bramcohen.livejournal.com/29886.html)
* [message stream encryption](http://wiki.vuze.com/w/Message_Stream_Encryption)
* [wikipedia page](https://en.wikipedia.org/wiki/BitTorrent_protocol_encryption)

## tls

### dramatization of a TLS handshake:

Alice: "hello, I speak english, spanish, and mandarin"
> (alice opens connection, and describes the cihper suite she supports)

Bob: "hi this is bob and I speak english, just ask carl"
> (bob selects cipher, identifies himself, with a reference.
> carl is a respected member of the community (a CA) who knows bob (has issued bob a cert).
> alice may choose to quickly contact carl to check if bob is cool)

Alice:
  (whispers) "hey bob, the key is 'foofoo'"
> (alice chooses a secret and encrypts it to bob)

> Bob/Alice now encrypt all further messages with the key.

### Further Reading

* [tls handshake](http://en.wikipedia.org/wiki/Transport_Layer_Security#TLS_handshake)

## SSH

SSH seems very complicated and has a variety of methods of authenticating. Flexibility in a security protocol is bad because then there are more edge cases and thus more surface area to audit / cracks for bugs to hide in. [djb agrees](http://curvecp.org/security.html)

SSH differs from our model p2p protocol because there is no key/address lookup system, and so a client cannot know that it's information on a host is uptodate. Also, it may only update information about a host by connecting to it

### Dramatization of SSH connection

Alice: "hi I want to talk to you, in english, spanish or mandarin (prefer english)"
> Alice opens a connection, and lists the ciphers she supports, with preference.

Bob: "hi I speak in english, spanish or mandarin (prefer english), okay lets whisper now"

Alice: passes a secret note to Bob
> generates DiffieHelman key, sends public DH key to bob.
> begins DH exchange

Bob: passes a note back, with I AM BOB signed on the outside.
> bob replies with bob's public DH key, bob's public RSA key, and a signature to prove they are bob.
> Alice now knows she is talking to Bob, Bob does not know who Alice is yet,
> but they do have secure communication now.

To be honest, this seems a little unfair on bob. He has answered the phone and identified himself to a stranger. It happens to be his friend Alice, but it _could_ be a telemarketer, who is now updating their database because they now know where bob lives.

Alice and Bob now have secure communication, and inside that, Alice authenticates to Bob.
I think the reason that SSH is designed this way is to support a variety of client authentication protocols, such as password, host based (checks the ip address of the client (!)), keyboard interactive (usually used for a password, the client's keyboard is directly connected to a program running on the server so it can be literally anything (!)), and finally, pubkeys.

We are only interested in pubkey based authentications.

The following is all private so Alice and Bob can speak normally.

Alice: hey I'm alice and I want to use pubkey auth with `algorithm` and `key`

Bob: okay go ahead "Alice"

Alice: see it's me: _signed, Alice!_

Bob: Alice it is you!!!

### Further Reading

* Architecture of SSH protocol [rfc4251](http://tools.ietf.org/html/rfc4251)
* SSH transport protocol [rfc4253](http://tools.ietf.org/html/rfc4253) (encryption, and server auth)
* SSH Client Authentication Protocol [rfc4252](http://tools.ietf.org/html/rfc4252)

## CurveCP

djb's new Mutual Authenticated UDP based protocol. It uses a UDP to get around some weaknesses in TCP (such as the ability to forge a reset packet and force a tcp connection to close) and also to use a different congestion than tcp does.

### Further Reading

* [curvecp website](http://curvecp.org)