# Tunnelgram

Easy, secure, end to end encrypted (E2EE) messenger.

## Advantages Over Other E2EE Messengers

* Every message is encrypted end to end. There is no way to send an unencrypted message.
* Users can login on **multiple clients**, read all of their conversations and messages, and send new messages. In this way, it works just like any other messenger.

## Support Tunnelgram

[![Become a patron on Patreon](https://c5.patreon.com/external/logo/become_a_patron_button.png)](https://www.patreon.com/tunnelgram)

# The Tunnelwire Encryption Scheme

How is Tunnelgram able to show all your messages on any client just like a regular messaging app if it's all end to end encrypted?

It uses the **Tunnelwire Encryption Scheme**. I invented this scheme in 2013, and Tunnelgram is the first application to use it.

The Tunnelwire Encryption Scheme has some requirements that can't be met by other messengers.

1. The server must never know your password.
2. The server must never know your private key.

With these two key principals, Tunnelgram can use the Tunnelwire Encryption Scheme to securely send encrypted messages between any number of parties.

## How the Tunnelwire Encryption Scheme Works

### Upon Registration

When a user registers, the client must:

1. Hash the password that the user entered, using a cryptographically secure, one way hashing algorithm. *Tunnelgram uses one iteration of SHA-512.*
2. Derive an encryption key and a remainder from the hash. Neither the key nor the password should be practically derivable from just the remainder. There are multiple ways to do this:
  * Remove some portion from the hash as the key. The remaining portion becomes the remainder. *Tunnelgram removes the first 32 bytes for a key and the next 16 bytes for an initialization vector.*
  * Use any portion of the hash as the key, then re-hash any portion of the hash to create the remainder.
  * Or use a combination of these techniques in any amount of iterations. This can add computational overhead which will make brute forcing a password harder, should the remainder become known to an attacker.
3. Send the remainder to the server as the user's password.
4. Generate a public/private key pair. *Tunnelgram generates a 1024 bit RSA key pair.*
5. Encrypt the private key with the encryption key it derived from the password hash using a symmetric encryption algorithm. *Tunnelgram uses AES-256 (14 rounds) in Output Feedback mode, with the additional bytes from the hash as the initialization vector.*
6. Send the encrypted private key and clear text public key to the server.

> :information_source: Some notes:
> * If you allow your users to use a weak password, then you might as well not encrypt anything, cause it's not secure.
> * Even though the password your server receives is already hashed, you should still hash it before you store it. Be overly cautious and keep your users secure.
> * If you just split apart your users' passwords and use one portion to encrypt and the other to authenticate, then you might as well just have them enter two passwords, cause that's essentially what you're doing. (Yeah, there are products that do this.)
> * Provide 2 factor auth. It is way more secure than just a password, and you can use it along with Tunnelwire.

### Upon Login

When a user logs in, the client must:

1. Complete steps 1-3 of the registration process in order to authenticate the user.
2. Retrieve the user's encrypted private key and clear text public key.
3. Use the key taken from the password hash to decrypt the private key.

> :information_source: The server never knows their actual password, so it can't decrypt their private key. Therefore, the server also never knows their private key. Boom, Tunnelwire.

## Using the Tunnelwire Encryption Scheme in Your Own Project

The Tunnelwire Encryption Scheme is licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/legalcode). You are free to use the scheme in your own software, provided you abide by the terms of the license.

In order to comply with the license, you must attribute the creator, Hunter Perrin, along with a license notice, and a link to the Tunnelwire Encryption Scheme.

You may use the following HTML:

```html
This software uses an encryption scheme based on the <a href="https://github.com/hperrin/tunnelgram#the-tunnelwire-encryption-scheme">Tunnelwire Encryption Scheme</a> by Hunter Perrin, which is licensed under the <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0 license</a>.
```

Or Markdown:

```md
This software uses an encryption scheme based on the [Tunnelwire Encryption Scheme](https://github.com/hperrin/tunnelgram#the-tunnelwire-encryption-scheme) by Hunter Perrin, which is licensed under the [CC BY 4.0 license](https://creativecommons.org/licenses/by/4.0/).
```

Or text:

```
This software uses an encryption scheme based on the Tunnelwire Encryption Scheme (https://github.com/hperrin/tunnelgram#the-tunnelwire-encryption-scheme) by Hunter Perrin, which is licensed under the CC BY 4.0 license (https://creativecommons.org/licenses/by/4.0/).
```

# How Tunnelgram Uses the Tunnelwire Encryption Scheme to Send Messages

## Upon Message Send

The client:

1. Generates a random, cryptographically secure 32 byte message key and 16 byte initialization vector, and encrypts the message with it using AES-256 (14 rounds) in Output Feedback mode.
2. Retrieves the public keys of all of the recipients.
3. Encrypts the message key and initialization vector with each of the recipients' public keys and the user's public key using RSA.
4. Sends the encrypted message and all of the encrypted copies of the message key to the server.

## Upon Message Receipt

The client:

1. Retrieves their copy of the encrypted message key and initialization vector.
2. Uses the user's private key to decrypt the message key and initialization vector.
3. Uses the message key and initialization vector to decrypt the message.

# How Tunnelgram Sends Images

Images are fairly easy. An HTML canvas element is used to size the image down until it is under 2MB, which is small enough to decrypt quickly and big enough to hold a good quality image. The message key is used to encrypt the image data in a web worker. A thumbnail is generated in the same way as well, and encrypted with the same key.

# How Tunnelgram Sends Videos

Videos are much harder. The closest thing to a universal format is H.264 video and AAC or MP3 audio in an MP4 container. VP9 and Vorbis in a WebM container would be perfect, if Apple supported it. Since many people use iPhones, that won't work. Android will record an MP4, so it can just be encrypted and sent, as long as it's under the arbitrary 20MB limit. I've found this limit is a good tradeoff in video quality/length to decryption time. But iPhones record H.264/AAC in a QuickTime container that can't be played on Android devices. Other devices may record in completely different formats/containers. Or the user may add a video well above the 20MB limit.

A normal messenger app handles this easily. It remuxes or transcodes the video on the server side. Bing bang boom, super compatible video, easy peasy. Tunnelgram can't do that, because *the video is encrypted before it's sent to the server*. The video needs to be made compatible before it's encrypted on the client. So Tunnelgram uses a custom version of FFMPEG compiled to WebAssembly called [FFMPEG.js](https://github.com/hperrin/ffmpeg.js). It loads the video into an ArrayBuffer and passes it to a Web Worker, which downloads and compiles the FFMPEG code in the browser then uses it to remux/transcode the video. For transcoding, it will use 1.5x the original bitrate or the maximum bitrate it can to get a video under 20MB. It uses a two pass transcoding strategy to get the best quality it can at this small file size.

Using this strategy, Tunnelgram can send a video from any client device that can be viewed on any other device, regardless of the type or size of the original video.

# Developing for Tunnelgram

If you'd like to work on Tunnelgram, follow these steps to get it up and running on your system:

1. Install [Docker](https://store.docker.com/search?type=edition&offering=community). You also need Docker Compose, which is included in the Community Edition.
2. Now run these commands:
  ```sh
  git clone https://github.com/hperrin/tunnelgram.git
  cd tunnelgram
  ./run.sh
  ```
3. Go to http://localhost:8080/

To rebuild on file changes, in the `app` directory:

```sh
npm run watch
```

# What's Next

Some other things I'd like to do to make Tunnelgram even more secure:

* Digital signatures.
  * Sign messages on send.
  * Save other user's public keys, and verify signatures with the stored keys.
* Key fingerprints.
  * Provide key fingerprints so user's can verify another user's public key when starting a conversation for the first time.
* User blocking.

# About Tunnelgram

Tunnelgram is written by me, Hunter Perrin. I wrote Tunnelgram because I wanted a better end to end encrypted messenger. I don't like Facebook or Google having all of my messages, especially the private ones.

None of my friends would use other end to end encrypted messengers, because they're either not easy to setup or not convenient to use.

For example, when you get a new phone, all of your encrypted conversations on Telegram or Facebook Messenger are inaccessible. Or if you're talking with someone on your phone in an encrypted chat, then you start using the web client on your desktop, you can't continue those chats.

This is a huge barrier to average users adopting an encrypted messenger. The goal of Tunnelgram is to break down that barrier, so anyone can use an encrypted messenger easily.

If you'd like to find me:

* Follow me on Twitter at https://twitter.com/SciActive
* Email me at hunter@sciactive.com.
* Find me on GitHub at https://github.com/hperrin and https://github.com/sciactive.
* Or send me a message on Tunnelgram at http://tngm.me/hperrin.
