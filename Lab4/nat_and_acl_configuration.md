### ACL Configuration

> Access Control Lists (ACLs) are used to filter traffic by defining a set of rules that permit or deny packets.

#### **General ACL Configuration Steps**

1.  **Create filtering rules** using the `access-list` command.
    > This defines the actual traffic filtering criteria.
2.  **Apply the filtering rules** to an interface with the `access-group` command.
    > This activates the ACL on a specific interface.

#### **Creating Filtering Rules**

*   **For TCP/UDP:**
    ```
    asa(config)# access-list <ACL_ID> permit/deny <protocol> <src_ip_address> <src_subnet mask> <operator> <port_number> <dst_ip_address> <dst_subnet mask> <operator> <port_number>
    ```
    > This command creates an ACL entry to filter TCP or UDP traffic based on source/destination IP addresses and port numbers.

*   **For ICMP:**
    ```
    asa(config)# access-list <ACL_ID> permit/deny icmp <src_ip_address> <src_subnet mask> <dst_ip_address> <dst_subnet mask> <ICMP_message_type>
    ```
    > This command creates an ACL entry to filter ICMP traffic (e.g., ping) based on source/destination IP and ICMP message type.

#### **Applying Filtering Rules**

```
asa(config)# access-group <ACL_ID> in/out interface <interface_name>
```
> This command applies a previously defined ACL to a specific interface, either for inbound (`in`) or outbound (`out`) traffic.

#### **ACL Maintenance**

*   **View ACLs:** `show access-list`
    > Use this command to display the configured ACLs and check their entries.
*   **Remove an ACL entry:** Precede the `access-list` command with `no`.
    > This is used to delete a specific rule from an ACL.
*   **Delete an ACL:** `clear access-list <ACL_ID>`
    > This command completely removes an entire ACL.

#### **Object Grouping**

> Object grouping is a feature that simplifies ACL management by allowing you to group multiple users, devices, or protocols into a single entity.

*   **Create an object group:**
    ```
    asa(config)# object-group <type_of_object> <group_ID> [protocol_name]
    ```
    > This command creates a group of a specific type. The types can be `icmp-type`, `network`, `protocol`, or `service`.

*   **Example: Network Object Group**
    ```
    asa(config)# object-group network <group_ID>
    asa(config-network)# network-object host <host_address>
    asa(config-network)# network-object <network_address> <subnet_mask>
    ```
    > These commands create a group of network objects, which can be individual hosts or entire subnets. This group can then be used in an ACL.

### NAT Configuration

> Network Address Translation (NAT) is used to modify network address information in packet headers. It is commonly used to allow devices with private IP addresses to communicate with the internet.

#### **Dynamic NAT**

> Dynamic NAT maps a pool of private IP addresses to a pool of public IP addresses.

1.  **Configure a network object:**
    ```
    ASA(config)# object network <my_host>
    ```
    > This command creates a network object that will contain the IP addresses to be translated.
2.  **Define the IP addresses to translate:**
    ```
    ASA(config-network-object)# subnet 192.168.1.0 255.255.255.0
    ```
    > This command specifies the subnet whose IP addresses will be translated.
3.  **Configure dynamic NAT:**
    ```
    ASA(config-network-object)# nat (inside,outside) dynamic interface/<ip address>
    ```
    > This command configures dynamic NAT for the specified network object, translating the private IPs to the public IP of the outside interface.

#### **Port Translation (PAT)**

> Port Address Translation (PAT), also known as NAT overload, maps multiple private IP addresses to a single public IP address by using different port numbers.

```
ASA(config)# object network <inside-network>
ASA(config-network-object)# subnet 192.168.1.0 255.255.255.0
ASA(config-network-object)# nat (inside,outside) dynamic interface
```
> These commands configure PAT, allowing all hosts on the `inside-network` to share the public IP address of the outside interface.

#### **Static NAT**

> Static NAT creates a fixed, one-to-one mapping between a private IP address and a public IP address. This is often used for servers that need to be accessible from the internet.

*   **Direct Static NAT:**
    ```
    ASA(config)# object network my_host
    ASA(config-network-object)# host 10.1.1.1
    ASA(config-network-object)# nat (inside,outside) static 199.199.199.2 dns
    ```
    > These commands create a static mapping between the private IP `10.1.1.1` and the public IP `199.199.199.2`. The `dns` keyword enables DNS record rewriting for the translated host.

*   **Static NAT with Port Translation:**
    ```
    ASA(config)# object network my-ftp-server
    ASA(config-network-object)# host 10.1.1.1
    ASA(config-network-object)# nat (inside,outside) static interface service tcp 21 2121
    ```
    > These commands create a static mapping that translates a specific port. In this example, traffic to the outside interface on TCP port 2121 will be forwarded to the internal FTP server at `10.1.1.1` on port 21.

#### **Verifying NAT**

*   `show global`
    > Displays the global IP addresses used for NAT.
*   `show nat`
    > Displays the NAT configuration.
*   `show xlate`
    > Displays the current active address translations in the firewall.