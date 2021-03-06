# system-dhcp

Manage DHCP configuration for DNSMasq.

## read
### Input

It takes a mandatory `action` argument.

#### list

List all available configurations.

Example:
```json
{
  "action": "list"
}
```

#### scan

Scan all available hosts on a specific network interface. `arp-scan` sends ARP packets to hosts on the local network and displays any responses that are received. 

Example:
```json
{
  "action": "scan",
  "nic": "eth0"
}
```
### Output

#### list
Inside the `configuration` field return:

- `reservations`: all configured reservation records from the `hosts`
- `ranges`: the list of green and blue interfaces with relative DHCP configuration.
  If DHCP is not configured for an interface, the `status` prop is set to `disabled`,
  `DhcpRangeEnd` and `DhcpRangeStart` are set to a reasonable default.

```json
{
  "status": [
    {
      "ip": "192.168.1.20",
      "mac": "xx:xx:xx:xx:xx:xx",
      "expire": 1539081950,
      "name": "myhost",
      "client": "xxxxxxxxxxxxxxx"
    },

    ...
    ],
  "configuration": {
    "ranges": [
      {
        "name": "br0",
        "type": "range",
        "props": {
          "DhcpTFTP": "",
          "status": "disabled",
          "DhcpDNS": "",
          "DhcpDomain": "",
          "DhcpLeaseTime": "",
          "DhcpRangeStart": "192.168.1.1",
          "DhcpWINS": "",
          "DhcpRangeEnd": "192.168.1.254",
          "DhcpGatewayIP": "",
          "DhcpNTP": ""
        }
      }
    ],
    "reservations": [
      {
        "name": "mypc",
        "type": "local",
        "props": {
          "MacAddress": "xx:xx:xx:xx:xx:xx",
          "IpAddress": "192.168.1.104",
          "Description": "My PC",
          "type": "local"
        }
      },
    ...
    ]
  }
}
```

#### scan 

Scan the hosts on the network

- `ip`: The IP address of the host
- `mac`: The mac address of the host 
- `name`:The vendor description of the host network interface
- `reserved`: if true an IP reservation has been made in the DHCP server.
- `host`: Display the hostname if an IP reservation has been made in the DHCP server, else set `null`.

```json
[
  {
    "ip": "192.168.56.1",
    "mac": "0a:00:27:00:00:00",
    "name": "(Unknown)",
    "reserved": true,
    "host": "plop.local"
  },
  {
    "ip": "192.168.56.2",
    "mac": "08:00:27:f0:78:c1",
    "name": "PCS Systemtechnik GmbH",
    "reserved": false,
    "host": null
  }
]
```

## validate

### Constraints

The request must contain an `action` field. Valid actions are:

- `create-reservation`
- `update-reservation`

Constraints for `create-reservation`

- IpAddress: must be a valid IP address
- MacAddress: must be a valid mac address
- name: muse be simple hostname
- all parameters are also validated against "dhcp-reservation" platform validator

Constraints for `update-reservation`:

- name: must be unique inside the hosts db
- same constraints from create-reservation

Constraints for `update-range`:

- name: must be an existing network interface
- DhcpRangeStart: a valid IP, lower than maximum range and lower than DhcpRangeEnd
- DhcpRangeEnd: a valid IP, greater than minimum range and greater than DhcpRangeStart
- DhcpLeaseTime: positive integer or empty
- DhcpDomain: domain or empty
- DhcpDNS, 'DhcpWINS, DhcpNTP, DhcpTFTP: a valid IP or empty


### Input

#### create-reservation

Example:
```json
{
  "action": "create-reservation",
  "IpAddress": "192.168.1.20",
  "MacAddress": "xx:xx:xx:xx:xx:xx",
  "name": "myhost",
  "Description": ""
}
```

#### update-reservation

Example:
```json
{
  "action": "update-reservation",
  "IpAddress": "192.168.1.20",
  "MacAddress": "xx:xx:xx:xx:xx:xx",
  "name": "myhost",
  "Description": ""
}
```

#### update-range

Example:
```json
{
  "action": "update-range",
  "DhcpRangeStart": "192.168.1.10",
  "DhcpRangeEnd": "192.168.1.50",
  "name": "br0",
  "status": "enabled",
  "DhcpGatewayIP": "",
  "DhcpLeaseTime": "",
  "DhcpDomain": "",
  "DhcpDNS": "",
  "DhcpWINS": "",
  "DhcpNTP": "",
  "DhcpTFTP": ""
}
```

## update

### update-reservation

Use the same input from validate.

### update-range

Use the same input from validate.

## create

### create-reservation

Use the same input from validate.

## delete

### delete-range

Delete the given range. The `name` field must contain the interface name.

Input example:
```json
{
    "name": "br0"
}
```


### delete-reservation

Delete the given reservation. Pass the the key to be deleted inside the `name` field.

Example:
```json
{
    "name": "myhost"
}
```
