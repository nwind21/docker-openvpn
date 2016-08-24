# OpenVPN for Docker

This is really a hacked up version of Kyle Manna's work in OpenVPN but changed to use TAP interfaces to support broadcast packets.

* Docker Registry @ [kylemanna/openvpn](https://hub.docker.com/r/kylemanna/openvpn/)
* GitHub @ [kylemanna/docker-openvpn](https://github.com/kylemanna/docker-openvpn)

OpenVPN server in a Docker container complete with an EasyRSA PKI CA.

As mentioned, the core of this OpenVPN is extensively tested on [Digital Ocean $5/mo node](http://bit.ly/1C7cKr3) and has a corresponding [Digital Ocean Community Tutorial](http://bit.ly/1AGUZkq).

## Quick Start

* Create the `$OVPN_DATA` volume container, i.e. `OVPN_DATA="ovpn-data"`

        docker run --name $OVPN_DATA -v /etc/openvpn busybox

* Initialize the `$OVPN_DATA` container that will hold the configuration files and certificates

        docker run --volumes-from $OVPN_DATA --rm nwind21/docker-openvpn ovpn_genconfig -u udp://VPN.SERVERNAME.COM
        docker run --volumes-from $OVPN_DATA --rm -it nwind21/docker-openvpn ovpn_initpki

* Start OpenVPN server process

        docker run --volumes-from $OVPN_DATA -d -p 1194:1194/udp --cap-add=NET_ADMIN nwind21/docker-openvpn

* Generate a client certificate without a passphrase

        docker run --volumes-from $OVPN_DATA --rm -it nwind21/docker-openvpn easyrsa build-client-full CLIENTNAME nopass

* Retrieve the client configuration with embedded certificates

        docker run --volumes-from $OVPN_DATA --rm nwind21/docker-openvpn ovpn_getclient CLIENTNAME > CLIENTNAME.ovpn

## Debugging Tips

* Create an environment variable with the name DEBUG and value of 1 to enable debug output (using "docker -e").

        docker run --volumes-from $OVPN_DATA -p 1194:1194/udp --privileged -e DEBUG=1 nwind21/docker-openvpn

* Test using a client that has openvpn installed correctly

        $ openvpn --config CLIENTNAME.ovpn

* Run through a barrage of debugging checks on the client if things don't just work

        $ ping 8.8.8.8    # checks connectivity without touching name resolution
        $ dig google.com  # won't use the search directives in resolv.conf
        $ nslookup google.com # will use search

## How Does It Work?

Refer to the Kyle Manna upstream README.  Aspects of how this container is built doesn't really differ much.

## OpenVPN Details

We use `tap` mode because we need to send broadcast packets.  `tun` mode does not support this.

The topology used is `net30`, because it works on the widest range of OS.
`p2p`, for instance, does not work on Windows.

The UDP server uses`10.200.100.0/24` for dynamic clients by default.

This client profile DOES NOT specify `redirect-gateway def1` unlike the parent container.
