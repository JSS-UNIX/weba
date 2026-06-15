# Get own domain-name
Virtual machines created by hosters (like Hetzner) usually create individual domain names for each VM.

## IP-Address
First step is to get the own global IP-Address:

```bash
ip addr
```
or to directly filter out the IP-Address:

```bash
ip addr show eth0 | grep inet | awk '{print $2}' | cut -d / -f 1
```

## PTR Entry
next step is to retrieve the PTR entry for the IP-Address:

```bash
dig -x *ip-addr* @8.8.8.8 +short
```

## Verify Forward DNS
last step is to verify if the Type A Record is also set properly:

```bash
dig -t A *ptr-name* @8.8.8.8 +short
```

## DNS-Name
In case the Type A Record resolves to the IP-Address of the virtual machine, it can be used as domain-name in the Browser.

```URL
http://<domain-name>
```
