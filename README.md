# TakumiPay — Stellar (Soroban)

Stellar Soroban smart contracts for TakumiPay. Features include: owner/admin roles, pausing, backend-signed merchant payment quotes with replay protection, platform fee treasury, per-token spending limits, generic transactions, point deposits, and timelocked withdrawals.

## Deployed Contracts (v2)

The contract has been deployed on the Stellar **testnet** network. Below are the details for the **v2** deployment (from [v2.json](file:///home/cstralpt/tkmy-contract/stellar/deployments/testnet/v2.json)).

| Field | Value |
| --- | --- |
| **Network** | `testnet` |
| **Version** | `2` |
| **Contract ID** | `CCLFTLVPHOKKDZYTMGU6UNXKFEN6VF3QVYEAJNULGIC7ZXTETAIPKKRZ` |
| **WASM Hash** | `6e2d4f603ac8167d70463f476798e288019db70c9b6c515112ec745b032be65b` |
| **Deployer** | `GCZ6IK35AZZU2DC5HLRSEE2I3F5YUBRTALUA3ILI7V2FV5KLT2OR4LWM` |
| **Owner** | `GCZ6IK35AZZU2DC5HLRSEE2I3F5YUBRTALUA3ILI7V2FV5KLT2OR4LWM` |
| **Backend Signer Pubkey** | `04466f114c2f959229d812cc68731c4625fb869e8eefcf47d3da251f82ba93b4` |
| **Published At** | `2026-07-14` |
| **WASM File** | `takumi_pay.optimized.wasm` |
| **Deploy Tx** | `5a2609e34b2e27d4f1f013119d64edd38c031de9a49f2577495a5cd3437b9b4e` |
| **Initialize Tx** | `d72b1661b640b5193c9ecf0eec5fddb513c2b058dc274f461360e6e6a6aa103f` |

### Changes Since V1
> [!NOTE]
> Merged `AllowedPointToken` into a single `AllowedPaymentToken` allowlist that now gates `create_transaction`, `process_merchant_payment`, AND `deposit_points` (v1 only gated `deposit_points`). `add_allowed_point_token`/`remove_allowed_point_token`/`is_allowed_point_token` were removed in favor of `add_allowed_payment_token`/`remove_allowed_payment_token`/`is_allowed_payment_token`. No upgrade path exists on this contract, so this is a fresh contract instance, not an in-place upgrade of v1 (`CAEVSB5RGLRR3MVXUNMG67JRA4AAMZH4GR5WNCODSITO6YQI2W7XWD32`).

---

## Directory Structure

All the Soroban smart contract source code and deployment scripts are located in the [stellar/](file:///home/cstralpt/tkmy-contract/stellar) directory.

```
stellar/
├── contracts/       # Soroban contract source files
├── deployments/     # Contract deployment records (JSON files)
├── scripts/         # Shell scripts for building/deploying/initializing
├── Cargo.toml       # Workspace definitions
└── Cargo.lock
```

## Quick Start (Stellar / Soroban)

For details on compiling, testing, and deploying, see the subfolder [stellar/README.md](file:///home/cstralpt/tkmy-contract/stellar/README.md).

### Build
To build the contract:
```sh
cd stellar
./scripts/build.sh
```

### Test
To run the test suite:
```sh
cd stellar
cargo test -p takumi-pay
```

---

## Mainnet Deployment Instructions

To deploy the TakumiPay contract to Stellar **mainnet**, follow these steps:

### 1. Build the Contract
Build the optimized WASM bytecode:
```sh
cd stellar
./scripts/build.sh
```

### 2. Deploy to Mainnet
Deploy the contract WASM using the Stellar CLI:
```sh
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/takumi_pay.optimized.wasm \
  --source <deployer-key-alias> \
  --network mainnet
```
This command will return the mainnet `<contract-id>`.

### 3. Initialize the Contract
Initialize the contract. The contract **owner** must be set to the specific address from the V2 deployment:
**`GCZ6IK35AZZU2DC5HLRSEE2I3F5YUBRTALUA3ILI7V2FV5KLT2OR4LWM`**

Run the initialization script (or execute `stellar contract invoke` directly):
```sh
./scripts/initialize.sh mainnet <deployer-key-alias> <contract-id> GCZ6IK35AZZU2DC5HLRSEE2I3F5YUBRTALUA3ILI7V2FV5KLT2OR4LWM <backend-signer-pubkey-hex>
```

### 4. Register Mainnet USDC as an Allowed Payment Token
On Stellar Mainnet, the official USDC asset is issued by Circle (`GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN`). Its corresponding Soroban Stellar Asset Contract (SAC) ID is:
**`CCW67TSZV3SSS2HXMBQ5JFGCKJNXKZM7UQUWUZPUTHXSTZLEO7SJMI75`**

To allow transaction processing, merchant payments, and point deposits with USDC, invoke `add_allowed_payment_token` on the contract. This call must be authorized and signed by the owner:
```sh
stellar contract invoke \
  --id <contract-id> \
  --source <owner-key-alias> \
  --network mainnet \
  -- \
  add_allowed_payment_token \
  --owner GCZ6IK35AZZU2DC5HLRSEE2I3F5YUBRTALUA3ILI7V2FV5KLT2OR4LWM \
  --token CCW67TSZV3SSS2HXMBQ5JFGCKJNXKZM7UQUWUZPUTHXSTZLEO7SJMI75
```

