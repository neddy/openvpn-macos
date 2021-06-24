# OpenVPN Server and Client Set Up on MacOS

## References

[OpenVPN 2x HOW TO](https://openvpn.net/community-resources/how-to/)

[Easy-RSA v3 OpenVPN Howto](https://community.openvpn.net/openvpn/wiki/EasyRSA3-OpenVPN-Howto?)

## Required Software

### Tunnelblick

Download and install Tunnelblick from the website: [Tunnelblick](https://tunnelblick.net/index.html)

### Easy RSA

Install Easy RSA using [Homebrew](https://brew.sh/)

```bash
brew install easy-rsa
```

## Generate the Certificates and Keys with Easy RSA

Start new PKI

```bash
easyrsa init-pki
```

Build CA

```bash
easyrsa build-ca
```

Generate Server Key and Certificate

```bash
easyrsa build-server-full UNIQUE_SERVER_SHORT_NAME
```

Generate Client Key(s) and Certificate(s)
```bash
easyrsa build-client-full UNIQUE_CLIENT_SHORT_NAME
```

*Note: inline auth files can be generated using [Easy-TLS](https://github.com/TinCanTech/easy-tls), details can be found at the bottom of the [Easy-RSA v3 OpenVPN Howto](https://community.openvpn.net/openvpn/wiki/EasyRSA3-OpenVPN-Howto?) page.*

Generate Diffie Hellman (DH) parameters

```bash
easyrsa gen-dh
```

Generate ta.key for tls-auth (optional security hardening)

```bash
openvpn --genkey --secret ta.key
```

## Copy Certificates to Folders for OpenVPN

Create folders: one for the server, and one for each of the client's configurations. Give each client config a unique name, these should match the names of the certificates generated for each client.

Then copy the following files from the pki folder to each of the following:

*Note: Easy-rsa created the pki folder at this location on my Mac. Some of the files are in subfolders*
```
/usr/local/etc/pki/
```

Server:
  - ca.crt
  - dh.pem
  - ta.key
  - server.crt
  - server.key

Client:
  - ca.crt
  - ta.key
  - client.crt
  - client.key 


## Create OpenVPN Configuration Files

Download the sample config files from here: [OpenVPN Sample Config Files](https://github.com/OpenVPN/openvpn/tree/master/sample/sample-config-files)

### Server Config

Make a copy of the `server.conf` file and give it the same name as the server, and copy it to the server config folder.

Next, edit the new server config file and modify the following (for a start):

  - Edit the file so that the ca, cert, key, dh and ta parameters point to the correct files (no path is required as these will be distributed in the config folder.)
  - Uncomment `user nobody` and `group nobody` unless you have Windows users connecting.
  - Add the local IP of the server, replace the `a.b.c.d` on this line:  `local a.b.c.d` (you should also set a static IP for the server).
  - Uncomment `topology subnet`
  - Uncomment `tls-auth ta.key 0`

*Note: For more advanced setup, such as allowing VPN clients to connect to other machines on the network other than the server, please see the [OpenVPN 2x HOW TO](https://openvpn.net/community-resources/how-to/) reference.*


### Client Config

Make a copy of the `client.conf` file and give it the same name as the client, and copy it to the client config folder. You will need to make one folder for each client.

Next, edit the new client config file and modify the following (for a start):

  - Edit the file so that the ca, cert, key, dh and ta parameters point to the correct files (no path is required as these will be distributed in the config folder.)
  - Edit the remote address, replacing `my-server-1` with the server address on this line `remote my-server-1 1194`, also make sure you edit the line that is not commented.
  - Uncomment `user nobody` and `group nobody` unless you have Windows users connecting.
  - Uncomment `tls-auth ta.key 1`


## Forward Port for OpenVPN and Set Static IP for OpenVPN Server

Login to your router and set a static IP for your OpenVPN server and forward port 1194 UDP to the OpenVPN server. You will need to Google if you don't know how to do this.

## Distribute OpenVPN Config Packages for Tunnelblick

We will be using Tunnelblick for both the server and client.

To import the config files and certificates into Tunnelblick, make a copy of the config folders created above and then at the extension `.tblk` to the folder, this will associated in with Tunnelblick and allow you to double click to install the config. Do this for the server and the clients, then distribute the packages to each, and double click to install (Tunnelblick will need to be installed on all machines).

## Test the connection

Once you have imported the configs, then in Tunnelblick on the server, click connect on the 'server' config, then do the same on the clients. If you need to troubleshoot anything, open Tunnelblick by clicking 'VPN Details' under the Tunnelblick icon in the system tray, and then check the log for the connection.

**That's it, you should now have a working OpenVPN setup.**

*Note: You will need to configure the server to start automatically if you want it to always run.*
