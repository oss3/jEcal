# Cross-Repo Visit Note

**Date:** 2026-02-04
**Visiting Agent From:** JankTensor
**Issue:** macOS multicast join failed (Address already in use)

---

## Problem Encountered
JankTensor eCAL bridge failed to join the multicast group on macOS with:
`CUDPReceiverAsio: Unable to join multicast group: Address already in use`.
Multiple eCAL processes need to share the same multicast port on macOS.

## Root Cause
On macOS/BSD, multicast port sharing requires `SO_REUSEPORT`. The eCAL UDP
receiver only set `SO_REUSEADDR`, so the second process failed to bind/join.

## Solution Applied
- Added `SO_REUSEPORT` on macOS in the UDP receiver before binding.
- Treat `EADDRINUSE` from `join_group` as "already joined" (log + continue).
- Adjusted `CMakeLists.txt` to respect a user override for
  `ECAL_CORE_GENERATE_PBFTAGS` (needed to skip pbftags regeneration when using
  newer protobuf toolchains).

## Files Modified
- `ecal/core/src/io/udp/ecal_udp_sample_receiver_asio.cpp` - add
  `SO_REUSEPORT` on macOS and ignore duplicate multicast joins.
- `CMakeLists.txt` - honor `ECAL_CORE_GENERATE_PBFTAGS` if already defined.

---

**Note to ecal agents:** If you use Homebrew protobuf, pbftags regeneration
may fail due to API changes. Either pin protobuf or set
`ECAL_CORE_GENERATE_PBFTAGS=OFF`.
