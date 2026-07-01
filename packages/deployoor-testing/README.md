# @deployoor/testing

> Test your deployoor deploys against a real in-memory EVM — no local node.

`createTestClients()` boots [tevm](https://tevm.sh) in-process and hands you ordinary viem wallet/public clients. Pass them straight to a generated deployer and your test deploys real contracts to a real EVM — no `hardhat node`, no anvil, no RPC. The tevm version is pinned by this package, so you never fight a version mismatch.

```bash
pnpm add -D @deployoor/testing
```

```ts
import { createTestClients } from "@deployoor/testing";
import { getOrDeployToken } from "../deployers";

it("deploys the token", async () => {
  const { walletClient, publicClient } = await createTestClients();
  const token = await getOrDeployToken({ walletClient, publicClient, args: [owner] });
  // it's a live contract on an in-memory chain — read/write against it
  expect(token.address).toMatch(/^0x/);
});
```

`createTestClients()` returns `{ account, accounts, chain, walletClient, publicClient, walletClientFor }` — `account` is the first prefunded account (bound to `walletClient`), and `chain.name` is what keys the `deployments/` records deployoor writes.

### Multiple accounts

`accounts` holds every prefunded account, and `walletClientFor(account)` builds a wallet client bound to any of them on the same EVM — for testing several addresses interacting:

```ts
const { accounts, walletClientFor } = await createTestClients();
const [owner, alice] = accounts;

const token = await getOrDeployToken({
  walletClient: walletClientFor(owner),
  publicClient,
  args: [owner.address],
});
await token.write.transfer([alice.address, 100n]); // owner sends
// alice acts with her own signer:
await token.write.approve([spender, 50n], { account: alice });
```

### tevm options

Pass tevm's `createMemoryClient` options straight through — e.g. fork a live chain, or change mining:

```ts
import { http } from "viem";
const { publicClient } = await createTestClients({ fork: { transport: http(process.env.MAINNET_RPC) } });
```

Mining defaults to `"auto"`; override it in the same options object.
