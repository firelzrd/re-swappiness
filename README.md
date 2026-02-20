# Re-swappiness

**A patch that fixes MGLRU's broken `vm.swappiness` behavior**

## The Problem

In the vanilla kernel, MGLRU (Multi-Gen LRU) shares a single generation counter (`max_seq`) across both anonymous and file page types. This design confines `swappiness` to a per-generation local scope rather than functioning as a global anon/file reclaim ratio control. As a result, `vm.swappiness` is effectively ignored under MGLRU — for example, setting `vm.swappiness=0` does not reliably prevent swapping.

## What Re-swappiness Does

Re-swappiness resolves this by:
- **Splitting `max_seq` into per-type (`max_seq[ANON_AND_FILE]`)**, allowing anonymous and file pages to age independently
- **Filtering VMA walks by type**, so anon and file aging proceed on separate schedules
- **Removing the cross-type `min_seq` synchronization constraint**, which previously blocked eviction progress
- **Replacing the PID controller in `get_type_to_scan()` with a proportional selector** that enforces the target anon:file scan ratio derived from `swappiness`

With this patch, `vm.swappiness` regains its intended meaning as a global ratio controller under MGLRU, consistent with Classic LRU behavior.

## Applying the Patch

```
cd /path/to/linux-source
patch -p1 < 0001-Re-swappiness-v1.0.patch
```

## Related Projects

- [le9uo](https://github.com/firelzrd/le9uo) — Working Set Protection for the Linux kernel. Combines with Re-swappiness for full working set protection with proper `swappiness` control on MGLRU-enabled kernels.

## License

GPL-2.0 — same as the Linux kernel.