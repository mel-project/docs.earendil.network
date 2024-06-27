# Running 1+ nodes on 1 machine

Running more than 1 Earendil node on the same machine is quite easy:

1. Create a separate config file for each node. Be sure to add the following 3 fields, with different values, to each config file:

   ```yaml
   # where to store this Earendil daemon's database
   db_path: /your/path/node1.db

   # listen port for this daemon's control protocol
   control_listen: 127.0.0.1:<node1_control_port>

   # listen port for socks5 proxy
   socks5:
     listen: 127.0.0.1:<node1_socks5_port>
     fallback: pass_through
   ```

   Manually specifying these fields is necessary when you're running more than 1 Earendil node on the same machine, because otherwise the different nodes will try to use the same default value. Two daemon cannot use the same database or listen to the same port!

2. Start a separate daemon for each node using the config files.
