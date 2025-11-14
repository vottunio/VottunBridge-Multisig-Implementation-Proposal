# M3 Testing Guide - Test Wallets & Seeds

**⚠️ WARNING: FOR TESTNET USE ONLY - DO NOT USE ON MAINNET**  
**⚠️ These are test wallets with no funds on mainnet - newly created for testing purposes only**

---

## Contract Information

**EVM Bridge Contract (Base Sepolia)**:  
`0xbC79b4a96186b0AFE09Ee83830e2Fb30E14d5Ddc`  
**Explorer**: https://sepolia.basescan.org/address/0xbC79b4a96186b0AFE09Ee83830e2Fb30E14d5Ddc

---

## Qubic Network - Test Wallets

### Admins (3 wallets)

#### Admin 1
- **Address**: `XABEFABIHWRWBAIJQJPWTIIQBUCBHBVWYYGFFJADQBKWFBORRVXWSCVBUKWL`
- **Seed**: `xfccearcehvadeokstomhjsyqaudyhkngbmlwvsxsppexhlhdvijyaq`

#### Admin 2
- **Address**: `EQMBBVYGZOFUIHEXFOXKTFTANEKBXLBXHAYDFFMREEMRQEVADYMMEWACTODD`
- **Seed**: `xpsxzzfqvaohzzwlbofvqkqeemzhnrscpeeokoumekfodtgzmwghtqm`

#### Admin 3
- **Address**: `HYJXEZSECWSKODJALRCKSLKVYUEBMAHDODYZUJIIYDPAGFKLMOTHTJXEBEWM`
- **Seed**: `ukzbkszgzpipmxrrqcxcppumxoxzerrvbjgthinzodrlyblkedutmsy`

### Managers (1 wallet)

#### Manager 1
- **Address**: `XABEFABIHWRWBAIJQJPWTIIQBUCBHBVWYYGFFJADQBKWFBORRVXWSCVBUKWL`
- **Note**: Same address as Admin 1 (can be both admin and manager for quick testing)

---

## EVM Network (Base Sepolia) - Test Wallets

### Wallet Configuration (JSON)

```json
{
  "generated": "2025-11-05T18:19:51.180Z",
  "network": "Base Sepolia Testnet",
  "warning": "FOR TESTNET USE ONLY - DO NOT USE ON MAINNET",
  "wallets": [
    {
      "role": "Admin 1 IN USE",
      "address": "0xDb29Aedd947eBa1560dd31CffEcf63bbB817aB4A",
      "privateKey": "0xc51455368051f8cc0c65afdd532b79debfb16d6b1e5a9e6898c72e67ad8e4d00",
      "mnemonic": "athlete rigid resist virus strike country music always random tiny love swear"
    },
    {
      "role": "Admin 2 IN USE",
      "address": "0x7002b4761B7B836b20F07e680b5B95c755197102",
      "privateKey": "0x232e0a3925b5cad3f5f5ff768ef3f47899a5a05f39a8433b06ccd9876059ff74",
      "mnemonic": "narrow auto twenty border infant exclude steel cause add mimic snack island"
    },
    {
      "role": "Admin 3 FREE",
      "address": "0xd661A118E93E24608C2c4768e232E38F45F1CA33",
      "privateKey": "0x9a6348991060dd98869e25f5088c3e8bd929538fd1593e8f7c17dffb62a31070",
      "mnemonic": "venture choose cruise blanket energy age bag tooth frown vapor deliver pupil"
    },
    {
      "role": "Manager 1 IN USE",
      "address": "0xcbEe35eCAaBbff134f9a236E00DdF14963D2Ed24",
      "privateKey": "0x28b18b5576a9b52a9259443e44f6f25b293061914dba1542f4009344d962d3ae",
      "mnemonic": "sort invest skin sister tribe myth giggle sadness trick timber cabbage abstract"
    },
    {
      "role": "Manager 2 IN USE",
      "address": "0xB2CE6EC2E25984397a51509aB57303FE4aDf667A",
      "privateKey": "0x107ef66060c68d6f551b7ccd281229254f292bb6572fca4813bbe02c78968519",
      "mnemonic": "rose approve drive update reform calm physical reform roast enroll boat enjoy"
    },
    {
      "role": "Operator 1 IN USE",
      "address": "0xad6719141371952d926f5f2d72Ed6bc636d77C3a",
      "privateKey": "0x5cca86de99e21e298d4e3fefe2c00f688ddc8f9958c6ec24564944392b8fe070",
      "mnemonic": "install asthma film explain pumpkin tortoise amazing humble display exotic hen parent"
    }
  ]
}
```

### EVM Wallets Summary

#### Admins (3 wallets)
- **Admin 1**: `0xDb29Aedd947eBa1560dd31CffEcf63bbB817aB4A`
- **Admin 2**: `0x7002b4761B7B836b20F07e680b5B95c755197102`
- **Admin 3**: `0xd661A118E93E24608C2c4768e232E38F45F1CA33`

#### Managers (2 wallets)
- **Manager 1**: `0xcbEe35eCAaBbff134f9a236E00DdF14963D2Ed24`
- **Manager 2**: `0xB2CE6EC2E25984397a51509aB57303FE4aDf667A`

#### Operators (1 wallet)
- **Operator 1**: `0xad6719141371952d926f5f2d72Ed6bc636d77C3a`

---

## Testing Notes

### Importing Wallets

**Qubic Wallets**:
- Use the Qubic address and private key with MetaMask Snap for Qubic network

**EVM Wallets**:
- Import to MetaMask using either:
  - Private key (prefixed with `0x`)
  - Mnemonic (seed phrase) - 12 words

### Network Configuration

- **EVM Network**: Base Sepolia (Chain ID: 84532)
- **RPC URL**: Use Base Sepolia public RPC or configure in MetaMask
- **Qubic Network**: Connect via MetaMask Snap

### Testing Workflow

1. Import wallets to MetaMask (both EVM and Qubic)
2. Switch to Base Sepolia network for EVM operations
3. Connect Qubic wallet via MetaMask Snap
4. Navigate to Admin dashboard (`/admin`)
5. Test multisig workflows:
   - Create proposal with Admin 1
   - Approve with Admin 2
   - Execute when threshold met (2-of-3)

---

**⚠️ REMINDER: These wallets are for TESTNET ONLY. Never use these private keys or seeds on mainnet!**
