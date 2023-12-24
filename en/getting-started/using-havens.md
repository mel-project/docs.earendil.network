# Using and hosting havens

**Havens** are anonymously hosted services, similar to [onion services in Tor](https://community.torproject.org/onion-services/). By hosting a haven, you can serve a TCP or UDP service, like a website, IRC server, or similar. Both you and your users will then be protected by Earendil's anonymity guarantees.

This tutorial will teach you how to use and host basic Earendil havens.

## Using havens

You can access HTTP-based havens right in your browser:

1.  In your existing config file, set up Earendil as a SOCKS5 proxy by adding this SOCKS5 section to your Earendil config file. It should look something like this:

    ```yaml
    socks5:
      listen: 127.0.0.1:23456 # localhost port where the proxy server listens
      fallback: pass_through # how to handle non-earendil traffic
    ```
2. `fallback` specifies how to handle non-Earendil traffic (like a request to tunnel `google.com:443`). There are 3 options:
   * `pass_through`: let all non-Earendil traffic through as if you're not using Earendil. Requests to `google.com` will behave the same way as if you weren't connected to the Earendil proxy.
   * `block`: block all non-Earendil traffic. Requests to `google.com` will fail.
   * `simple_proxy`: proxy non-Earendil traffic via a specified `simple-proxy` server, similar to how you use Tor as a web proxy. The next tutorial will cover this use case.
3.  Now, set your browser to use `localhost:23456` as a SOCKS5 proxy.

    For Firefox this looks like:

    ![image](https://hackmd.io/\_uploads/SkLZ828Sp.png)
4.  Visit

    ```!
    http://qcmnt2mbchhanm7fzacybswzknbsw3zp.haven:12345
    ```

    like you would any ordinary website. You should be greeted with:

    ![image](https://hackmd.io/\_uploads/rJMmF3LHT.png)

You just visited your first Earendil haven! With this setup, you can visit any Earendil haven you know the address to.

Note that all Earendil haven websites are HTTP only, since certificate authorities generally do not issue certificates to `.haven` domains. HTTPS is unnecessary because Earendil traffic is already encrypted.

## Hosting havens

As an introduction to hosting havens, let's first host a website as a haven.

### Starting a localhost web server

The first step is to **set up a web server that listens on port 8000**. For our example, we'll host a small Nginx webserver.

1. Install Nginx if it's not already installed. On Debian Linux, you would type `sudo apt install nginx`, but this differs from system to system.
2. In the nginx config file (most likely located at `/etc/nginx/nginx.conf`), look for a section that configures a server listening on 8000, and change that to the example below:

```
server {
    listen       8000;
    server_name  localhost;

    location / {
      root /usr/share/nginx/html;
      index index.html;
    }
}
```

3. Then, start your Nginx server. On Linux, use `systemctl start nginx`
4. You should now be able to see your Nginx webserver on `localhost:8000`!

### Setting up the haven

1. Add this `haven` block to your relay's config file, replacing `your_random_seed` with the output of step 2:

```yaml
# havens we're hosting
havens:
  - identity_file: /your/path/identity.secret # replace with a writable path for storing identity secret
    rendezvous: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz # our chosen rendezvous relay
    handler:
      type: tcp_service
      listen_dock: 12345
      upstream: 127.0.0.1:8000 # where web server is listening
```

* `identity_file`: a writable path for storing your haven's identity secret
* `rendezvous` is the fingerprint of your chosen _rendezvous relay_. This is a relay node that is responsible for receiving and forwarding all the messages meant for your haven, so that your IP address can be kept private from clients of your haven. All havens must have a rendezvous relay; you can read more about the haven protocol's architecture [here](https://docs.earendil.network/wiki/protocols/haven-protocol). For now, we’ll use the same test relay that we bootstrapped with throughout this tutorial: `ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz`.
* `handler` specifies how to handle traffic to the haven. Here, we use TCP [port forwarding](https://en.wikipedia.org/wiki/Port\_forwarding) to forward all haven traffic to the web server on port 8000.

2. Restart the `earendil` daemon 
3. Print out your haven's address with

```shell-session
earendil control havens-info
```

You should see something like:

```
TcpForward - qcmnt2mbchhanm7fzacybswzknbsw3zp:12345
```

which tells you that a `tcp_service` haven listening at **dock** `12345` (the equivalent of a port number within Earendil) has fingerprint qcmnt2mbchhanm7fzacybswzknbsw3zp.

People can now find your haven at `qcmnt2mbchhanm7fzacybswzknbsw3zp.haven:12345`!
