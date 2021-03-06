# BGP convergence test plan using single DUT

- [BGP convergence test plan using single DUT](#bgp-convergence-test-plan-using-single-dut)
  - [Introduction](#Introduction)
  - [Overview](#Overview)
    - [Scope](#Scope)
    - [Testbed](#Testbed)
  - [Topology](#Topology)
    - [SONiC switch as ToR](#SONiC-switch-as-ToR)
    - [SONiC switch as Leaf](#SONiC-switch-as-Leaf)
  - [Setup configuration](#Setup-configuration)
  - [Test methodology](#Test-methodology)
  - [Test cases](#Test-cases)
    - [Test case # 1 – Convergence performance when remote link fails (route withdraw)](#test-case--1--convergence-performance-when-remote-link-fails-route-withdraw)
      - [Test objective](#Test-objective)
      - [Test steps](#Test-steps)
      - [Test results](#Test-results)
    - [Test case # 2 – RIB-IN Convergence](#Test-case--2--RIB-IN-Convergence)
      - [Test objective](#Test-objective-1)
      - [Test steps](#Test-steps-1)
      - [Test results](#Test-results-1)
    - [Call for action](#Call-for-action)
## Introduction
The corresponding PR assosciated with this testplan is [PR 2926](https://github.com/Azure/sonic-mgmt/pull/2926)

## Overview
The purpose of these tests is to test the overall convergence of a data center network by simulating multiple newtork devices such as ToR/Leafs and using SONiC switch DUT as one of the ToR/Leaf, closely resembling production environment.

### Scope
These tests are targeted on fully functioning SONiC system. The purpose of these tests are to measure convergence when some unexpected failures such as remote link failure, local link failure, node failure or link faults etc occur and some expected failures such as maintenance or upgrade of devices occur in the SONiC system.

### Keysight Testbed
The tests will run on following testbeds:
* t0

![Single DUT Topology ](Single_DUT_Topology.png)

## Topology
### SONiC switch as ToR

![SONiC DUT as ToR ](Switch_as_ToR.png)

### SONiC switch as Leaf

![SONiC DUT as ToR ](Switch_acting_as_Leaf.png)

## Setup configuration
IPv4 EBGP neighborship will be configured between SONiC DUT and directly connected test ports. Test ports inturn will simulate the ToR's and Leafs by advertising IPv4/IPv6, dual-stack routes.

## Test Methodology
Following test methodologies will be used for measuring convergence. 
* Traffic generator will be used to configure ebgp peering between chassis ports and SONiC DUT by advertising IPv4/IPv6, dual-stack routes. 
* Receiving ports will be advertising the same VIP(virtual IP) addresses. 
* Data traffic will be sent from  server to these VIP addresses. 
* Depending on the test case, the faults will be generated. Local link failures can be simulated on the port by "simulating link down" event. 
* Remote link failures can be simulated by withdrawing the routes.
* Control to data plane convergence will be measured by noting down the precise time of the control plane event and the data plane event. Convergence will be measured by taking the difference between contol and data plane events. Traffic generator will create those events and provide us with the control to data plane convergence value under statistics.
* RIB-IN Convergence is the time it takes to install the routes in its RIB and then in its FIB to forward the traffic without any loss. In order to measure RIB-IN convergence, initially IPv4/IPv6 routes will not be advertised. Once traffic is sent, IPv4/IPv6 routes will be advertised and the timestamp will be noted. Once the traffic received rate goes above the configured threshold value, it will note down the data plane above threshold timestamp. The difference between these two event timestamps will provide us with the RIB-IN convergence value.
* Route capacity can be measured by advertising routes in a linear search fashion. By doing this we can figure out the maximum routes a switch can learn and install in its RIB and then in its FIB to forward traffic without any loss.

## Test cases
### Test case # 1 – Convergence performance when remote link fails (route withdraw)
#### Test objective
Measure the convergence time when remote link failure event happens with in the network.

<p float="left">
  <img src="/Single_link_failure.png" width="500"  hspace="50"/>
  <img src="/Failover_convergence.png" width="380" /> 
</p>


#### Test steps
* Configure IPv4 EBGP sessions between Keysight ports and the SONiC switch.
* Advertise IPv4 routes along with AS number via configured IPv4 BGP sessions.
* Configure and advertise same IPv4 routes from both the test ports.
* Configure another IPv4 session to send the traffic. This is the server port from which traffic will be sent to the VIP addresses.
* Start all protocols and verify that IPv4 BGP neighborship is established.
* Create a data traffic between the server port and receiver ports where the same VIP addresses are configured and enable tracking by "Destination Endpoint" and by "Destination session description".
* Set the desired threshold value for receiving traffic. By default it will be set to 95% of expected receiving rate.
* Apply and start the data traffic.
* Verify that traffic is equally distributed between the receiving ports without any loss.
* Simulate remote link failure by withdrawing the routes from one receiving port. 
* Verify that the traffic is re-balanced and use the other available path to route the traffic.
* Drill down by "Destination Endpoint" under traffic statistics to get the control plane to data plane convergence value.
* In general the convergence value will fall in certain range. In order to achieve proper results, run the test multiple times and average out the test results. 
* Set it back to default configuration.
#### Test results
![Single remote link failure](Single_remote_link_failure.png)

For above test case, below are the test results when multiple remote link fails.

![Multiple link failure](Multi_link_failure.png)

![Multiple remote link failure](Multiple_remote_link_failure.png)

### Test case # 2 – RIB-IN Convergence 
#### Test objective
Measure the convergence time to install the routes in its RIB and then in its FIB to forward the packets after the routes are advertised.

<p float="left">
  <img src="/Switch_as_ToR.png" width="500" hspace="50"/>
  <img src="/RIB-IN_Convergence_graph.png" width="380" /> 
</p>

#### Test steps
* Configure IPv4 EBGP sessions between Keysight ports and the SONiC switch using IxNetwork web UI.
* Configure IPv4 routes via configured IPv4 BGP sessions. Initially disable the routes so that they don't get advertised after starting the protocols.
* Configure the same IPv4 routes from both the Keysight receiving ports.
* Configure another IPv4 session to send the traffic. This is the server port from which traffic will be sent to the VIP addresses.
* Start all protocols and verify that IPv4 BGP neighborship is established.
* Create a data traffic between the server port and receiver ports where the same VIP addresses are configured and enable tracking by "Destination Endpoint" and by "Destination session description".
* Set the desired threshold value for receiving traffic. By default it will be set to 95% of expected receiving rate.
* Apply and start the data traffic.
* Verify that no traffic is being forwarded. 
* Enable/advertise the routes which are already configured. 
* Control plane event timestamp will be noted down and once the receiving traffic rate goes above the configured threshold value, it will note down the data plane threshold timestamp.
* The difference between these two event timestamp will provide us with the RIB-IN convergence time.
* In general the convergence value will fall in certain range. In order to achieve proper results, run the test multiple times and average out the test results. 
* Set it back to default configuration.
#### Test results
![RIB-IN Convergence](RIB-IN_Convergence.png)

In order to measure RIB-IN capacity of the switch, we can follow the same test methodology as RIB-IN convergence test. Below are the results for RIB-IN capacity test.

![RIB-IN Capacity Test](RIB-IN_Capacity_Test.png)
### Call for action
* Solicitate experience in multi-DUT system test scenarios.
