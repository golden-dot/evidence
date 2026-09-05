# Evidence Adjudicator

Evidence Adjudicator is a reusable GenLayer Intelligent Contract primitive for reaching consensus on claims derived from live public web sources.

Instead of trusting a single AI response, the contract has a leader independently retrieve and evaluate source material and has validators independently reproduce the evaluation. Consensus is accepted only when the consensus-critical semantic fields agree.

## Why GenLayer?

Traditional smart contracts can store and verify deterministic data, but they cannot directly establish many real-world facts from changing web content.

Evidence Adjudicator uses GenLayer's nondeterministic execution model to:

1. Retrieve public source material.
2. Evaluate the source against a caller-defined question.
3. Apply caller-defined verification criteria.
4. Independently repeat the evaluation through validators.
5. Compare stable decision fields.
6. Persist the consensus-backed result.

The contract does not claim that a source is objectively true. It establishes consensus about what the supplied source supports, refutes, or cannot establish under the supplied criteria.

## Contract State

Each verification case contains:

* `case_id`
* `question`
* `source_url`
* `verification_criteria`
* `status`
* `decision`
* `evidence`
* `source_relevant`
* `evidence_found`

The lifecycle is:

`PENDING -> RESOLVED`

A case cannot be resolved twice.

## Consensus Design

The nondeterministic operation is executed through `gl.vm.run_nondet_unsafe`.

The leader:

* retrieves the source;
* evaluates the question;
* returns a structured decision.

The validator:

* independently retrieves the source;
* independently evaluates the same question and criteria;
* does not trust the leader's reasoning;
* compares the consensus-critical fields.

The fields that must agree are:

* `decision`
* `source_relevant`
* `evidence_found`

The natural-language `evidence` explanation is deliberately not an equivalence requirement because independent LLM executions may express the same conclusion differently.

This follows GenLayer's Equivalence Principle approach of comparing semantic decision fields rather than requiring identical reasoning text.

## Decisions

### SUPPORTED

The source provides sufficient evidence satisfying the verification criteria.

### REFUTED

The source provides sufficient evidence contradicting the claim or failing an explicitly applicable requirement.

### INCONCLUSIVE

The source does not provide enough evidence to reach either conclusion.

## Security and Robustness

The contract:

* accepts HTTPS sources only;
* rejects duplicate case IDs;
* rejects empty questions;
* rejects empty verification criteria;
* validates structured LLM responses;
* treats webpage instructions as untrusted source content;
* requires independent validator agreement before state is written.

## Example Use Cases

The primitive is intentionally domain-agnostic.

A builder could use it to verify:

* whether public documentation contains a required feature;
* whether a policy page satisfies a defined requirement;
* whether an official announcement establishes a particular event;
* whether a public document supports or contradicts a stated claim.

## Example

```python
create_case(
    "docs-check-1",
    "Does the source state that the protocol supports feature X?",
    "https://example.com/documentation",
    "The source must explicitly state support for feature X."
)
```

Then:

```python
resolve_case("docs-check-1")
```

The resulting state can be read using:

```python
get_case("docs-check-1")
```

## Testing

The repository should include:

### Direct tests

Fast in-memory tests for:

* valid case creation;
* duplicate prevention;
* invalid URLs;
* missing questions;
* missing criteria;
* case views;
* resolution state handling.

### Integration test

A Studio test should exercise:

`create_case -> resolve_case -> get_case`

with real GenLayer consensus.

## Validation

Use the GenLayer tooling to lint the contract and run direct tests before deployment.

```bash
genvm-lint check contracts/evidence_adjudicator.py
pytest tests/direct/ -v
gltest tests/integration/ -v -s
```

## Design Goal

Evidence Adjudicator is intended as infrastructure for other GenLayer builders rather than a single-purpose application.

The contract separates:

* the external evidence source,
* the verification question,
* the verification policy,
* the consensus decision,
* and the resulting evidence record.

That makes the same primitive usable across multiple verification workflows.
