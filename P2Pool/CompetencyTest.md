# P2Poolv2 Competency Test Implementation

This is my implementation for the competency test to be submitted with my Summer of Bitcoin 2025 P2Poolv2 proposal. The task is to create a mesh or random P2P network with around 10+ nodes, simulate "shares" as messages sent by each node to its peers every few seconds, and hardcode the latency. I’ve split this into two parts for better understanding and development.

---

![image](https://github.com/user-attachments/assets/a64fa1c1-d340-45ce-b521-ba7035d30eb1)


## Part 1 - Setting Up the Network

### What I Did
- Created a mesh P2P network with 10 nodes, where every node connects to every other node.
- Used NS3 to build the network and CMake to organize and build the code.

### How I Set It Up
1. **Installed Tools**:
   - I used WSL with Ubuntu on my Windows computer.
   - Installed NS3 (version 3.44), CMake, and necessary tools with:
     ```bash
     sudo apt update && sudo apt install -y build-essential gcc g++ cmake git
     ```
   - Downloaded and extracted NS3:
     ```bash
     mkdir ~/ns3
     cd ~/ns3
     wget https://www.nsnam.org/releases/ns-allinone-3.44.tar.bz2
     tar xjf ns-allinone-3.44.tar.bz2
     cd ns-allinone-3.44/ns-3.44
     ```

2. **Prepared NS3 with CMake**:
   - Made a `build` folder and set up CMake:
     ```bash
     mkdir build
     cd build
     cmake -DNS3_EXAMPLES=ON -DNS3_TESTS=ON ..
     ```
   - Built NS3:
     ```bash
     make -j4
     ```

3. **Wrote Network Code**:
   - I put the network setup in `scratch/p2poolv2-competency-test.cc`.
   - Here’s the code for the network part:

```cpp
#include "ns3/core-module.h"
#include "ns3/network-module.h"
#include "ns3/internet-module.h"
#include "ns3/point-to-point-module.h"

using namespace ns3;

int main(int argc, char* argv[]) {
    LogComponentEnable("P2Poolv2CompetencyTest", LOG_LEVEL_INFO);

    // Create 10 nodes
    uint32_t numNodes = 10;
    NodeContainer nodes;
    nodes.Create(numNodes);

    // Set up connections with placeholder latency (to be hardcoded in Part 2)
    PointToPointHelper p2p;
    p2p.SetDeviceAttribute("DataRate", StringValue("5Mbps"));
    p2p.SetChannelAttribute("Delay", StringValue("100ms")); // Placeholder latency

    // Install internet stack
    InternetStackHelper stack;
    stack.Install(nodes);

    // Connect all nodes in a mesh
    Ipv4AddressHelper address;
    address.SetBase("10.1.0.0", "255.255.255.0");
    Ipv4InterfaceContainer interfaces;
    std::vector<NetDeviceContainer> devices;

    for (uint32_t i = 0; i < numNodes; i++) {
        for (uint32_t j = i + 1; j < numNodes; j++) {
            NetDeviceContainer dev = p2p.Install(nodes.Get(i), nodes.Get(j));
            devices.push_back(dev);
            interfaces.Add(address.Assign(dev));
            address.NewNetwork();
        }
    }

    Simulator::Stop(Seconds(30.0));
    Simulator::Run();
    Simulator::Destroy();
    return 0;
}
```

- **Explanation**:  
  - This sets up 10 nodes and connects them all to each other (a full mesh).
  - I added a placeholder 100ms delay, which I’ll adjust in Part 2.
  - The `InternetStackHelper` gives the nodes internet-like rules to communicate.

### Result
- The network is ready with 10 nodes connected in a mesh. I’ll add the share-sending part next.

---

## Part 2 - Simulating Shares with Latency

### What I Did
- Added a way for each node to generate and send "shares" (messages) to its peers every 5 seconds.
- Hardcoded the latency to 0.1 seconds (100 milliseconds) for all messages.
- Included logging to track when shares are sent and received.

### How I Implemented It
1. **Updated the Code**:
   - I expanded the code in `scratch/p2poolv2-competency-test.cc` to include a `ShareApplication` for sending shares.

```cpp
#include "ns3/core-module.h"
#include "ns3/network-module.h"
#include "ns3/internet-module.h"
#include "ns3/point-to-point-module.h"
#include "ns3/applications-module.h"
#include "ns3/packet.h"
#include "ns3/simulator.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE("P2Poolv2CompetencyTest");

// Class to handle sending and receiving shares
class ShareApplication : public Application {
public:
    ShareApplication() : m_socket(0), m_nodeId(0), m_shareCount(0) {}
    virtual ~ShareApplication() { m_socket = 0; }

    void Setup(Ptr<Node> node, uint32_t nodeId, Ipv4InterfaceContainer interfaces, uint32_t numNodes) {
        m_node = node;
        m_nodeId = nodeId;
        m_interfaces = interfaces;
        m_numNodes = numNodes;
        TypeId tid = TypeId::LookupByName("ns3::UdpSocketFactory");
        m_socket = Socket::CreateSocket(m_node, tid);
        m_socket->SetAllowBroadcast(true);
        m_socket->Bind();
        m_socket->SetRecvCallback(MakeCallback(&ShareApplication::ReceiveShare, this));
    }

private:
    virtual void StartApplication() {
        Simulator::Schedule(Seconds(1.0), &ShareApplication::GenerateShare, this);
    }

    virtual void StopApplication() {
        if (m_socket) m_socket->Close();
    }

    void GenerateShare() {
        m_shareCount++;
        std::ostringstream msg;
        msg << "Share-" << m_nodeId << "-" << m_shareCount;
        Ptr<Packet> packet = Create<Packet>((uint8_t*)msg.str().c_str(), msg.str().size());
        for (uint32_t i = 0; i < m_numNodes; i++) {
            if (i != m_nodeId) {
                Ipv4Address destAddr = m_interfaces.GetAddress(i);
                m_socket->SendTo(packet, 0, InetSocketAddress(destAddr, 12345));
                NS_LOG_INFO("Node " << m_nodeId << " sent " << msg.str() << " to Node " << i << " at " << Simulator::Now().GetSeconds() << "s");
            }
        }
        Simulator::Schedule(Seconds(5.0), &ShareApplication::GenerateShare, this); // Send every 5 seconds
    }

    void ReceiveShare(Ptr<Socket> socket) {
        Ptr<Packet> packet;
        Address from;
        while ((packet = socket->RecvFrom(from))) {
            if (packet->GetSize() > 0) {
                uint8_t* buffer = new uint8_t[packet->GetSize()];
                packet->CopyData(buffer, packet->GetSize());
                std::string msg((char*)buffer, packet->GetSize());
                NS_LOG_INFO("Node " << m_nodeId << " received " << msg << " from " << InetSocketAddress::ConvertFrom(from).GetIpv4() << " at " << Simulator::Now().GetSeconds() << "s");
                delete[] buffer;
            }
        }
    }

    Ptr<Socket> m_socket;
    Ptr<Node> m_node;
    uint32_t m_nodeId;
    uint32_t m_shareCount;
    Ipv4InterfaceContainer m_interfaces;
    uint32_t m_numNodes;
};

int main(int argc, char* argv[]) {
    LogComponentEnable("P2Poolv2CompetencyTest", LOG_LEVEL_INFO);

    // Create 10 nodes
    uint32_t numNodes = 10;
    NodeContainer nodes;
    nodes.Create(numNodes);

    // Set up connections with hardcoded 100ms latency
    PointToPointHelper p2p;
    p2p.SetDeviceAttribute("DataRate", StringValue("5Mbps"));
    p2p.SetChannelAttribute("Delay", StringValue("100ms")); // Hardcoded latency

    // Install internet stack
    InternetStackHelper stack;
    stack.Install(nodes);

    // Connect all nodes in a mesh
    Ipv4AddressHelper address;
    address.SetBase("10.1.0.0", "255.255.255.0");
    Ipv4InterfaceContainer interfaces;
    std::vector<NetDeviceContainer> devices;

    for (uint32_t i = 0; i < numNodes; i++) {
        for (uint32_t j = i + 1; j < numNodes; j++) {
            NetDeviceContainer dev = p2p.Install(nodes.Get(i), nodes.Get(j));
            devices.push_back(dev);
            interfaces.Add(address.Assign(dev));
            address.NewNetwork();
        }
    }

    // Add share application to each node
    for (uint32_t i = 0; i < numNodes; i++) {
        Ptr<ShareApplication> app = CreateObject<ShareApplication>();
        app->Setup(nodes.Get(i), i, interfaces, numNodes);
        nodes.Get(i)->AddApplication(app);
        app->SetStartTime(Seconds(1.0));
        app->SetStopTime(Seconds(30.0));
    }

    // Save a record of events
    AsciiTraceHelper ascii;
    p2p.EnableAsciiAll(ascii.CreateFileStream("p2poolv2-competency-test.tr"));

    // Run for 30 seconds
    Simulator::Stop(Seconds(30.0));
    Simulator::Run();
    Simulator::Destroy();
    return 0;
}
```

- **Explanation**:  
  - The `ShareApplication` makes each node send a share every 5 seconds.
  - The latency is hardcoded as 100ms in the `p2p.SetChannelAttribute` line.
  - Logs show when shares are sent and received, like "Node 0 sent Share-0-1" and "Node 1 received Share-0-1".

### Running the Simulation
- From the `build` folder, I ran:
  ```bash
  ./ns3 run scratch/p2poolv2-competency-test
  ```
- It ran for 30 seconds and showed logs like:
  - "Node 0 sent Share-0-1 to Node 1 at 1s"
  - "Node 1 received Share-0-1 from 10.1.0.1 at 1.1s"

### Result
- The network had 10 nodes in a mesh.
- Shares were sent every 5 seconds with a 100ms delay.
- The trace file `p2poolv2-competency-test.tr` recorded all events.

## Conclusion
I successfully completed the competency test by setting up a 10-node mesh network and simulating shares with a hardcoded 100ms latency. This proves I can use NS3 and CMake for the P2Poolv2 project!
