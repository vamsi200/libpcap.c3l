# libpcap.c3i

C3 bindings for libpcap (pcap.h, bpf.h)
>Note:
We link against pcap, so libpcap must be installed on system:
```json
"linked-libraries": [
  "pcap"
]
```

Source: https://github.com/the-tcpdump-group/libpcap

## Installation
Fetch using c3l:

```bash
c3l fetch https://github.com/vamsi200/libpcap.c3l
```

get c3l from:

- https://github.com/konimarti/c3l

## Example Usage

### Find available interfaces
```c3
import libpcap;

fn void main() {
    Pcap_if_t* alldevs;
    CChar[1024] errbuf;

    CInt ret = pcap::pcap_findalldevs(
        &alldevs,
        &errbuf[0]
    );

    if (ret == -1) {
        io::printfn("failed to find interfaces: %s", &errbuf[0]);
        return;
    }

    Pcap_if_t* dev = alldevs;

    while (dev != null) {
        io::printfn("interface: %s", (ZString)dev.name);
        dev = dev.next;
    }

    pcap::pcap_freealldevs(alldevs);
}
```

### Capture packets with a BPF filter

```c3
import libpcap;

fn void main() {
    CChar[1024] errbuf;

    Pcap_t* pcap = pcap::pcap_open_live(
        "enp4s0",
        65535,
        1,
        1000,
        &errbuf[0]
    );

    if (pcap == null) {
        io::printfn("failed to open interface: %s", &errbuf[0]);
        return;
    }

    Bpf_program filter;

    CInt ret = pcap::pcap_compile(
        pcap,
        &filter,
        "tcp port 443",
        1,
        0
    );

    if (ret == -1) {
        io::printfn("failed to compile filter");
        return;
    }

    ret = pcap::pcap_setfilter(pcap, &filter);

    if (ret == -1) {
        io::printfn("failed to set filter: %s", pcap::pcap_geterr(pcap));
        return;
    }

    while (true) {
        char* data;
        Pcap_pkthdr* header;

        CInt out = pcap::pcap_next_ex(pcap, &header, &data);

        if (out == 0) continue;

        if (out == -1) {
            io::printfn("capture error: %s", pcap::pcap_geterr(pcap));
            break;
        }

        io::printfn(
            "captured packet: %d bytes",
            header.len
        );
    }

    pcap::pcap_close(pcap);
}
```

### Open offline capture
```c3
import libpcap;

fn void main() {
    CChar[1024] errbuf;

    Pcap_t* pcap = pcap::pcap_open_offline(
        "capture.pcap",
        &errbuf[0]
    );

    if (pcap == null) {
        io::printfn("failed to open capture: %s", &errbuf[0]);
        return;
    }

    io::printfn("capture opened successfully");

    pcap::pcap_close(pcap);
}
```
