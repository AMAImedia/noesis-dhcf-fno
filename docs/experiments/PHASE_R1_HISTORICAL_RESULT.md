# NOESIS — Deterministic Hybrid Control Framework for Frozen Neural Operators (DHCF-FNO)

# Phase R.1 — Historical Result

## Benchmark Summary

| Phase | PASS | PASS_CF_LIMITED | IQS Mean | Max LUFS Drift |
|------|------|----------------|---------|---------------|
| R.0 | 0 | 10 | 0.4947 | 1.609 dB |
| R.1 | 10 | 0 | 0.7027 | 0.0078 dB |

## Improvements

Drift reduction:
1.609 dB → 0.0078 dB  
Reduction factor: **206×**

IQS improvement:
0.4947 → 0.7027  
Increase: **+42%**

All tracks moved from PASS_CF_LIMITED → PASS.

## Key Case

metal genre:

before:
drift = 1.609 dB

after:
drift = 0.0055 dB

This confirms the expected behavior of **Stage 8.5 v4** limiter architecture.

## Conclusion

Stage 8.5 v4 successfully stabilizes LUFS drift and improves perceptual quality across all tested genres.

The system now satisfies deterministic mastering constraints required by the NOESIS protocol.
