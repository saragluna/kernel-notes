```c
<linux/page-flags.h>

/*
 * Various page->flags bits:
 *
 * PG_reserved is set for special pages, which can never be swapped out. Some
 * of them might not even exist (eg empty_bad_page)...
 *
 * The PG_private bitflag is set on pagecache pages if they contain filesystem
 * specific data (which is normally at page->private). It can be used by
 * private allocations for its own usage.
 *
 * During initiation of disk I/O, PG_locked is set. This bit is set before I/O
 * and cleared when writeback _starts_ or when read _completes_. PG_writeback
 * is set before writeback starts and cleared when it finishes.
 *
 * PG_locked also pins a page in pagecache, and blocks truncation of the file
 * while it is held.
 *
 * page_waitqueue(page) is a wait queue of all tasks waiting for the page
 * to become unlocked.
 *
 * PG_uptodate tells whether the page's contents is valid.  When a read
 * completes, the page becomes uptodate, unless a disk I/O error happened.
 *
 * PG_referenced, PG_reclaim are used for page reclaim for anonymous and
 * file-backed pagecache (see mm/vmscan.c).
 *
 * PG_error is set to indicate that an I/O error occurred on this page.
 *
 * PG_arch_1 is an architecture specific page state bit.  The generic code
 * guarantees that this bit is cleared for a page when it first is entered into
 * the page cache.
 *
 * PG_highmem pages are not permanently mapped into the kernel virtual address
 * space, they need to be kmapped separately for doing IO on the pages.  The
 * struct page (these bits with information) are always mapped into kernel
 * address space...
 *
 * PG_hwpoison indicates that a page got corrupted in hardware and contains
 * data with incorrect ECC bits that triggered a machine check. Accessing is
 * not safe since it may cause another machine check. Don't touch!
 */
 
 /*
 * Don't use the *_dontuse flags.  Use the macros.  Otherwise you'll break
 * locked- and dirty-page accounting.
 *
 * The page flags field is split into two parts, the main flags area
 * which extends from the low bits upwards, and the fields area which
 * extends from the high bits downwards.
 *
 *  | FIELD | ... | FLAGS |
 *  N-1           ^       0
 *               (NR_PAGEFLAGS)
 *
 * The fields area is reserved for fields mapping zone, node (for NUMA) and
 * SPARSEMEM section (for variants of SPARSEMEM that require section ids like
 * SPARSEMEM_EXTREME with !SPARSEMEM_VMEMMAP).
 */
enum pageflags {
    PG_locked,      /* Page is locked. Don't touch. */
    PG_error,
    PG_referenced,
    PG_uptodate,
    PG_dirty,
    PG_lru,
    PG_active,
    PG_waiters,     /* Page has waiters, check its waitqueue. Must be bit #7 and in the same byte as "PG_locked" */
    PG_slab,
    PG_owner_priv_1,    /* Owner use. If pagecache, fs may use*/
    PG_arch_1,
    PG_reserved,
    PG_private,     /* If pagecache, has fs-private data */
    PG_private_2,       /* If pagecache, has fs aux data */
    PG_writeback,       /* Page is under writeback */
    PG_head,        /* A head page */
    PG_mappedtodisk,    /* Has blocks allocated on-disk */
    PG_reclaim,     /* To be reclaimed asap */
    PG_swapbacked,      /* Page is backed by RAM/swap */
    PG_unevictable,     /* Page is "unevictable"  */
#ifdef CONFIG_MMU
    PG_mlocked,     /* Page is vma mlocked */
#endif
#ifdef CONFIG_ARCH_USES_PG_UNCACHED
    PG_uncached,        /* Page has been mapped as uncached */
#endif
#ifdef CONFIG_MEMORY_FAILURE
    PG_hwpoison,        /* hardware poisoned page. Don't touch */
#endif
#if defined(CONFIG_IDLE_PAGE_TRACKING) && defined(CONFIG_64BIT)
    PG_young,
    PG_idle,
#endif
    __NR_PAGEFLAGS,

    /* Filesystems */
    PG_checked = PG_owner_priv_1,

    /* SwapBacked */
    PG_swapcache = PG_owner_priv_1, /* Swap page: swp_entry_t in private */
     
    /* Two page bits are conscripted by FS-Cache to maintain local caching
     * state.  These bits are set on pages belonging to the netfs's inodes
     * when those inodes are being locally cached.
     */
    PG_fscache = PG_private_2,  /* page backed by cache */

    /* XEN */
    /* Pinned in Xen as a read-only pagetable page. */
    PG_pinned = PG_owner_priv_1,
    /* Pinned as part of domain save (see xen_mm_pin_all()). */
    PG_savepinned = PG_dirty,
    /* Has a grant mapping of another (foreign) domain's page. */
    PG_foreign = PG_owner_priv_1,

    /* SLOB */
    PG_slob_free = PG_private,

    /* Compound pages. Stored in first tail page's flags */
    PG_double_map = PG_private_2,

    /* non-lru isolated movable page */
    PG_isolated = PG_reclaim,
};
```

**more notes**:

- `PG_referenced` and `PG_active` control how actively a page is used by the system. This information is important when the swapping subsystem has to select which page to swap out.
- `PG_uptodate` indicates that the data of a page have been read without error from a block device.
- `PG_dirty` is set when the contents of the page have changed as compared to the data on hard disk.
- `PG_lru` helps implement page reclaim and swapping. The kernel uses two least recently used lists to distinguish between active and inactive pages. The bit is set if the page is held on one of these lists. 
- `PG_private` must be set if the value of the `private` element in the `page`structure is non-`NULL`. Pages that are used for I/O use this field to subdivide the page into *buffers*, but other parts of the kernel find different uses to attach private data to a page.
- `PG_writeback` is set for pages whose contents are in the process of being written back to a block device.
- `PG_swapcache` is set if the page is in the swap cache; in this case, `private`contains an entry of type `swap_entry_t`