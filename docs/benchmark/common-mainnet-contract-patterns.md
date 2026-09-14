# Common mainnet contract patterns

This note supports [issue #3517](https://github.com/ethereum/execution-specs/issues/3517). It defines a reproducible way to select representative execution cases before adding them to the benchmark suite. It deliberately does not claim a frequency ranking: the repository currently has no attached mainnet sample or measurement artifact from which such a ranking could be derived.

## What should be sampled

A useful sample should describe the execution shape, not only the application name. The first tranche should cover these patterns:

| Pattern | Execution pressure | Candidate EELS surface |
| --- | --- | --- |
| ERC-20 transfer / approve | warm and cold storage reads/writes; logs | SLOAD, SSTORE, LOG |
| ERC-20 transferFrom | allowance update plus balance updates | nested storage access and revert paths |
| AMM swap with a callback | external call, callback, and final invariant check | CALL, returndata, storage |
| Multicall / router batch | repeated calls with partial failure handling | call depth and revert propagation |
| Proxy dispatch | calldata forwarding and delegated storage access | DELEGATECALL, return data |
| Contract creation | initcode execution and code deposit | CREATE, memory expansion, code size |

These are execution-pattern categories, not claims that any particular deployed contract is representative. A future data-backed update can attach chain, block range, selector frequencies, and sample addresses without changing the category names.

## Evidence record

Each proposed case should carry a small evidence record next to its test or fixture:

pattern: erc20-transfer
source: mainnet-trace
chain: ethereum
block_range: <measured range>
selector: 0xa9059cbb
contracts: <sample size>
selection: <query or reproducible procedure>
expected_surface: [SLOAD, SSTORE, LOG]
privacy: public execution data only

The measured range and sample size are intentionally placeholders until a contributor supplies the measurement. This prevents an unverified frequency statement from becoming part of the specification repository.

## From a pattern to a benchmark

1. Record the selection procedure and preserve the raw input or a content hash.
2. Reduce one trace to the smallest transaction setup that retains the execution shape.
3. Map the retained operations to an existing benchmark helper under tests/benchmark.
4. Keep one scenario per execution pressure. Do not mix a transaction-type regression with an unrelated gas benchmark.
5. Compare the generated gas expectation with the fork-specific implementation and record the fork in the test name.
6. Add a regression assertion for the failure path when the pattern relies on revert propagation or an access-list distinction.

This separation makes a benchmark explainable: reviewers can tell which part is derived from observed execution and which part is a synthetic stress parameter.

## Acceptance checklist

A pattern is ready for a benchmark PR when:

- the evidence record is reproducible and does not expose private data;
- the test names the EVM mechanism under pressure;
- the setup and attack path are independent and readable;
- expected gas is derived from the fork API rather than copied from a client;
- the case has a positive path and, where relevant, a revert or boundary path;
- the PR states whether the input is observed, reduced, or synthetic.

This document is a contribution guide for #3517, not a substitute for measured mainnet data. The next increment should add one evidence-backed pattern and its benchmark in a separate, reviewable change.
