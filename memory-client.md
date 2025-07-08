* In `src/services/client/memory.ts`, create a new `Allocation` base
  class encapsulating `allocationId` and the host allocation list.
* Change `TransferableAllocation` to be a subclass of `Allocation`.
* In `src/services/client/memory.ts`, create a new `OwnedAllocation`
  subclass of `Allocation`.
* Provide methods to access chunk information and release the allocation, mirroring `TransferableAllocation`.
* Update `MemoryClient.requestOwnedAllocation` to return `OwnedAllocation`.
* Add JSDoc comments for the new class and updated methods.
