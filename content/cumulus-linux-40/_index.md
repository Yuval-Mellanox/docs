---
title: Cumulus Linux 4.0 User Guide
weight: -40
subsection: true
cascade:
    product: Cumulus Linux
    version: "4.0"
    old: true
    author: NVIDIA
toc: 1
---
## What is Cumulus Linux?

NVIDIA® Cumulus Linux is the first full-featured Linux operating system for the networking industry. The {{<exlink url="https://www.debian.org/releases/buster/" text="Debian Buster" >}}-based, networking-focused distribution runs on hardware produced by a {{<exlink url="https://cumulusnetworks.com/hcl/" text="broad partner ecosystem" >}}, ensuring unmatched customer choice regarding silicon, optics, cables, and systems.

This user guide provides in-depth documentation on the Cumulus Linux installation process, system configuration and management, network solutions, and monitoring and troubleshooting recommendations. In addition, the quick start guide provides an end-to-end setup process to get you started.

Cumulus Linux 4.0 includes the latest NetQ agent and CLI, which is installed by default on the Cumulus Linux switch. Use NetQ to monitor and manage your data center network infrastructure and operational health. Refer to the {{<kb_link url="cumulus-netq-33/" text="NetQ documentation" >}} for details.

For a list of the new features in this release, see {{<link url="Whats-New" text="What's New">}}. For bug fixes and known issues present in this release, refer to the {{<link url="Cumulus-Linux-4.0-Release-Notes" text="Cumulus Linux 4.0 Release Notes">}}.

### Open Source Contributions

To implement various Cumulus Linux features, Cumulus Networks has forked various software projects, like CFEngine `Netdev` and some Puppet Labs packages. Some of the forked code resides in the Cumulus Networks {{<exlink url="https://github.com/CumulusNetworks" text="GitHub repository" >}} and some is available as part of the Cumulus Linux repository as Debian source packages.

### Hardware Compatibility List

You can find the most up-to-date hardware compatibility list (HCL) {{<exlink url="https://cumulusnetworks.com/hcl/" text="here" >}}. Use the HCL to confirm that your switch model supports by Cumulus Linux. The HCL is updated regularly, listing products by port configuration, manufacturer and SKU part number.

### Stay up to Date

- Subscribe to our {{<exlink url="https://lists.cumulusnetworks.com/listinfo/cumulus-product-bulletin" text="product bulletin" >}} mailing list to receive important announcements and updates about issues that arise in our products.
- Subscribe to our {{<exlink url="https://lists.cumulusnetworks.com/listinfo/cumulus-security-announce" text="security announcement" >}} mailing list to receive alerts whenever we update our software for security issues.

## PDF Documents
You can download a PDF version of the complete Cumulus Linux {{% version %}} user guide {{% pdf_link "here." %}}