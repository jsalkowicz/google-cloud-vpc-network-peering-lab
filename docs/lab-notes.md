# Detailed Lab Notes — VPC Network Peering

## Task 1A — Checked the VPC Subnet Ranges

**Action:** Reviewed the subnet ranges for `mynetwork` and `privatenet` before setting up VPC peering.

**Observation:** `mynetwork` was using addresses from the `10.x.x.x` range, while `privatenet` had two subnets:

- `privatesubnet-us` — `172.16.0.0/24`
- `privatesubnet-notus` — `172.20.0.0/24`

The subnet ranges did not overlap.

**Evidence**

![mynetwork subnet ranges](../evidence/01-mynetwork-subnet-ranges.png)

![privatenet subnet ranges](../evidence/02-privatenet-subnet-ranges.png)

**What I learned:** Before two VPCs can be peered, their subnet ranges have to be separate. If the networks use overlapping addresses, the peering cannot be set up correctly because the same IP range could exist on both sides.

**Why it mattered:** This was the first thing I needed to check before connecting the networks. Good IP planning matters because overlapping address space can prevent networks from being connected later.

---

## Task 2 — Configure VPC Network Peering

### Task 2A — Started the Peering Connection

**Action:** Opened the VPC Network Peering page and started creating the connection between `mynetwork` and `privatenet`.

**Observation:** There were no active peering connections yet, so the two VPCs were still separate.

![VPC peering before connection](../evidence/03-vpc-peering-before-connection.png)

**What I learned:** VPC peering does not happen automatically just because two networks are in the same project. The connection has to be created and configured.

**Why it mattered:** This gave me a clear starting point before I connected the two networks.

### Task 2B — Created Both Sides of the Peering

**Action:** Created the two peering configurations:

- `peering-1-2`: `mynetwork` → `privatenet`
- `peering-2-1`: `privatenet` → `mynetwork`

**Observation:** After I created both entries, they appeared on the VPC Network Peering page but initially showed as **Inactive** while Google Cloud updated the connection state.

![Both peering entries inactive](../evidence/04-peering-created-inactive.png)

**What I learned:** Each VPC keeps its own side of the peering configuration. Both sides have to exist and match before the overall peering relationship can become active.

**Why it mattered:** This showed me that VPC peering is a two-sided connection. Creating only one side is not enough to connect the networks.

### Task 2C — Verified the Peering Became Active

**Action:** Refreshed the VPC Network Peering page after both sides had been created.

**Observation:** Both `peering-1-2` and `peering-2-1` changed to **Active**.

![Both peerings active](../evidence/05-peering-active.png)

**What I learned:** The peering relationship only becomes active after both VPCs have matching peering configurations.

**Why it mattered:** Once both sides were active, the two VPCs were ready to exchange private traffic using their internal IP ranges.

---

## Task 3A — Verified Peering Routes

**Action:** Opened the Routes page after the VPC peering connection became active and filtered the routes for `mynetwork`.

**Observation:** Google Cloud automatically added peering routes for the `privatenet` subnet ranges:

- `172.16.0.0/24`
- `172.20.0.0/24`

Both routes showed `peering-1-2` as the next hop.

![Peering routes](../evidence/06-peering-routes.png)

**What I learned:** Once VPC peering becomes active, Google Cloud automatically adds routes that let each network reach the subnet ranges in the other VPC.

**Why it mattered:** These routes are what make private IP communication possible between the two networks. Without a route to the other VPC's subnet, the traffic would have nowhere to go.

---

## Task 3B — Tested Private Connectivity from `mynetwork` to `privatenet`

**Action:** Connected to `mynet-us-vm` and pinged the internal IP address of `privatenet-us-vm`.

**Observation:** The ping worked after the VPC peering connection was active. All three packets were received with `0% packet loss`.

![Private ping from mynetwork to privatenet](../evidence/07-private-ping-mynetwork-to-privatenet.png)

**What I learned:** Once the peering was active, `mynet-us-vm` could reach `privatenet-us-vm` using its internal IP address.

**Why it mattered:** This confirmed that the peering connection created working private connectivity between the two VPCs without needing to use the VM's external IP address.

---

## Task 3C — Tested Private Connectivity in the Other Direction

**Action:** Connected to `privatenet-us-vm` and pinged the internal IP address of `mynet-us-vm`.

**Observation:** The ping worked in the opposite direction too. All three packets were received with `0% packet loss`.

![Private ping from privatenet to mynetwork](../evidence/08-private-ping-privatenet-to-mynetwork.png)

**What I learned:** The peering connection allowed private communication between the two VPCs in both directions.

**Why it mattered:** Testing the return path helped confirm that the peering was working both ways and that the first successful ping was not just a one-way result.

---

## Task 3D — Tested Internal DNS Across the Peering

**Action:** From `privatenet-us-vm`, I tried to reach `mynet-us-vm` by hostname instead of using its internal IP address.

**Observation:** The hostname lookup failed with:

```text
Name or service not known
```

The internal IP was reachable, but the VM name did not resolve across the peered network.

![DNS lookup failure across peering](../evidence/09-internal-dns-across-peering-fails.png)

**What I learned:** VPC peering gives the networks a private IP path to each other, but it does not automatically make Compute Engine internal DNS names available across the peering connection.

**Why it mattered:** This showed me that working network connectivity and working DNS are two different things. A system can be reachable by IP even when its hostname does not resolve.
