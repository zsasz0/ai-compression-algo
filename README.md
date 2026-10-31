# CAIE: Context AI Encoding

Context AI Encoding (CAIE) is a proposed lossless text-compression method that uses shared context to try to reconstruct omitted text. An omission is accepted only when the decoder reproduces the original bytes exactly.

## Methodology

1. **Preserve the source.** Keep the original byte sequence unchanged; define the character encoding and preserve details such as line endings.
2. **Find promising spans.** Rank repeated or highly predictable phrases using model scores and lightweight text heuristics. Prediction ranks candidates; it does not prove they are recoverable.
3. **Represent a gap.** Replace a candidate span with an unambiguous marker or structured gap record. Include any boundary, length, escaping, or protocol data the decoder needs.
4. **Reconstruct with shared rules.** The sender and receiver must agree on the decoder, model/profile, protocol version, and decoding settings. A causal language model does not inherently fill arbitrary gaps, so the gap-filling procedure must be specified.
5. **Verify exactly.** Compare reconstructed bytes with the original bytes. Reject any candidate that differs, even by one byte. A digest can detect a mismatch, but cannot recover missing text.
6. **Compare complete sizes.** Include markers, metadata, model-related side information, headers, and integrity data. Compare the complete CAIE packet after ordinary compression (for example, zstd) with the same compressor applied to the original.
7. **Fall back when needed.** Send the ordinary compressed original whenever CAIE fails exact reconstruction or does not produce a smaller valid packet.

## Example

Original:

```text
The quick brown fox jumps over the lazy dog.
```

Candidate representation:

```text
The quick brown fox <GAP> the lazy dog.
```

The gap may be omitted only if the agreed decoder reconstructs `jumps over` with exactly the original spacing and bytes. A plausible alternative is not sufficient.

## Key limitations

- Language models may produce plausible text rather than the exact original.
- Sender and receiver need compatible reconstruction resources and deterministic rules.
- Candidate search can be expensive and requires practical bounds.
- Standard compressors may already encode predictable text efficiently; CAIE will not help every input.
- Shared model knowledge and any required side information must be accounted for fairly.
- CAIE is a proposal, not a demonstrated universal improvement or an originality claim.

The governing principle is: **predict to choose what to try, verify exact bytes to decide what is valid, and count every required byte before claiming compression.**
