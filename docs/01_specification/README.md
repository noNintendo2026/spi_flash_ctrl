# 01 — Specification

[← Back to README](../../README.md)

This chapter defines **what** the SPI-Flash Controller must do. It is the
input to every subsequent stage (architecture, design, verification,
integration).

## Contents

| Document | Purpose | Status |
|---|---|---|
| [requirements.md](requirements.md) | Functional and non-functional requirements | In progress |
| [spi_protocol.md](spi_protocol.md) | SPI protocol applied to Flash memory | Complete |
| [flash_memory.md](flash_memory.md) | Target Flash device specification | Pending |
| [flash_commands.md](flash_commands.md) | Supported command subset | Pending |
| [csr_interface.md](csr_interface.md) | Memory-mapped register interface | Pending |

## Diagrams

Protocol diagrams referenced by this chapter live in:

- [`diagrams/protocol/`](../../diagrams/protocol/) — transaction waveforms,
  operation flow, physical connection

## Next Chapter

→ [02 — Architecture](../02_architecture/)
