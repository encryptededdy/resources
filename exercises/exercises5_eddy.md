# Network control plane: Review exercises

> **NB**
> This sheet is new: If you notice any errors or omissions, please do make contact via email/Piazza or send a [pull request](https://help.github.com/articles/about-pull-requests/).

## 5.1. Introduction

1. What is the role of the network layer? What does we call the **protocol data unit** (PDU) associated with the network layer? (With the transport layer?)

    Transport layer **segments** are encapsulated in network layer **packets**. The network layer handles the routing/switching of traffic between clients and server.

2. Which **planes** comprise the transport layer. Which is "on top"?

    The **data plane**, underpinned by the **control plane**. The data plane handles moving data between physical interfaces of a router, whereas the control plane handles working out the path that the data actually needs to take (and hence controlling the routing that happens on the data plane).

3. What distinctions might someone make between **forwarding** and **routing**? Can you think of nice analogies?

    Forwarding involves sending a packet to a destination based on precomputed/supplied information (e.g. a switch just looks at the destination MAC and forwards the packet to the appropriate iface), whereas routing requires actually computing the path for the packet to take based on destn IP.

4. How are **forwarding tables** (FIB) likely to differ from **routing tables** (RIB)?

    Forwarding table stores MAC -> Interface, whereas Routing tables store IP -> Interface (which requires more computation, using routing algorithms).

---

## 5.2. Routing Algorithms

1. What are the constituent parts of a **graph**? Recall alternative terms for each constituent.

    Nodes (routers), edges (verticies, links)

2. What is a **path** in a graph? What is the **length** of a path? What is the **cost** of path? What is a **least-cost** path? In what sense is a **shortest path** a **least-cost** path? Are least-cost paths **unique**?

3. Describe how routing protocols may be characterised/classified with respect to:
    - communication pattern
    - awareness for network **topology**
    - awareness for link **congestion**
    
    a. Communication Pattern: ??? *is this SDN vs. traditional?*
    
    b. Network Topology awareness: Link-state RP knows the entire network topology, whereas distance vector RPs only need to know cost to neighbours
    
    c. RPs which update weights based on congestion can be considered more dynamic. Ones that don't really change link weights based on congestion can be considered more static.
    

4. **Link costs** are also called **administrative weights**: Discuss this terminology.

    Because an network admin can define a policy, and thus the link costs can depend on administrative policy

5. Must link costs be symmetric (i.e. `c(u, v) == c(v, u)`)? Discuss.

    ??? Examples explored in class are (all undirected graphs), but this doesn't appear to be a requirement? 

6. Under what condition(s) is a complete set of least-cost paths guaranteed to exist?

    If it's a complete graph then surely a set of least-cost paths must exist.

7. What are **routing loops**? Do routing loops actually give rise to infinite cycling? Discuss.

    In the case of Distance Vectors, due to the slow propogation of information, a packet can be passed in a loop continiously because each router **thinks** a different path is optimal. In reality this doesn't result in inf cycling because the packet's TTL would expire.

### 5.2.1. LS Routing

1. Recall examples of real LS protocols.

    OSPF (Open Shortest Path First)

2. What does **link-state** refer to in "LS routing algorithm/protocol?

    Requires knowledge of every link in the network in order to calculate the path

3. What is the purpose of a **link-state broadcast**? Why "broadcast"?

    To distribute information about link costs. Broadcast because it is sent to every router on the network.

4. What steps would be involved in a **link-state broadcast**?

    Collect information about the costs of links adjacent to you, then broadcast said information to the rest of the network.

5. What is the correct pronunciation of **Dijkstra**?

    *why is this a question*

6. What inputs does Dijkstra's algorithm require? What outputs does it produce?

    A list of nodes, and a list of edges and their associated weights

7. What other data structure is required internally?

    a. A set of nodes for which the minimal cost path has already been determined

    b. A set of estimated costs to get to the rest of the nodes

8. Describe the initialization phase of Dijkstra's algorithm.

    For nodes adjacent to the start node, write the costs to them to the set of estimated costs. Make the estimates for the rest of the nodes infinite.

9. Describe each step of the "main loop". How are ties resolved?

    Add the lowest cost estimate to the finalised set. Then, using this, re-calculate the costs for everything else. The way ties are broken doesn't matter.

10. How many steps does (vanilla)) algorithm require to solve the single-source least-cost path problem? Justify the claim of **quadratic complexity**.

    n^2, where n is the number of nodes. We go through n nodes and it takes n time (in a basic implementation) to find the minimum next node to access. Relaxation then takes n time.

11. How might this complexity be reduced?

    A different data structure to store the estimated costs, like a priority queue can reduce the time it takes to find the minimum next node below n time.

12. Describe how the **density** of the graph affects run-time?

    The denser the graph, the longer relaxation takes. This doesn't affect big-O run time (as that's worst case).

13. Which parts of the LS algorithm involve inter-node communication?

    Acquiring the costs of each link

14. How would we compute the **forwarding table** from the outputs of Dijkstra's algorithm?

    Checking the "next hop" from the source, for the path to each destination (we can do this by going through the predecessor list)

15. Can a link-state scheme produce **routing loops**? Discuss.

    No, assuming all nodes are updated simultaneously, using the same information.

### 5.2.2. DV Routing

1. Recall examples of real DV protocols.

    BGP

2. In "**distance-vector** (DV) protocol/algorithm", what does "distance-vector" refer to?

    ?? We work with vectors of distances between nodes?

3. How does the DV algorithm we have discussed relate to "the" [Bellman-Ford](https://en.wikipedia.org/wiki/Bellman–Ford_algorithm) algorithm?

    We are essentially computing Bellman-Ford, except each router is computing it on their own without a view of the "whole" network.

4. What inputs are required by each router in a DV protocol?

    The distance vector / distance table from it's immediate neighbours, along with knowledge of the costs of edges it's next to.

5. Recall and describe the **dynamic programming** equation? What does it characterize?

    The distance from x to y is the minimum of the distance from x to any node v, plus the distance from v to y.

6. How is the equation **relaxed** in the DV algorithm? Describe the minimization that takes place at each step.

    When we recieve updated information of the distance from our neighbouring nodes to other nodes, we update our own table based on the DP equation using this new information.

7. Why do we call the DV procedure **iterative**, **asynchronous**, **distributed**?

    a. Because many iterations of the same algorithm occur before arriving at the optimal solution.

    b. Not all nodes compute simultaneously

    c. All nodes compute their own table

8. What would happen if a link cost changed? (Why might it have changed?)

    The cost change slowly propogates through the network

9. Can the DV iteration produce **routing loops**? Explain.

    Yes. e.g. If a link were to go down and not all nodes were updated, a packet can just be constantly chucked between two routers.

10. Describe the symptom of the **count-to-infinity** problem and explain the underlying cause. Does an effective remedy exist?

    If a link is broken, the cost of the link may count (very) slowly towards infinity. There does not exist a 100% effective remedy.

### 5.2.1-2. Routing protocols

1. Contrast the efficacy of LS and DV algorithms in terms of **communication complexity**, **speed of convergence**, and **robustness**.

2. What is a **tree**? What is a **shortest-path tree**? Why is it always a tree?

3. What is the significance of the shortest path tree?

4. What do we mean by **traffic engineering**?

5. How can **oscillations** arise in **congestion-sensitive** routing? Are these peculiar to LS or DV schemes?

6. Why might this oscillation be regarded undesirable? How might it be addressed in practice?

---

## 5.3. Intra-AS Routing: OSPF

1. What is an **autonomous system** (**AS**)? What is the relationship between **ISP** and **AS**? (Are they the same?)

    An AS is a network administratored by the same adminstrator. The internet is a network of ASes. Usually an ISP is its own AS, but not necessairly.

2. How are autonomous systems addressed? Who allocates these addresses?

    ?? AS number? ICANN?

3. What explains/motivates/justifies the partitioning of the Internet into **autonomous systems**?

    You can't store all internet devices in a routing table lol

4. What does OSPF stand for? (What does this tell us?)

    Open Shortest Path First - It's a open standard, and works based on shortest path

5. What type of routing protocol does it employ?

    Link-State (Dijkstra)

6. How are OSPF messages transmitted? What does this imply?

    Over IP, as broadcast. This implies it's link state as each router shares its information with every other router.

7. How are OSPF link costs (**administration weights**) assigned?

    By the network administrator, based on ToS requirements.

8. When do OSPF routers broadcast link-state information?

    When costs change.

9. Which "advanced features" does OSPF provide?

    Secure and multiple shortest-paths allowed. Hierachical OSPF allowed.

10. What is an OSPF **area**? How do areas limit the complexity of routing?

    Routers in each area only need to have a routing table for their area. To access the rest of the AS, go through a border router to access the backbone (which is its own area)

11. How are areas identified (addressed)?

    ?? By the administrator?

12. Describe each category of router in OSPF. What is the difference between an **Area Border Router** (ABR) and a **Backbone Router** (BR)?

    Area Border router links backbone <-> area, whereas backbone links areas together and to the boundary router (to access other ASes)

---

## 5.4. Inter-AS Routing: BGP

1. "_BGP is arguably the **most important** of all the Internet protocols (the only other contender would be the IP protocol ...)_" [KR-423]: What important role does it serve?

2. How are BGP messages sent? Who/what sends BGP messages? (ASs or routers? Protocol?)

3. Contrast BGP **gateway** routers and **internal** routers? (If BGP is an **inter**-AS protocol, why mention internal routers?)

4. What kind of "addresses" appear in BGP forwarding tables?

5. What is a BGP **route**?

6. Explain the `NEXT_HOP` attribute. What is the `NEXT_HOP` if the destination subnet is contained in the current AS?

7. Explain the `AS_PATH` **attribute**. Why store the entire path? Isn't it good enough to know the next hop and destination?

8. What is **hot potato routing**? (Is it an alternative to OSPF shortest-path routing?)

9. Explain the BGP **route selection algorithm**. What is the `LOCAL_PREF` attribute? Why does Step 2 come before Step 3?

10. Describe the **routing policy** typically adopted by commercial ISPs (in the absence of negotiated agreements).

11. What is a **dual/multi-homed** ISP? Why is BGP-style routing policy particularly relevant in this case?

12. How does BGP's **path vector** routing compare with DV- and LS routing?

13. Why are different protocols used for **inter-AS** and **intra-AS** routing?

---

## 4.4. SDN: Data Plane

1. Recall examples of [**middleboxes**](https://en.wikipedia.org/wiki/Middlebox) whose functions can be performed by SDN systems.

2. In what ways does "generalized" forwarding generalize traditional **destination-based forwarding**? (Is "generalized forwarding" standard/official terminology?)

3. What happens if a packet matches more than one flow table pattern?

4. How are forwarding/flow tables populated?

5. What are forwarding devices typically referred to at L2? At L3? In SDN?

6. Packet matching fields from the [OpenFlow 1.0 Specifications](https://www.opennetworking.org/images/stories/downloads/sdn-resources/onf-specifications/openflow/openflow-spec-v1.0.0.pdf#page=22) are shown below: What do `src`, `dst`, `dl`, `nw`, and `tp` refer to? Contrast `in` and `src`.

```c++
// Fields to match against flows
struct ofp_match
{
uint32_t wildcards;           // Wildcard fields
uint16_t in_port;             // Input switch port
uint8_t dl_src[OFP_ETH_ALEN]; // Ethernet source address
uint8_t dl_dst[OFP_ETH_ALEN]; // Ethernet destination address
uint16_t dl_vlan;             // Input VLAN id
uint8_t dl_vlan_pcp;          // Input VLAN priority
uint8_t pad1[1];              // Align to 64-bits
uint16_t dl_type;             // Ethernet frame type.
uint8_t nw_tos;               // IP ToS (actually DSCP field, 6 bits)
uint8_t nw_proto;             // IP protocol or lower 8 bits of ARP opcode
uint8_t pad2[2];              // Align to 64-bits
uint32_t nw_src;              // IP source address
uint32_t nw_dst;              // IP destination address
uint16_t tp_src;              // TCP/UDP source port
uint16_t tp_dst;              // TCP/UDP destination port
};
OFP_ASSERT(sizeof(struct ofp_match) == 40); // bytes
```

7. Describe typical **counters** in OpenFlow tables.

8. How might the various types of **actions** in  OpenFlow relate to traditional network applications?

9. Which fields are not available for matching in OpenFlow 1.0? Why not?

## 5.5. SDN: Control Plane

1. What distinguishes SDN from "traditional" routing?

2. Has **unbundling** has been beneficial for consumers in another market? Discuss.

3. How do SDN-capable **packet switches** compare with "traditional" L3 **routers**?

4. Identify or describe the layers of the SDN **control plane**.

5. What kind of "state" is kept in the **network-wide state management layer**?

6. Where does OpenFlow fit into the SDN control plane?

7. Describe OpenFlow's **controller-to-switch** message types.

8. Describe OpenFlow's **switch-to-controller** message types.

9. What kind of routing protocol is implemented in OpenFlow? (LS, DV, of PV?) Discuss.

10. Identify a modern SDN controller and recall some of its features.

---

## 5.6. ICMP

1. What is the role of ICMP? How does it relate to TCP?

2. Does ICMP only report errors? Discuss.

3. How are ICMP messages specified?

4. How does `ping` work?

5. How does `traceroute` work?

6. The TTL-based implementation of `traceroute` seems inefficient: Wouldn't a _dedicated_ `traceroute` message type save on echo requests that differ only in TTL?

---

## 5.7. Network Management and SNMP

1. Identify the key components of network management and given an example or description of each

2. Describe the types of SNMP messages.

3. Which protocol carries SNMP messages? What is the implication.

4. What is the latest version of SNMP? What features did it introduce?

---

## Review

1. Which routing protocols have we discussed? How might they be classified?
