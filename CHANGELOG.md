# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-04-02

### Added

- RSDP v1/v2 discovery from BIOS memory regions and UEFI configuration table
- RSDT and XSDT root table parsing with automatic SDT enumeration
- MADT parser: Local APIC, I/O APIC, Interrupt Source Override, NMI, x2APIC, I/O SAPIC
- FADT parser with all fields through ACPI 6.x including Generic Address Structures
- MCFG parser for PCIe ECAM base address discovery
- HPET parser with register offset constants and counter enable/disable/read helpers
- Power management: ACPI enable/disable, S5 shutdown, reboot via reset register
- PM timer microsecond delay using 3.579545 MHz ACPI timer
- Sleep state entry (S1-S4) via PM1 control register
- S5 sleep type extraction from DSDT AML bytecode
- Checksum validation on all parsed tables
