# Parapet Finances

A local-first, open-source personal finance and wealth management application built on the principle that your financial data belongs to you — and only you.

## Why Parapet?

A parapet is the fortified wall you stand on to see clearly. That's what this application is: a secure vantage point over your complete financial picture — budgets, debt, investments, taxes — without surrendering your data to anyone.

Most financial tools ask you to trust a company with your most sensitive information. Parapet takes a different approach: **your data never leaves your devices, and decrypting it requires your physical presence.**

## Core Principles

### 🔒 Security First

Security isn't a feature of Parapet — it's the foundation everything else is built on.

- **Hardware security keys required** — Every user must authenticate with a password AND a YubiKey. There is no fallback to SMS, email codes, or authenticator apps. This is a deliberate, non-negotiable design decision.
- **Local encryption** — All financial data is stored in a SQLCipher encrypted database on your device. The encryption key is a random 256-bit value that can only be unlocked by combining something you **know** (your password) with something you **have** (your physical YubiKey).
- **No server stores your data** — There is no cloud database holding your financial records. Sync between your devices is end-to-end encrypted. Even if you use the optional cloud relay, the server stores only encrypted blobs it cannot read.
- **No background access** — The database is only decrypted when you are actively using the application with your hardware key present. When the app is closed, your data is an unreadable blob on disk.

An attacker would need to simultaneously possess your device, your YubiKey, and your password to access your data. That's a higher security bar than most banking applications.

### 🏠 Data Ownership

You own your financial data. Completely and without compromise.

- **Local-first architecture** — Your device is the source of truth, not a server you don't control.
- **No accounts to create** — There is no sign-up, no email collection, no user database on our end.
- **No vendor lock-in** — Your data is stored in an open format (SQLite). With your encryption key, you can always access it directly.
- **Rebuildable by design** — Parapet aggregates data from your financial institutions, files, and manual entries. If you ever lose access to your database, the data can be reconstructed from its original sources.
- **Open source** — The code is public. The encryption is auditable. You don't have to trust our claims — you can verify them.

### 📊 Complete Financial Picture

Parapet is designed to answer the questions that matter:

- **What is my budget and where do I stand?** — Track spending against budgets by category with real-time status.
- **What is my income versus expenses?** — See cash flow for any time period across all accounts.
- **How much am I paying in interest?** — Break down debt payments into principal, interest, and fees to understand the true cost of borrowing.
- **What are my investment returns?** — Track performance across individual holdings or entire portfolios with time-weighted return calculations.
- **How much have I spent on taxes?** — Categorize and total property tax, school tax, income tax, and other tax payments across the year.

## Architecture

Parapet is a cross-platform .NET MAUI application with a clear separation between the desktop and mobile experience.

### Hub and Spoke Model

```
Mobile (Spoke)                    Desktop (Hub)
┌────────────────────┐           ┌────────────────────────────┐
│ Capture receipts   │──push──→  │ Import files & bank data   │
│ Quick transactions │           │ Reconcile & categorize     │
│ View your finances │←──pull──  │ AI analysis & reporting    │
│ Propose edits      │           │ Review & approve changes   │
└────────────────────┘           └────────────────────────────┘
```

- **Desktop (Windows / macOS)** — The full workstation experience. Import transactions from banks (via Plaid), CSV, Excel, and QuickBooks files. Reconcile data, manage budgets, review investment performance, and run AI-powered analysis. The desktop is the authoritative source of truth.
- **Mobile (Android / iOS)** — A lightweight capture and viewing tool. Photograph receipts, log quick transactions on the go, and review your financial status. Edits made on mobile are submitted as proposals and reconciled on the desktop where screen real estate supports informed decisions.

### Event Sourcing

Every financial action is stored as an immutable event in an append-only log. Current balances, budget status, and investment performance are **projections** — materialized views derived by replaying events. This architecture provides:

- **Complete audit trail** — Every change is recorded. Nothing is overwritten.
- **Point-in-time views** — Replay events to see your financial state as of any date.
- **Safe rebuilds** — If a projection has a bug, fix the code and rebuild from events. No data is lost.
- **Natural sync model** — Devices exchange events, not state. Sync is append-only with no complex conflict resolution.

### Sync Options

| Tier | How it works | Cost |
|------|-------------|------|
| **Local Network** | Devices discover each other on your home network via mDNS and sync directly over mutual TLS. Your data never touches a server. | Free |
| **Cloud Relay** | End-to-end encrypted blobs are stored on a relay server. The server cannot read your data. Sync works across any network. | Included with Plaid subscription |

Both tiers use the same sync engine. The only difference is the transport.

## Technology

- **.NET MAUI** — Cross-platform UI for Windows, macOS, Android, and iOS
- **SQLCipher** — AES-256 encrypted SQLite database
- **YubiKey (HMAC-SHA1 Challenge-Response)** — Hardware-bound encryption keys
- **Argon2id** — Memory-hard key derivation
- **AES-256-GCM** — Key wrapping with authenticated encryption
- **MediatR** — CQRS command/query dispatch
- **ONNX Runtime** — Local AI models for receipt OCR and transaction categorization

## Security Model

```
Password → Argon2id → Challenge → YubiKey HMAC → HKDF → Key Encryption Key
                                                              │
                            ┌─────────────────────────────────┘
                            │
                            ▼
                   Unwrap Master Database Key (AES-256-GCM)
                            │
                            ▼
                   Decrypt SQLCipher Database
```

- The **Master Database Key** is a random 256-bit key generated at setup. It is never stored in plaintext.
- Each registered YubiKey holds an independently wrapped copy of the master key (key registration model, similar to LUKS).
- A minimum of **two YubiKeys** must be registered (primary + backup).
- Losing both YubiKeys means losing access to the local database — but since Parapet is an aggregator, the data can be rebuilt from source institutions.

## Project Structure

```
ParapetFinances/
├── src/
│   ├── Parapet.Domain/            # Entities, events, value objects — zero dependencies
│   ├── Parapet.Application/       # CQRS commands, queries, interfaces
│   ├── Parapet.Infrastructure/    # SQLCipher, encryption, Plaid, sync, AI
│   ├── Parapet.UI.Shared/         # Shared MAUI controls, styles, view models
│   ├── Parapet.Desktop/           # Desktop MAUI app (hub)
│   └── Parapet.Mobile/            # Mobile MAUI app (spoke)
├── tests/
│   ├── Parapet.Domain.Tests/
│   ├── Parapet.Application.Tests/
│   ├── Parapet.Infrastructure.Tests/
│   └── Parapet.Integration.Tests/
├── docs/
│   ├── architecture/              # Detailed design documentation
│   └── decisions/                 # Architecture Decision Records
└── ParapetFinances.sln
```

## Getting Started

### Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download) with MAUI workloads installed
- Two [YubiKey](https://www.yubico.com/) hardware security keys (USB-C, NFC, or Lightning depending on your devices)
- YubiKey Slot 2 configured for HMAC-SHA1 Challenge-Response (via [YubiKey Manager](https://www.yubico.com/support/download/yubikey-manager/))

### Build

```bash
git clone https://github.com/tj-cappelletti/parapet-finances.git
cd parapet-finances/ParapetFinances
dotnet build
```

### First Run

1. Launch the desktop application
2. Create a password (minimum 16 characters)
3. Register your primary YubiKey
4. Register your backup YubiKey
5. Your encrypted database is created and ready

## Contributing

Parapet is open source and contributions are welcome. Please read the architecture documentation in `docs/` before submitting changes to understand the security model and design decisions.

Security-critical code (encryption, key derivation, hardware key interaction) requires thorough review. If you find a security vulnerability, please report it responsibly.

## License

This project is licensed under the [Apache License 2.0](LICENSE).

Apache 2.0 was chosen over MIT specifically for its explicit patent protections. Parapet implements cryptographic operations, key derivation schemes, and hardware security key integrations — areas where the patent landscape is active. Apache 2.0 ensures that every contributor grants a perpetual, irrevocable patent license for their contributions, and includes a retaliation clause that terminates the license of anyone who initiates patent litigation against the project. This protects both the maintainers and the community using Parapet.
