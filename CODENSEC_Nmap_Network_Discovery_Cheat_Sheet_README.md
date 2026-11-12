# CODENSEC --- Complete Nmap & Network Discovery Cheat Sheet

> **Learn. Practice. Exploit. Elevate.**\
> A practical Nmap reference for beginner-to-advanced penetration
> testing students.

**Written & Prepared by:** Saadullah Abdul Wahid\
**Community:** CODENSEC\
**YouTube:** [CODENSEC ---
YouTube](https://www.youtube.com/@CodenSec_01)

------------------------------------------------------------------------

## About This Guide

Nmap (Network Mapper) is an open-source tool for network discovery, port
scanning, service/version identification, operating-system
fingerprinting, and security auditing. This guide follows a practical
assessment workflow: identify live systems, map exposed ports, identify
services, enumerate what is exposed, validate observations, and preserve
results.

It is designed as a live-session companion and a reusable post-session
reference. Commands are examples, not guarantees: output depends on the
target, network path, firewall rules, privileges, Nmap version, and
service configuration.

> \[!IMPORTANT\] **Authorization required.** Scan only systems and
> networks you own or have explicit written permission to assess. The
> lab examples below are intended for an isolated virtual network. Do
> not scan public IP ranges, school/company networks, or third-party
> systems without authorization. NSE scripts are executable code; review
> script purpose and impact before running them.

## Table of Contents

-   [Learning Objectives](#learning-objectives)
-   [Lab Requirements and Network
    Setup](#lab-requirements-and-network-setup)
-   [Install and Verify Nmap on Kali
    Linux](#install-and-verify-nmap-on-kali-linux)
-   [A Reliable Nmap Workflow](#a-reliable-nmap-workflow)
-   [1. Network Discovery](#1-network-discovery)
-   [2. Host Discovery](#2-host-discovery)
-   [3. TCP Port Scanning](#3-tcp-port-scanning)
-   [4. UDP Port Scanning](#4-udp-port-scanning)
-   [5. Full Port Scanning](#5-full-port-scanning)
-   [6. Service and Version Detection](#6-service-and-version-detection)
-   [7. OS Detection](#7-os-detection)
-   [8. Advanced Nmap Scanning](#8-advanced-nmap-scanning)
-   [9. NSE Script Scanning](#9-nse-script-scanning)
-   [10. Service Enumeration](#10-service-enumeration)
-   [11. Web Server Discovery](#11-web-server-discovery)
-   [Understanding Port States](#understanding-port-states)
-   [Timing Templates T0--T5](#timing-templates-t0t5)
-   [Output Formats and Evidence](#output-formats-and-evidence)
-   [Important Nmap Flags Reference](#important-nmap-flags-reference)
-   [Complete Practical Scanning
    Workflow](#complete-practical-scanning-workflow)
-   [Student Practical Exercise](#student-practical-exercise)
-   [Troubleshooting](#troubleshooting)
-   [Quick-Reference Commands](#quick-reference-commands)
-   [Further Reading](#further-reading)

------------------------------------------------------------------------

## Learning Objectives

By the end of this guide, students should be able to:

1.  Explain the difference between host discovery and port scanning.
2.  Identify live hosts in an authorized lab subnet.
3.  Perform TCP SYN, TCP connect, and UDP scans.
4.  Select port ranges, including all TCP ports.
5.  Identify service names, versions, and likely operating systems.
6.  Interpret `open`, `closed`, `filtered`, and `open|filtered` results.
7.  Use timing templates responsibly and understand their trade-offs.
8.  Select and run appropriate NSE scripts after reviewing their impact.
9.  Enumerate common services such as SSH, FTP, DNS, SMB, HTTP, and
    HTTPS.
10. Save scan evidence in formats suitable for humans and tools.

## Lab Requirements and Network Setup

### Required tools and systems

  Component          Purpose
  ------------------ -------------------------------------------------
  Kali Linux         Assessment workstation with Nmap
  Metasploitable 2   Intentionally vulnerable Linux target
  DVWA               Deliberately vulnerable web application
  Nmap               Discovery, scanning, detection, and enumeration

### Example isolated lab addressing

Use a **host-only** or otherwise isolated virtual network. The following
addresses are examples; confirm the actual addresses assigned by your
hypervisor.

  System                           Example address
  ---------------------------- -------------------
  Kali Linux                       `192.168.56.10`
  Metasploitable 2                `192.168.56.101`
  Separate DVWA VM (if used)      `192.168.56.102`
  Example lab subnet             `192.168.56.0/24`

Metasploitable 2 may host DVWA itself, depending on the image/build. If
DVWA is installed on a separate VM, scan that VM's assigned IP. Do not
assume the example addresses match your setup.

### Confirm your lab network

On Kali:

``` bash
ip -br addr
ip route
```

Find the interface and subnet connected to the isolated lab. Then set
shell variables to your real lab values:

``` bash
export LAB_NET="192.168.56.0/24"
export MS2="192.168.56.101"
export DVWA="192.168.56.102"
```

Replace these values if your lab uses different addresses. Verify
reachability and target identity before scanning.

> \[!WARNING\] Never bridge intentionally vulnerable machines to a
> public or production network. Keep Metasploitable 2 and DVWA isolated,
> use snapshots, and power them off when the exercise is complete.

------------------------------------------------------------------------

## Install and Verify Nmap on Kali Linux

Nmap is commonly available in Kali's package repositories.

``` bash
sudo apt update
sudo apt install -y nmap
nmap --version
nmap --help
```

Useful checks:

``` bash
command -v nmap
nmap -V
```

-   `apt update` refreshes package metadata.
-   `apt install -y nmap` installs Nmap and accepts the package
    manager's confirmation prompt.
-   `--version` / `-V` displays the installed version and build details.
-   `--help` displays a concise option summary.

Use `sudo` when a scan technique requires raw-packet privileges, such as
a conventional SYN scan or OS detection. An unprivileged user can still
run TCP connect scans and many other Nmap functions.

------------------------------------------------------------------------

## A Reliable Nmap Workflow

A disciplined assessment avoids jumping straight to aggressive scans.

1.  **Confirm authorization and scope.** Record approved IPs, subnets,
    exclusions, and test window.
2.  **Identify your interface and subnet.** Use `ip -br addr` and
    `ip route`.
3.  **Discover hosts.** Use `-sn` against the approved lab subnet.
4.  **Scan selected hosts.** Start with common TCP ports, then expand if
    scope allows.
5.  **Enumerate services.** Use `-sV` on discovered open ports.
6.  **Check UDP deliberately.** UDP scans can take longer and often
    return ambiguous states.
7.  **Fingerprint cautiously.** Add `-O` where useful; validate the
    result independently.
8.  **Use NSE selectively.** Inspect scripts and choose safe, relevant
    checks.
9.  **Investigate services.** Use protocol-specific scripts and manual
    validation.
10. **Save evidence.** Use `-oA` and retain the exact command,
    timestamp, and target scope.
11. **Validate and report.** Distinguish observed facts from inferred
    versions or OS guesses.

A scan result is a point-in-time observation, not proof that a service
is always available or vulnerable.

------------------------------------------------------------------------

## 1. Network Discovery

**Purpose:** Identify systems and network exposure within an authorized
IP range. Discovery is usually the first step before detailed port
scanning.

### Essential commands

List the targets Nmap would consider, without sending scan probes:

``` bash
nmap -sL 192.168.56.0/24
```

Discover active hosts without performing a port scan:

``` bash
sudo nmap -sn 192.168.56.0/24
```

Discover hosts on a local Ethernet segment (Nmap generally uses
ARP/neighbor discovery on local networks):

``` bash
sudo nmap -sn 192.168.56.0/24
```

### Important flags

  -----------------------------------------------------------------------
  Flag                                Meaning
  ----------------------------------- -----------------------------------
  `-sL`                               List scan; lists targets and may
                                      perform name resolution, but does
                                      not probe target ports

  `-sn`                               Host discovery only; disables the
                                      port scan

  `-n`                                Skip DNS resolution

  `-R`                                Always resolve DNS names

  `-PR`                               ARP discovery on local Ethernet

  `-oN file.txt`                      Save normal output
  -----------------------------------------------------------------------

### Example interpretation

``` text
Nmap scan report for 192.168.56.10
Host is up (0.00040s latency).
Nmap scan report for 192.168.56.101
Host is up (0.00080s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned
```

This illustrative output means Nmap received responses indicating two
hosts were up. It does **not** mean only two devices exist; firewalls,
disabled responses, routing, or discovery method can hide hosts.

### Common mistakes and tester tips

-   **Mistake:** Treating `-sL` as a live-host scan. It lists targets;
    it does not establish that they are online.
-   **Mistake:** Assuming no response means a host is powered off. ICMP
    and discovery probes may be filtered.
-   **Tip:** On a local LAN, ARP discovery is often more reliable than
    ICMP echo alone.
-   **Tip:** Record the exact subnet and exclusions before scanning; an
    incorrect CIDR can expand scope significantly.

------------------------------------------------------------------------

## 2. Host Discovery

**Purpose:** Determine which target addresses appear reachable before
spending time on port scans.

### Essential commands

Default discovery only:

``` bash
sudo nmap -sn 192.168.56.0/24
```

ICMP echo discovery:

``` bash
sudo nmap -sn -PE 192.168.56.0/24
```

TCP SYN discovery probes to selected ports:

``` bash
sudo nmap -sn -PS22,80,443 192.168.56.0/24
```

TCP ACK discovery probes:

``` bash
sudo nmap -sn -PA80,443 192.168.56.0/24
```

UDP discovery probes:

``` bash
sudo nmap -sn -PU53,161 192.168.56.0/24
```

If discovery probes are blocked and the target is explicitly in scope,
skip host discovery and attempt the requested scan:

``` bash
sudo nmap -Pn -p 22,80,443 192.168.56.101
```

### Important flags

-   `-PE`: ICMP echo request.
-   `-PS<ports>`: TCP SYN discovery probes.
-   `-PA<ports>`: TCP ACK discovery probes.
-   `-PU<ports>`: UDP discovery probes.
-   `-Pn`: Treat targets as online and skip the normal host discovery
    stage.
-   `-sn`: Do host discovery only; no port scan.

Nmap's default discovery probes vary by privilege and network context.
On a local Ethernet segment it commonly uses ARP/neighbor discovery.
Discovery results are affected by routing and firewall policy.

### Common mistakes and tester tips

-   **Mistake:** Using `-Pn` on a large range without considering the
    impact. Nmap will attempt the requested scan against every specified
    address.
-   **Mistake:** Believing `-Pn` makes a host reachable. It only tells
    Nmap not to exclude it based on discovery.
-   **Tip:** If a known lab host is missing, compare discovery methods
    and check its VM network mode and firewall.
-   **Tip:** Use `--reason` to display why Nmap assigned a host or port
    state.

------------------------------------------------------------------------

## 3. TCP Port Scanning

**Purpose:** Determine which TCP ports are open, closed, or filtered.

### TCP SYN scan

``` bash
sudo nmap -sS -p 1-1000 192.168.56.101
```

`-sS` sends TCP SYN probes and interprets responses without completing a
normal TCP connection in the usual case. It is often called a half-open
scan and generally requires raw-packet privileges.

### TCP connect scan

``` bash
nmap -sT -p 1-1000 192.168.56.101
```

`-sT` uses the operating system's `connect()` call to establish a full
TCP connection. It is useful when raw-packet privileges are unavailable,
but it completes connections and may be more visible to the application.

### SYN vs. connect scan

  -----------------------------------------------------------------------
  Feature                 SYN (`-sS`)             Connect (`-sT`)
  ----------------------- ----------------------- -----------------------
  Method                  Sends SYN probes and    Uses OS TCP connect
                          analyzes replies        call

  TCP handshake           Normally does not       Completes it for open
                          complete it             ports

  Privileges              Usually requires        Works without
                          elevated/raw-packet     raw-packet privileges
                          privileges              

  Typical use             Common privileged TCP   Unprivileged fallback
                          scan                    or when connect
                                                  behavior is needed

  Visibility              Not inherently          May be logged by
                          invisible               services and
                                                  applications
  -----------------------------------------------------------------------

Neither method is guaranteed to be stealthy. Firewalls, IDS/IPS,
endpoint logging, and network telemetry may detect either.

### Useful examples

Scan common ports:

``` bash
sudo nmap -sS --top-ports 100 192.168.56.101
```

Scan selected ports:

``` bash
sudo nmap -sS -p 21,22,23,25,80,139,445 192.168.56.101
```

Show why Nmap assigned states:

``` bash
sudo nmap -sS --reason -p 22,80,443 192.168.56.101
```

### Common mistakes and tester tips

-   **Mistake:** Running `-sS` without privileges and assuming the
    requested scan type was used. Check Nmap's output and warnings.
-   **Mistake:** Treating a port's service-name label as verified
    software. Use `-sV` for service probing.
-   **Tip:** Start with a small, justified port set; expand after
    reviewing results.
-   **Tip:** A filtered result is useful evidence about network
    controls, not a failed scan.

------------------------------------------------------------------------

## 4. UDP Port Scanning

**Purpose:** Identify UDP services such as DNS, SNMP, DHCP, NTP, and
TFTP.

### Essential commands

Scan common UDP ports:

``` bash
sudo nmap -sU --top-ports 20 192.168.56.101
```

Scan selected UDP ports:

``` bash
sudo nmap -sU -p 53,67,68,69,123,161 192.168.56.101
```

Combine TCP and UDP checks:

``` bash
sudo nmap -sS -sU -p T:22,80,443,U:53,123,161 192.168.56.101
```

Add version detection to selected UDP ports:

``` bash
sudo nmap -sU -sV -p 53,123,161 192.168.56.101
```

### Why UDP scans are different

UDP has no TCP-style handshake. An open UDP service may respond with
application data, while a closed UDP port commonly returns an ICMP
port-unreachable message. Many open UDP services ignore empty or
unexpected probes. Firewalls may also drop probes and replies.

As a result, UDP scans can be slower and more ambiguous than TCP scans.
Nmap may retry probes and apply rate limits based on network feedback.

### Common mistakes and tester tips

-   **Mistake:** Interpreting `open|filtered` as confirmed open. It is
    inconclusive.
-   **Mistake:** Scanning every UDP port first. Begin with relevant
    common ports, then expand if justified.
-   **Tip:** Use `-sV` to send service-aware probes where appropriate.
-   **Tip:** Correlate Nmap results with application-specific validation
    and network captures when a UDP result matters.

------------------------------------------------------------------------

## 5. Full Port Scanning

**Purpose:** Check the full TCP port range rather than only Nmap's
default set of 1,000 commonly used ports.

### Essential commands

All TCP ports (1--65535):

``` bash
sudo nmap -sS -p- 192.168.56.101
```

All TCP ports with service/version detection:

``` bash
sudo nmap -sS -p- -sV 192.168.56.101
```

All TCP ports with a faster timing template (use only when appropriate
for the lab):

``` bash
sudo nmap -sS -p- -T4 192.168.56.101
```

Scan a defined range:

``` bash
sudo nmap -sS -p 1-5000 192.168.56.101
```

Scan the 1,000 most common ports:

``` bash
sudo nmap --top-ports 1000 192.168.56.101
```

### Port specification

  Syntax                   Meaning
  ------------------------ -----------------------------------------------
  `-p 22`                  One TCP port (for a TCP scan)
  `-p 1-1024`              Inclusive port range
  `-p 22,80,443`           Comma-separated ports
  `-p-`                    TCP ports 1--65535
  `-F`                     Fast mode; scans fewer ports than the default
  `--top-ports 100`        Most common 100 ports
  `-p T:80,443,U:53,161`   Explicit TCP and UDP port lists

`-p-` means all TCP ports when used with a TCP scan; it does not
automatically scan all UDP ports. To scan all UDP ports, explicitly use
`-sU -p 1-65535`, understanding that this can be very time-consuming.

### Practical approach

A common two-stage method is:

``` bash
sudo nmap -sS -p- --open -oA results/ms2-all-tcp 192.168.56.101
sudo nmap -sV -sC -p 21,22,80,139,445 192.168.56.101 -oA results/ms2-detail
```

Replace the second command's ports with the ports actually found in the
first scan. Create the output directory first:

``` bash
mkdir -p results
```

### Common mistakes and tester tips

-   **Mistake:** Assuming a default scan covers every port. By default,
    Nmap scans the 1,000 most common ports for the selected protocol.
-   **Mistake:** Applying `-sV` to all 65,535 ports without considering
    time and traffic.
-   **Tip:** Find ports first, then perform focused version detection.
-   **Tip:** `--open` filters displayed results to ports Nmap considers
    open; retain full output if you need closed/filtered evidence too.

------------------------------------------------------------------------

## 6. Service and Version Detection

**Purpose:** Probe discovered open ports to identify the service
protocol, product, and possible version.

### Essential commands

``` bash
sudo nmap -sV 192.168.56.101
```

Version detection on selected ports:

``` bash
sudo nmap -sV -p 21,22,80,139,445 192.168.56.101
```

Use lighter version probing:

``` bash
sudo nmap -sV --version-light 192.168.56.101
```

Use maximum version-probe intensity:

``` bash
sudo nmap -sV --version-all 192.168.56.101
```

### Important flags

-   `-sV`: Enables service/version detection.
-   `--version-light`: Uses a lower probe intensity (level 2).
-   `--version-all`: Tries all version probes (level 9); may take
    longer.
-   `--version-intensity 0-9`: Sets the probe intensity.
-   `-p`: Restricts the ports to probe.

### Example output (illustrative)

``` text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1
80/tcp open  http    Apache httpd 2.2.8
```

The result is a fingerprint-based identification, not a guarantee that
every version string is complete or accurate. Proxies, banners,
backported security fixes, and customized services can make version
interpretation difficult.

### Common mistakes and tester tips

-   **Mistake:** Declaring a vulnerability solely because a version
    string looks old. Confirm the exact build, vendor patches,
    configuration, and applicability.
-   **Mistake:** Assuming the conventional port determines the service.
    Services can run on non-standard ports.
-   **Tip:** Keep service detection focused on ports already found.
-   **Tip:** Validate important findings with authenticated inventory,
    vendor advisories, or service-specific checks where authorized.

------------------------------------------------------------------------

## 7. OS Detection

**Purpose:** Estimate the target operating system using TCP/IP stack
characteristics and responses.

### Essential commands

``` bash
sudo nmap -O 192.168.56.101
```

Combine OS detection with service detection:

``` bash
sudo nmap -O -sV 192.168.56.101
```

Aggressive detection bundle:

``` bash
sudo nmap -A 192.168.56.101
```

`-A` enables OS detection, version detection, default NSE scripts, and
traceroute. It is not a single OS-detection flag and may generate more
traffic than a focused scan.

### Important flags

-   `-O`: Enable OS detection.
-   `--osscan-guess`: Make a more aggressive guess when Nmap cannot
    identify an exact match.
-   `--osscan-limit`: Limit OS detection to promising targets.
-   `-A`: Enable several advanced features together; use only when
    appropriate.

### Limitations

OS detection is based on network-stack fingerprinting. It can be
inconclusive when:

-   A firewall or middlebox modifies or blocks probes.
-   The target is behind NAT, a proxy, or a load balancer.
-   The host has few suitable open/closed TCP ports.
-   The operating system uses a customized or shared network stack.
-   The target's responses do not match Nmap's fingerprint database
    closely.

Treat OS results as an estimate. Confirm with other authorized evidence.

### Common mistakes and tester tips

-   **Mistake:** Reporting an OS guess as certainty.
-   **Tip:** Use `-O` after identifying a responsive host and useful
    port behavior.
-   **Tip:** Compare fingerprint output with asset inventory or
    authenticated checks before making a high-impact decision.

------------------------------------------------------------------------

## 8. Advanced Nmap Scanning

Use advanced scan options to answer a specific assessment question, not
simply to make a command look sophisticated.

### Timing templates T0--T5

``` bash
sudo nmap -sS -T3 192.168.56.101
```

  -----------------------------------------------------------------------
  Template                General behavior        Practical guidance
  ----------------------- ----------------------- -----------------------
  `-T0`                   Paranoid; extremely     Specialized, controlled
                          slow and highly         testing
                          serialized              

  `-T1`                   Sneaky; very slow       Specialized, controlled
                                                  testing

  `-T2`                   Polite; slower and      Slower/fragile
                          reduces network load    environments

  `-T3`                   Normal/default timing   Sensible baseline

  `-T4`                   Aggressive; assumes a   Common in a stable lab
                          reasonably              
                          fast/reliable network   

  `-T5`                   Insane; very aggressive Can miss results or
                          timing                  overload fragile
                                                  targets
  -----------------------------------------------------------------------

Timing templates change timing behavior; they do not make a scan
undetectable. Faster timing can reduce accuracy on congested, lossy, or
rate-limited networks. Slower timing does not guarantee evasion.

### Traceroute

``` bash
sudo nmap --traceroute -p 80 192.168.56.101
```

Shows the network path as observed by Nmap, subject to routing and
filtering.

### Disable DNS resolution

``` bash
sudo nmap -n -sS 192.168.56.101
```

Useful when reverse DNS is slow or irrelevant.

### Reason reporting and verbosity

``` bash
sudo nmap --reason -v -sS 192.168.56.101
```

-   `--reason`: Shows the response reason behind host/port state
    decisions.
-   `-v`: Increases progress and result detail; repeat as `-vv` for more
    verbosity.

### Exclude a host or port

``` bash
sudo nmap 192.168.56.0/24 --exclude 192.168.56.1
sudo nmap 192.168.56.101 --exclude-ports 22,23
```

Use exclusions to honor written scope and avoid infrastructure that is
not approved for testing.

### Firewall and host-discovery limitations

A firewall can drop discovery probes, reset TCP connections, rate-limit
responses, or allow only selected protocols. Consequently:

-   A host may be online but appear down.
-   A port may appear `filtered` because Nmap cannot distinguish open
    from closed.
-   A middlebox may respond on behalf of the destination.
-   Different scan methods may produce different observations.

If a known lab system is not discovered, verify its IP, route, VM
network mode, and firewall before interpreting the scan. Use `-Pn` only
for confirmed in-scope targets.

------------------------------------------------------------------------

## 9. NSE Script Scanning

**Purpose:** Nmap Scripting Engine (NSE) extends Nmap with Lua scripts
for discovery, service checks, configuration review, and other tasks.

### Inspect scripts before running them

``` bash
ls /usr/share/nmap/scripts/
nmap --script-help "http-title"
nmap --script-help "smb-os-discovery"
```

### Safe starting examples

Run the default script set against an authorized lab target:

``` bash
sudo nmap -sC -sV 192.168.56.101
```

Run one specific script:

``` bash
sudo nmap --script http-title -p 80,8080 192.168.56.101
```

Run selected discovery scripts:

``` bash
sudo nmap --script "banner,ssh-hostkey" -p 22 192.168.56.101
```

Run a relevant safe-category script set on a lab host:

``` bash
sudo nmap --script "safe" 192.168.56.101
```

The `safe` category is intended for scripts not expected to crash
services, but this is not a substitute for reviewing a script and
testing its impact. Scripts are not sandboxed.

### NSE categories

  ------------------------------------------------------------------------
  Category                General purpose          Caution
  ----------------------- ------------------------ -----------------------
  `auth`                  Authentication-related   May interact with login
                          checks                   mechanisms

  `broadcast`             Broadcast/network        Can affect a local
                          discovery                segment

  `brute`                 Credential guessing      Intrusive; explicit
                                                   approval required

  `default`               Common default scripts   Review scripts and
                          (`-sC`)                  target impact

  `discovery`             Gather additional        Check what data is
                          information              queried

  `dos`                   Denial-of-service        Avoid outside a
                          testing                  specifically approved
                                                   test

  `exploit`               Exploit-related checks   High risk; do not run
                                                   casually

  `external`              Uses external services   May disclose target
                                                   data externally

  `fuzzer`                Sends varied/unusual     Can disrupt services
                          inputs                   

  `intrusive`             May be disruptive or     Requires explicit
                          conspicuous              approval

  `malware`               Checks for               Interpret results
                          malware-related          carefully
                          indicators               

  `safe`                  Intended to be           Still review before use
                          non-disruptive           

  `version`               Enhances service/version Often runs as part of
                          detection                version detection

  `vuln`                  Vulnerability checks     Findings require
                                                   validation
  ------------------------------------------------------------------------

### Find and update the script database

``` bash
sudo nmap --script-updatedb
```

This updates Nmap's local script database after script changes. It does
not download arbitrary scripts from the internet.

### Practical caution

Avoid broad commands such as `--script vuln`, `--script exploit`,
`--script brute`, or `--script dos` against production systems. A script
category is not a guarantee of safety or a vulnerability verdict. Review
individual script documentation, arguments, and source before use.

------------------------------------------------------------------------

## 10. Service Enumeration

**Purpose:** Collect service-specific information after discovering an
open port. Enumeration should be targeted to the service actually
observed.

### SSH (usually TCP/22)

``` bash
sudo nmap -sV -p 22 --script ssh-hostkey,ssh2-enum-algos 192.168.56.101
```

Can report SSH host keys and supported algorithm information. It does
not authenticate as a user.

### FTP (usually TCP/21)

``` bash
sudo nmap -sV -p 21 --script ftp-anon,ftp-syst 192.168.56.101
```

-   `ftp-anon` checks whether anonymous FTP access is allowed.
-   `ftp-syst` requests server system information where supported.

Anonymous access may expose files; access only your lab or an explicitly
authorized service.

### SMB (usually TCP/445 and sometimes TCP/139)

``` bash
sudo nmap -sV -p 139,445 --script smb-os-discovery,smb-protocols 192.168.56.101
```

Optional share listing (may disclose share names and access
information):

``` bash
sudo nmap -p 445 --script smb-enum-shares 192.168.56.101
```

SMB script output can be affected by protocol versions, signing, guest
access, and authentication requirements. Do not assume a listed share is
readable.

### DNS (usually TCP/UDP 53)

Basic service check:

``` bash
sudo nmap -sU -sV -p 53 192.168.56.101
```

Check for a DNS service over TCP:

``` bash
sudo nmap -sT -sV -p 53 192.168.56.101
```

A zone-transfer test requires a known zone name and explicit
authorization:

``` bash
sudo nmap -p 53 --script dns-zone-transfer \
  --script-args dns-zone-transfer.domain=lab.example 192.168.56.101
```

Use a zone name that belongs to your lab. A failed transfer does not
prove that DNS is secure; it only means that this test did not obtain a
transfer.

### HTTP / HTTPS (usually TCP/80 and TCP/443)

``` bash
sudo nmap -sV -p 80,443 --script http-title,http-server-header 192.168.56.101
```

Enumerate common HTTP paths on your lab web server:

``` bash
sudo nmap -p 80 --script http-enum 192.168.56.101
```

`http-enum` can generate multiple requests and may be logged. Use it
only against approved lab services.

### Common mistakes and tester tips

-   **Mistake:** Running every service script against every port. Match
    scripts to the service and port.
-   **Mistake:** Treating a banner as definitive proof of the underlying
    software build.
-   **Tip:** Capture the exact port, protocol, script name, and output
    in your notes.
-   **Tip:** Validate meaningful findings manually, and distinguish "not
    detected" from "not present."

------------------------------------------------------------------------

## 11. Web Server Discovery

**Purpose:** Identify web services, basic server information, page
titles, and common exposed paths on authorized web targets.

### Discover common web ports

``` bash
sudo nmap -sS -p 80,443,8000,8080,8443 192.168.56.101
```

### Detect service and page metadata

``` bash
sudo nmap -sV -p 80,443,8080,8443 \
  --script http-title,http-server-header 192.168.56.101
```

### Check common paths

``` bash
sudo nmap -p 80 --script http-enum 192.168.56.101
```

### Inspect HTTPS

``` bash
sudo nmap -sV -p 443 --script ssl-cert,ssl-enum-ciphers 192.168.56.101
```

TLS scripts can reveal certificate and cipher configuration details.
Their output is not a complete web-application security assessment.

### DVWA lab example

If DVWA is hosted on the Metasploitable 2 machine, use that machine's
IP. If it is a separate VM, use the separate DVWA IP:

``` bash
sudo nmap -sV -p 80,443,8080 192.168.56.102 \
  --script http-title,http-server-header
```

Then open the DVWA URL in a browser using the correct protocol, IP, and
path for your installation. Nmap identifies network-facing services; it
does not replace application testing.

### Common mistakes and tester tips

-   **Mistake:** Assuming port 80 is the only web port. Check other
    approved ports such as 8080 and 8443.
-   **Mistake:** Assuming a title or server header identifies every
    framework or component.
-   **Tip:** Record redirects, virtual-host behavior, TLS certificates,
    and the exact URL used during validation.
-   **Tip:** Do not run content discovery against a public site unless
    it is explicitly in scope.

------------------------------------------------------------------------

## Understanding Port States

Nmap reports the state of a port **from the scanning host's network
perspective**. A state is not an intrinsic property of a service.

  -----------------------------------------------------------------------
  State                   Meaning                 Typical observation
  ----------------------- ----------------------- -----------------------
  `open`                  An application is       TCP SYN/ACK, or a UDP
                          accepting connections   application response
                          or packets on that port 

  `closed`                The port is reachable,  TCP RST or ICMP port
                          but no application is   unreachable for UDP
                          listening               

  `filtered`              A filter or network     No response or certain
                          obstacle prevents Nmap  ICMP errors
                          from determining open   
                          vs. closed              

  `unfiltered`            Port is reachable, but  Often associated with
                          Nmap cannot determine   ACK scans
                          whether it is open or   
                          closed                  

  `open|filtered`         Nmap cannot distinguish Common in UDP and some
                          an open port from a     stealth scan outcomes
                          filtered one            

  `closed|filtered`       Nmap cannot distinguish Can occur with IP
                          a closed port from a    protocol scans
                          filtered one            
  -----------------------------------------------------------------------

### Illustrative examples

``` text
22/tcp  open     ssh
23/tcp  closed   telnet
80/tcp  filtered http
53/udp  open|filtered domain
```

Interpretation:

-   `22/tcp open`: a TCP service responded in a way consistent with an
    open port.
-   `23/tcp closed`: the host responded, but no TCP service appears to
    be listening there.
-   `80/tcp filtered`: filtering prevents a reliable open/closed
    decision.
-   `53/udp open|filtered`: no response was sufficient to distinguish an
    open UDP service from packet filtering.

Use `--reason` to inspect the response that led to a state:

``` bash
sudo nmap --reason -sS -sV -p 22,23,80 192.168.56.101
```

------------------------------------------------------------------------

## Timing Templates T0--T5

Nmap timing templates tune probe timing and parallelism. They are not
stealth levels.

  -----------------------------------------------------------------------
  Option                  Name                    Use
  ----------------------- ----------------------- -----------------------
  `-T0`                   Paranoid                Very slow, specialized
                                                  testing

  `-T1`                   Sneaky                  Very slow, specialized
                                                  testing

  `-T2`                   Polite                  Lower network load

  `-T3`                   Normal                  Default

  `-T4`                   Aggressive              Stable, low-latency
                                                  networks

  `-T5`                   Insane                  Very aggressive;
                                                  increased risk of
                                                  inaccurate results
  -----------------------------------------------------------------------

Example:

``` bash
sudo nmap -sS -T4 -p 1-1000 192.168.56.101
```

**Tester rule:** Begin with the default timing unless there is a reason
to change it. On a fragile or rate-limited target, slower timing may be
more reliable. On a stable isolated lab, `-T4` is often practical. Do
not assume `-T0` or `-T1` avoids detection.

------------------------------------------------------------------------

## Output Formats and Evidence

Save results so they can be reviewed, compared, parsed, and attached to
an assessment report.

### Normal output

``` bash
sudo nmap -sV 192.168.56.101 -oN scan-normal.txt
```

Human-readable output.

### XML output

``` bash
sudo nmap -sV 192.168.56.101 -oX scan.xml
```

Structured output suitable for tools and automation. XML is generally
preferable to grepable output for programmatic processing.

### Grepable output

``` bash
sudo nmap -sV 192.168.56.101 -oG scan.gnmap
```

Legacy line-oriented format useful for simple text processing. It is
deprecated for many modern workflows; prefer XML for new integrations.

### All major output formats

``` bash
mkdir -p results
sudo nmap -sV 192.168.56.101 -oA results/ms2-service-scan
```

`-oA` writes three files using the same basename:

-   `results/ms2-service-scan.nmap` --- normal output
-   `results/ms2-service-scan.xml` --- XML output
-   `results/ms2-service-scan.gnmap` --- grepable output

### Useful evidence options

-   `-v` / `-vv`: Increase verbosity.
-   `--reason`: Include reasons for states.
-   `-d`: Debug output; use when troubleshooting.
-   `--packet-trace`: Show packets sent and received; can produce
    substantial output.
-   `-oN`, `-oX`, `-oG`, `-oA`: Save scan output in different formats.

For reproducibility, record the Nmap version, exact command, date/time,
source machine, target scope, and any exclusions. Protect scan files
because they may contain sensitive network details.

------------------------------------------------------------------------

## Important Nmap Flags Reference

  -----------------------------------------------------------------------
  Flag                                Description
  ----------------------------------- -----------------------------------
  `-sL`                               List targets without probing them

  `-sn`                               Host discovery only; no port scan

  `-Pn`                               Skip normal host discovery; treat
                                      targets as online

  `-PS`                               TCP SYN host-discovery probe

  `-PA`                               TCP ACK host-discovery probe

  `-PE`                               ICMP echo discovery probe

  `-PU`                               UDP host-discovery probe

  `-sS`                               TCP SYN scan

  `-sT`                               TCP connect scan

  `-sU`                               UDP scan

  `-sA`                               TCP ACK scan; useful for firewall
                                      rule mapping, not open-port
                                      identification

  `-sV`                               Service/version detection

  `-O`                                OS detection

  `-A`                                OS detection, version detection,
                                      default NSE scripts, and traceroute

  `-sC`                               Run default NSE scripts; equivalent
                                      to `--script=default`

  `--script`                          Select NSE scripts, categories, or
                                      expressions

  `--script-help`                     Show script help

  `--script-args`                     Supply script arguments

  `-p`                                Select ports or ranges

  `-p-`                               All TCP ports 1--65535 for a TCP
                                      scan

  `-F`                                Fast mode; fewer ports than default

  `--top-ports N`                     Scan N most common ports

  `--open`                            Show only open (or potentially
                                      open) ports

  `-n`                                Disable DNS resolution

  `-R`                                Force DNS resolution

  `-T0`--`-T5`                        Timing templates

  `--reason`                          Show reasons for states

  `--traceroute`                      Trace route to targets

  `--exclude`                         Exclude hosts from scan

  `--exclude-ports`                   Exclude ports from scan

  `-oN`                               Normal output file

  `-oX`                               XML output file

  `-oG`                               Grepable output file

  `-oA`                               Output in normal, XML, and grepable
                                      formats

  `-v`                                Verbose output

  `-d`                                Debug output
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## Complete Practical Scanning Workflow

The following sequence is a guided example for a lab target. Confirm the
real target address and scope first.

### Step 1 --- Verify the target and save a results directory

``` bash
export TARGET="192.168.56.101"
mkdir -p results
ip route
```

### Step 2 --- Discover hosts in the lab subnet

``` bash
sudo nmap -sn 192.168.56.0/24 -oA results/lab-discovery
```

### Step 3 --- Scan common TCP ports

``` bash
sudo nmap -sS --top-ports 1000 --reason "$TARGET" \
  -oA results/target-common-tcp
```

### Step 4 --- Scan all TCP ports

``` bash
sudo nmap -sS -p- --open "$TARGET" \
  -oA results/target-all-tcp
```

### Step 5 --- Enumerate discovered TCP services

Replace the example ports with the ports actually found:

``` bash
sudo nmap -sV -sC -p 21,22,80,139,445 "$TARGET" \
  -oA results/target-service-enum
```

### Step 6 --- Check selected UDP services

``` bash
sudo nmap -sU -sV -p 53,123,161 "$TARGET" \
  -oA results/target-udp
```

### Step 7 --- Estimate the operating system

``` bash
sudo nmap -O --osscan-limit "$TARGET" \
  -oA results/target-os
```

### Step 8 --- Perform focused web enumeration (if HTTP is open)

``` bash
sudo nmap -p 80,443,8080 \
  --script http-title,http-server-header "$TARGET" \
  -oA results/target-web
```

### Step 9 --- Review evidence

``` bash
ls -lh results/
```

Review the `.nmap` file for human-readable results and the `.xml` file
for structured data. Do not report a service, version, OS, or
vulnerability as confirmed unless the evidence supports that level of
certainty.

------------------------------------------------------------------------

## Student Practical Exercise

### Scenario

You are conducting an authorized internal assessment of your isolated
CODENSEC lab. Your assigned target is Metasploitable 2. Your objective
is to map exposed services and document what you observe---without
exploiting the machine.

### Student tasks

1.  Confirm Kali's IP address and lab subnet.
2.  Discover active hosts in the lab subnet using `-sn`.
3.  Identify the Metasploitable 2 host in the discovery output.
4.  Scan the top 100 TCP ports.
5.  Perform a full TCP port scan.
6.  Run version detection against discovered open ports.
7.  Run a focused OS-detection attempt and record whether it is
    conclusive.
8.  Scan selected UDP ports: 53, 123, and 161.
9.  If HTTP is open, run `http-title` and `http-server-header`.
10. Save all key scans using `-oA`.

### Suggested command starter

``` bash
export TARGET="192.168.56.101"
mkdir -p results

sudo nmap -sn 192.168.56.0/24 -oA results/discovery
sudo nmap -sS --top-ports 100 "$TARGET" -oA results/top100
sudo nmap -sS -p- --open "$TARGET" -oA results/all-tcp
```

Continue with targeted `-sV`, `-O`, UDP, and web checks based on your
results.

### Student report template

  Field                                Student notes
  ------------------------------------ ---------------
  Tester / team                        
  Date and time                        
  Nmap version                         
  Source IP                            
  Target IP                            
  Scope / authorization                
  Host discovery result                
  Open TCP ports                       
  Open / open\|filtered UDP results    
  Identified services and versions     
  OS detection result and confidence   
  Web server observations              
  Limitations / filtered ports         
  Evidence filenames                   
  Recommended next validation step     

### Discussion questions

1.  Why can a host be online but absent from a basic discovery scan?
2.  What is the key difference between `-sS` and `-sT`?
3.  Why can UDP ports remain `open|filtered`?
4.  Does `-p-` scan all UDP ports? Explain.
5.  Why should an OS fingerprint be reported as an estimate?
6.  Why is `-oA` useful during a professional assessment?
7.  Why should NSE scripts be reviewed before execution?

------------------------------------------------------------------------

## Troubleshooting

  ------------------------------------------------------------------------------------------------
  Problem                     Possible cause          What to check
  --------------------------- ----------------------- --------------------------------------------
  `nmap: command not found`   Nmap is not installed   `sudo apt update && sudo apt install nmap`
                              or not in PATH          

  Target appears down         Discovery probes        Confirm IP/route; test approved discovery
                              filtered, wrong IP, VM  methods; use `-Pn` only for confirmed
                              network mismatch        in-scope hosts

  SYN scan privilege warning  Not running with        Use `sudo`, or use `-sT` if appropriate
                              required privileges     

  All ports show filtered     Firewall, routing, or   Verify target reachability and lab
                              filtering device        firewall/network mode

  UDP results are slow        UDP probes often        Start with relevant ports; avoid unnecessary
                              require retries and     all-port UDP scans
                              ICMP feedback may be    
                              limited                 

  Version is unknown          Service does not return Check the correct port, try focused `-sV`,
                              a recognizable response and validate manually
                              or uses a custom        
                              protocol                

  OS detection is             Insufficient responses, Try after identifying useful open/closed
  inconclusive                filtering, or unusual   ports; treat guesses cautiously
                              network stack           

  NSE script not found        Script database or      Check `/usr/share/nmap/scripts/`; run
                              package files may be    `sudo nmap --script-updatedb`
                              outdated/missing        

  Scan takes too long         Large scope, all ports, Narrow scope/ports, use justified timing,
                              UDP, high retries, or   and split discovery from enumeration
                              slow timing             

  Output file not created     Directory missing or    `mkdir -p results`; write to a directory you
                              insufficient write      can access
                              permission              
  ------------------------------------------------------------------------------------------------

### Useful diagnostic commands

``` bash
nmap --version
ip -br addr
ip route
ping -c 2 192.168.56.101
sudo nmap -sn --reason 192.168.56.101
sudo nmap -Pn -sT -p 22,80,443 192.168.56.101
```

A failed `ping` does not prove a host is down. ICMP may be blocked while
TCP services remain reachable.

------------------------------------------------------------------------

## Quick-Reference Commands

  -----------------------------------------------------------------------------------------------------------------
  Goal                                Command
  ----------------------------------- -----------------------------------------------------------------------------
  List targets only                   `nmap -sL 192.168.56.0/24`

  Discover live hosts                 `sudo nmap -sn 192.168.56.0/24`

  Skip host discovery                 `sudo nmap -Pn 192.168.56.101`

  SYN scan                            `sudo nmap -sS 192.168.56.101`

  TCP connect scan                    `nmap -sT 192.168.56.101`

  UDP scan                            `sudo nmap -sU 192.168.56.101`

  Top 100 ports                       `sudo nmap --top-ports 100 192.168.56.101`

  All TCP ports                       `sudo nmap -p- 192.168.56.101`

  Service/version detection           `sudo nmap -sV 192.168.56.101`

  OS detection                        `sudo nmap -O 192.168.56.101`

  Default NSE scripts                 `sudo nmap -sC 192.168.56.101`

  Combined advanced scan              `sudo nmap -A 192.168.56.101`

  Show reasons                        `sudo nmap --reason 192.168.56.101`

  Web title/header                    `sudo nmap -p 80,443 --script http-title,http-server-header 192.168.56.101`

  Save all formats                    `sudo nmap -sV 192.168.56.101 -oA results/scan`
  -----------------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

## Important Notes for the Live Session

-   Demonstrate the command's purpose before running it.
-   Confirm the target IP and authorization before every new scope.
-   Explain the difference between an observed result and an inference.
-   Start with discovery and common ports; use full scans when
    justified.
-   Avoid disruptive NSE categories in a shared or production
    environment.
-   Keep a clean record of commands and output for students to review.
-   Reset lab snapshots if a practical exercise changes target
    configuration.

## Further Reading

-   [Official Nmap Documentation](https://nmap.org/docs.html)
-   [Nmap Reference Guide](https://nmap.org/book/man.html)
-   [Nmap Network Scanning --- Official Book](https://nmap.org/book/)
-   [Nmap Host Discovery](https://nmap.org/book/man-host-discovery.html)
-   [Nmap NSE Usage and Script
    Categories](https://nmap.org/book/nse-usage.html)
-   [Nmap Output Formats](https://nmap.org/book/man-output.html)
-   [Kali Linux Nmap Package](https://www.kali.org/tools/nmap/)

------------------------------------------------------------------------

## CODENSEC Community

**CODENSEC --- Learn. Practice. Exploit. Elevate.**

Prepared for cybersecurity learners, aspiring penetration testers, and
the CODENSEC community.

**Writer:** Saadullah Abdul Wahid\
**YouTube:**
[https://www.youtube.com/@CodenSec_01](https://www.youtube.com/@CodenSec_01)

If you use or share this guide, retain the CODENSEC branding and writer
attribution. Adapt lab IP addresses and commands to your authorized
environment.

> **Practice responsibly. Think like a tester. Document like a
> professional.**
