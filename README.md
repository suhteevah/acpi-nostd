# acpi-nostd

[![Crates.io](https://img.shields.io/crates/v/acpi-nostd.svg)](https://crates.io/crates/acpi-nostd)
[![docs.rs](https://docs.rs/acpi-nostd/badge.svg)](https://docs.rs/acpi-nostd)
[![License](https://img.shields.io/crates/l/acpi-nostd.svg)](https://github.com/suhteevah/acpi-nostd#license)

A `#![no_std]` ACPI table parser with power management for bare-metal Rust operating systems and kernels.

## Features

- **RSDP v1/v2** discovery from BIOS memory regions or UEFI configuration table
- **RSDT / XSDT** root table parsing with automatic table enumeration
- **MADT** (Multiple APIC Description Table) for SMP CPU and I/O APIC discovery
- **FADT** (Fixed ACPI Description Table) with all fields through ACPI 6.x
- **MCFG** (PCI Express ECAM) for memory-mapped PCIe configuration access
- **HPET** (High Precision Event Timer) with register access helpers
- **Power management**: ACPI enable/disable, shutdown (S5), reboot, sleep states, PM timer delays
- **Checksum validation** on all parsed tables
- Depends only on `log` and `alloc` (no `std`)

## Usage

Add to your `Cargo.toml`:

```toml
[dependencies]
acpi-nostd = "0.1"
```

### Initialize from a UEFI RSDP address

```rust,no_run
use acpi_nostd::{AcpiTables, Rsdp};

let rsdp_addr: u64 = /* from UEFI config table */;
# let rsdp_addr: u64 = 0;

unsafe {
    let tables = acpi_nostd::init_from_rsdp_addr(rsdp_addr).unwrap();

    // Enumerate discovered tables
    for (sig, addr) in &tables.tables {
        log::info!("Found ACPI table '{}' at {:#X}", sig, addr);
    }
}
```

### Parse the MADT for CPU cores

```rust,no_run
use acpi_nostd::madt::Madt;

# let madt_addr: u64 = 0;
unsafe {
    let madt = Madt::from_address(madt_addr).unwrap();
    let cpus = madt.local_apics();
    log::info!("{} CPUs discovered", cpus.len());

    for ioapic in madt.io_apics() {
        log::info!("I/O APIC {} at {:#X}", ioapic.io_apic_id, ioapic.io_apic_address);
    }
}
```

### ACPI shutdown

```rust,no_run
use acpi_nostd::{Fadt, PowerManager};

# let fadt_addr: u64 = 0;
# let dsdt_addr: u64 = 0;
unsafe {
    let fadt = Fadt::from_address(fadt_addr).unwrap();
    let mut pm = PowerManager::new(fadt);

    // Parse S5 sleep type from DSDT (required before shutdown)
    pm.parse_s5_from_dsdt(dsdt_addr).unwrap();

    // Power off
    pm.shutdown().unwrap();
}
```

### PCIe ECAM via MCFG

```rust,no_run
use acpi_nostd::mcfg::Mcfg;

# let mcfg_addr: u64 = 0;
unsafe {
    let mcfg = Mcfg::from_address(mcfg_addr).unwrap();
    if let Some(entry) = mcfg.find_segment(0, 0) {
        let cfg_addr = entry.config_address(0, 2, 0).unwrap();
        log::info!("PCIe config space for 00:02.0 at {:#X}", cfg_addr);
    }
}
```

## Supported ACPI Tables

| Signature | Structure | Description |
|-----------|-----------|-------------|
| `RSDP`    | `Rsdp`    | Root System Description Pointer (entry point) |
| `RSDT`    | `Rsdt`    | Root System Description Table (32-bit pointers) |
| `XSDT`    | `Xsdt`    | Extended System Description Table (64-bit pointers) |
| `APIC`    | `Madt`    | Multiple APIC Description Table (CPUs, I/O APICs, interrupt overrides) |
| `FACP`    | `Fadt`    | Fixed ACPI Description Table (power management registers) |
| `MCFG`    | `Mcfg`    | PCI Express Memory-Mapped Configuration (ECAM base addresses) |
| `HPET`    | `Hpet`    | High Precision Event Timer |
| `DSDT`    | (parsed by `PowerManager`) | Differentiated System Description Table (S5 sleep type extraction) |

## Requirements

- `#![no_std]` environment with a global allocator (`extern crate alloc`)
- Physical memory containing ACPI tables must be identity-mapped or otherwise accessible via raw pointers
- For power management I/O port access: x86/x86_64 target (uses `in`/`out` instructions)

## License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or <http://www.apache.org/licenses/LICENSE-2.0>)
- MIT License ([LICENSE-MIT](LICENSE-MIT) or <http://opensource.org/licenses/MIT>)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in this work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without any additional terms or conditions.

---

---

---

---

---

---

---

## Support This Project

If you find this project useful, consider buying me a coffee! Your support helps me keep building and sharing open-source tools.

[![Donate via PayPal](https://img.shields.io/badge/Donate-PayPal-blue.svg?logo=paypal)](https://www.paypal.me/baal_hosting)

**PayPal:** [baal_hosting@live.com](https://paypal.me/baal_hosting)

Every donation, no matter how small, is greatly appreciated and motivates continued development. Thank you!
