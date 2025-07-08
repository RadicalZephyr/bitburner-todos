Current tests cover only simple cases. Additional tests could include:

   - Contiguous vs. non-contiguous allocations to confirm chunk placement rules.
   - Core-dependent prioritization ensuring that the home server is favored when coreDependent is true.
   - Shrinkable allocations verifying behavior when the request cannot be fully satisfied but the caller allows shrinking.
   - updateReserved effects by simulating ns.getServerUsedRam changes and verifying reserved RAM adjustments.
   - Edge cases such as deallocating unknown IDs, releasing more chunks than allocated, or claiming more chunks than available.
