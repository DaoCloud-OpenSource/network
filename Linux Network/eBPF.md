Linux kernel bypass models - improve networking performance by going around the Linux networking stack (needs modiﬁed device drivers), e.g.
● Intel Data Plane Development Kit (DPDK) -
● VPP (FD.io) - open source version of the Cisco’s vector packet processing But also, e.g OpenDataPlane, OpenFastPath, netmap, Snabb, pf_ring, etc.

Linux kernel fast path models, which try to process as much data early on the data path as possible, in Linux driver code or on NIC itself:
● eBPF - extended Berkeley Packet Filter (BPF)
● XDP - eXpress Data Path -
    ○ uses eBPF programs and performs processing RX packet-pages directly before the driver. It can run as native or oﬄoaded (BPF in NIC, or via DPDK).