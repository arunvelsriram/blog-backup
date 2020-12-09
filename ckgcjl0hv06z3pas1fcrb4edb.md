## You Don't Need to Type a Lengthy SSH Command

1. With `ssh` we often deal with lengthy domain names and plain IP addresses. To `ssh` easily we usually create short aliases by adding entries to `/etc/hosts`. This can be done using `~/.sshconfig` itself:

    ```
    Host my-server-1
       Hostname 192.168.1.10

    Host my-server-2
       Hostname my-lenghthy-domain-name.example.com
    ```

    Now,

    To access `192.168.1.10`:

    ```bash
    ssh user@my-server-1
    ```

    To access `my-lenghthy-domain-name.example.com`:

    ```bash
    ssh user@my-server-2
    ```

2. In [SSH via Jump Server in One Step](/posts/ssh-via-jump-server-in-one-step/) I shared about ssh-ing via jump servers in one step using the `-J` option. This too can be configured in `~/.sshconfig`.

    To access `192.168.1.10` that is accessible only through `192.168.1.2` we would do:

    ```bash
    ssh -J user@192.168.1.2 user@192.168.1.10
    ```

    With the following in `~/.sshconfig`:

    ```
    Host 192.168.1.10
       ProxyJump 192.168.1.2
    ```

    We could just do:

    ```bash
    ssh user@192.168.1.10
    ```

3. Similarly, many other configs like username, key filename could be pushed to `~/.sshconfig`:

    ```
    Host my-server
       Hostname 192.168.1.10
       ProxyJump 192.168.1.2
       User foo
       IdentityFile ~/foo.pem

    Host my-jump-server
       Hostname 192.168.1.2
       User bar
       IdentityFile ~/bar.pem
    ```

    Now just by doing `ssh my-server` we will have access to `192.168.1.10`.

Pushing these configurations to `~/.sshconfig` will be very helpful if you `ssh` into many machines often. We could also share this with other members of the team easily.

We could also auto-generate the configuration as part of our infrastructure automation. For example, we could make a  [terraform](https://www.terraform.io/)  code that spawns VMs to provide this configuration as  [output](https://www.terraform.io/docs/configuration/outputs.html) .

