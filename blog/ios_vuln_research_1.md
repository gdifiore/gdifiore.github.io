# Finding iOS Vulnerabilities Through... Apple's Comments?

While exploring differences between XNU versions, I stumbled upon a patched memory management bug. Surprisingly, a detailed comment left by Apple provided the key to uncovering a subtle yet critical vulnerability impacting memory safety.

## Notes

If you want to explore yourself you need to:

Download:
- the sources for `xnu-6153.121.1` and `xnu-6153.141.1` (available at [opensource.apple.com](opensource.apple.com))
- a tool to diff directories/files

This article also assumes that the reader has knowledge of MacOS, mach, IPC, C code, etc.

## A Small Observation

Traversing through the XNU sources with `WinDiff` revealed that `/osfmk/ipc/ipc_kmsg.c` is not the same between versions, meaning that maybe a bug was patched, or maybe it was just a typo that an OCD programmer felt the need to correct.

However, the side view revealed a 52 line difference between files, so it can't be just a typo. But while reviewing the code I noticed a...

## Big Comment

Apple was kind enough to leave behind a comment explaining exactly what problem the code fixed!

The comment left in `ipc_kmsg_copyout_ool_descriptor` reveals that the issue stems from the mismanagement of offsets during deallocation. Apple’s note explains how a physical copy into a rounded range could leak memory if the size being deallocated doesn’t cover the full range.
```c
/*
 * vm_map_copy_overwrite does a full copy
 * if size is too small to optimize.
 * So we tried skipping the offset adjustment
 * if we fail the 'size' test.
 *
 * if (size >= VM_MAP_COPY_OVERWRITE_OPTIMIZATION_THRESHOLD_PAGES * ctive_page_size) {
 *
 * This resulted in leaked memory especially on the
 * older watches (16k user - 4k kernel) because we
 * would do a physical copy into the start of this
 * rounded range but could leak part of it
 * on deallocation if the 'size' being deallocated
 * does not cover the full range. So instead we do
 * the misalignment adjustment always so that on
 * deallocation we will remove the full range.
 */
```

Now let's take a look at the code to see what this is talking about.

## `size` Matters

In the previous version of XNU (`xnu-6153.121.1`), the (interesting part of the) function is:

```c
mach_msg_descriptor_t * ipc_kmsg_copyout_ool_descriptor(mach_msg_ool_descriptor_t *dsc, mach_msg_descriptor_t *user_dsc, int is_64bit, vm_map_t map, mach_msg_return_t *mr)
{
	vm_map_copy_t                       copy;
	vm_map_address_t                    rcv_addr;
	mach_msg_copy_options_t             copy_options;
	vm_map_size_t                       size;
	mach_msg_descriptor_type_t  dsc_type;

	//SKIP_PORT_DESCRIPTORS(saddr, sdsc_count);

	copy = (vm_map_copy_t)dsc->address;
	size = (vm_map_size_t)dsc->size;
	copy_options = dsc->copy;
	assert(copy_options != MACH_MSG_KALLOC_COPY_T);
	dsc_type = dsc->type;

	if (copy != VM_MAP_COPY_NULL) {
		kern_return_t kr;

		rcv_addr = 0;
		if (vm_map_copy_validate_size(map, copy, &size) == FALSE) {
			panic("Inconsistent OOL/copyout size on %p: expected %d, got %lld @%p",
			    dsc, dsc->size, (unsigned long long)copy->size, copy);
		}

		kr = vm_map_copyout_size(map, &rcv_addr, copy, size);
	// ...
```
The interesting part of the above code is the last few lines, so first let's take a look at what `vm_map_copy_validate_size()` does.

```c
/*
 * Returns true if *size matches (or is in the range of) copy->size.
 * Upon returning true, the *size field is updated with the actual size of the
 * copy object (may be different for VM_MAP_COPY_ENTRY_LIST types)
 */
boolean_t
vm_map_copy_validate_size(
	vm_map_t		dst_map,
	vm_map_copy_t		copy,
	vm_map_size_t		*size)
{
	if (copy == VM_MAP_COPY_NULL)
		return FALSE;
	vm_map_size_t copy_sz = copy->size;
	vm_map_size_t sz = *size;
	switch (copy->type) {
	case VM_MAP_COPY_OBJECT:
	case VM_MAP_COPY_KERNEL_BUFFER:
		if (sz == copy_sz)
			return TRUE;
		break;
	case VM_MAP_COPY_ENTRY_LIST:
		/*
		 * potential page-size rounding prevents us from exactly
		 * validating this flavor of vm_map_copy, but we can at least
		 * assert that it's within a range.
		 */
		if (copy_sz >= sz &&
		    copy_sz <= vm_map_round_page(sz, VM_MAP_PAGE_MASK(dst_map))) {
			*size = copy_sz;
			return TRUE;
		}
		break;
	default:
		break;
	}
	return FALSE;
}
```
`vm_map_copy_validate_size` checks that the `size` field aligns with the actual size of the `vm_map_copy_t` object. However, its behavior differs based on the type of copy:

- For `VM_MAP_COPY_OBJECT` and `VM_MAP_COPY_KERNEL_BUFFER`, it checks for exact matches.
- For `VM_MAP_COPY_ENTRY_LIST`, it allows ranges due to page-size rounding.

Page size mismatches (e.g., 16k user vs. 4k kernel) introduce misalignment risks. If the size field doesn't account for the full memory range during deallocation, leftover memory remains inaccessible but allocated, leading to leaks.

If size doesn't match the rounded range, the kernel might only partially deallocate memory, leading to leaks.

But there isn't anything inherently wrong with that, because `vm_map_copy_validate_size()` says that
```c
/*
 * potential page-size rounding prevents us from exactly
 * validating this flavor of vm_map_copy, but we can at least
 * assert that it's within a range.
 */
```
Anyways, let's continue to step through the code with `vm_map_copyout_size()`

```c
/*
 *	Routine:	vm_map_copyout_size
 *
 *	Description:
 *		Copy out a copy chain ("copy") into newly-allocated
 *		space in the destination map. Uses a prevalidated
 *		size for the copy object (vm_map_copy_validate_size).
 *
 *		If successful, consumes the copy object.
 *		Otherwise, the caller is responsible for it.
 */
kern_return_t
vm_map_copyout_size(
	vm_map_t		dst_map,
	vm_map_address_t	*dst_addr,	/* OUT */
	vm_map_copy_t		copy,
	vm_map_size_t		copy_size)
{
	return vm_map_copyout_internal(dst_map, dst_addr, copy, copy_size,
	                               TRUE, /* consume_on_success */
	                               VM_PROT_DEFAULT,
	                               VM_PROT_ALL,
	                               VM_INHERIT_DEFAULT);
}
```
So it's supposed to copy `copy` into a new area of memory (`dst_map`) of address `*dst_addr` with size `copy_size`. But what's the problem with that?

`ipc_kmsg_copyout_ool_descriptor()` doesn't implement a proper check if `copy_size` covers the entire range of memory needed (beyond `vm_map_copy_validate_size()`).

If the offset for `size` is not adjusted to cover the entire range of memory, when a physical copy happens at the start of the *rounded* (not true) range, and the `size` passed doesn't cover the full range, it can leak memory when deallocated.

To fix this, XNU (`xnu-6153.141.1`) implemented a check if `size` is correct:
```c
// ...
    boolean_t                           misaligned = FALSE;
// ...

		if ((copy->type == VM_MAP_COPY_ENTRY_LIST) &&
		    (trunc_page(copy->offset) != copy->offset ||
		    round_page(dsc->size) != dsc->size)) {
			misaligned = TRUE;
		}

		if (misaligned) {
			vm_map_address_t        rounded_addr;
			vm_map_size_t   rounded_size;
			vm_map_offset_t effective_page_mask, effective_page_size;

			effective_page_mask = VM_MAP_PAGE_MASK(map);
			effective_page_size = effective_page_mask + 1;

			rounded_size = vm_map_round_page(copy->offset + size, effective_page_mask) - vm_map_trunc_page(copy->offset, effective_page_mask);

			kr = vm_allocate_kernel(map, (vm_offset_t*)&rounded_addr, rounded_size, VM_FLAGS_ANYWHERE, 0);

			if (kr == KERN_SUCCESS) {
				/*
				 * vm_map_copy_overwrite does a full copy
				 * if size is too small to optimize.
				 * So we tried skipping the offset adjustment
				 * if we fail the 'size' test.
				 *
				 * if (size >= VM_MAP_COPY_OVERWRITE_OPTIMIZATION_THRESHOLD_PAGES * effective_page_size) {
				 *
				 * This resulted in leaked memory especially on the
				 * older watches (16k user - 4k kernel) because we
				 * would do a physical copy into the start of this
				 * rounded range but could leak part of it
				 * on deallocation if the 'size' being deallocated
				 * does not cover the full range. So instead we do
				 * the misalignment adjustment always so that on
				 * deallocation we will remove the full range.
				 */
				if ((rounded_addr & effective_page_mask) !=
				    (copy->offset & effective_page_mask)) {
					/*
					 * Need similar mis-alignment of source and destination...
					 */
					rounded_addr += (copy->offset & effective_page_mask);

					assert((rounded_addr & effective_page_mask) == (copy->offset & effective_page_mask));
				}
				rcv_addr = rounded_addr;

				kr = vm_map_copy_overwrite(map, rcv_addr, copy, size, FALSE);
			}
		} else {
			kr = vm_map_copyout_size(map, &rcv_addr, copy, size);
		}
	// ...
```

The specific fix is contained in one line:
```c
	(rounded_addr & effective_page_mask) == (copy->offset & effective_page_mask)
```
By ensuring the size field is adjusted for alignment issues, they prevent partial deallocation.

## Exploitation

The comment specifically references older Apple Watches as vulnerable to this (due to the mismatch of user & kernel page sizes). This is an issue, for example, large user pages split into smaller kernel pages increase the likelihood of misalignment.

One potential technique may be using mach messages.

Vulnerability Trigger:
- An attacker crafts a mach message with a mismatched size field in the out-of-line (OOL) descriptor.
- This message is passed to the kernel for processing. If the misalignment adjustment isn't applied, the kernel fails to deallocate the full range.

Exploitation Goals:
- Denial-of-Service: By repeatedly triggering the leak, attackers could exhaust kernel memory, leading to a system crash.
- Information Disclosure (potential): Depending on what memory is leaked, it might reveal sensitive data.

## Conclusion

Was this comment intentional?

We may never know, but it gave insight into the work of an Apple employee, and may be a first step for someone wanting to learn vulnerability research and exploitation.

### References
[(MIT) mach_msg_descriptor](http://web.mit.edu/darwin/src/modules/xnu/osfmk/man/mach_msg_descriptor.html)

[(FreeBSD) COPY](https://www.freebsd.org/cgi/man.cgi?query=copyout&apropos=0&sektion=9&manpath=FreeBSD+11-current&format=html)

[(Jonathan Levin) XXR - The XNU Xref](http://newosxbook.com/xxr/index.jl)

<hr>
<b><center>Me on: <a href="https://github.com/gdifiore/">GitHub</a>