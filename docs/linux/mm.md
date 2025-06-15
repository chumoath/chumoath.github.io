# mm

### linux v2.4

```c
// mm/page_alloc.c

// 使用 zone的bitmap，zone->free_area[i].map

MARK_USED change_bit((index) >> (1+(order)), (area)->map)

rmqueue
	MARK_USED
	
	expand
		MARK_USED	
		
// bootmem 是所有低端内存管理数据结构。

// 内存管理数据结构：
pg_data_t contig_page_data;

// *gmap 为 mem_map
// 当前 pg_data_t 的 struct page *[]
*gmap = pgdat->node_mem_map = lmem_map;

// 当前 pg_data_t 管理的page个数
pgdat->node_size = totalpages;

// 当前 pg_data_t 物理内存的起始地址 (0)
pgdat->node_start_paddr = zone_start_paddr;

// mem_map 同 lmem_map，为 0
pgdat->node_start_mapnr = (lmem_map - mem_map);

// 当前 pg_data_t 的 zone 数据结构
pgdat->node_zones

pgdat->node_zonelists

// 每个zone的size 和 start 已经在 paging_init 中指定了
free_area_init_core
	
	// 最简单的情况： lmem_map == mem_map, offset初始化为 0
	offset = lmem_map - mem_map;	
	
	for (j = 0; j < MAX_NR_ZONES; j++) {
		
		// zone->offset: 当前 zone 起始物理地址
		zone->offset = offset;
		
		// zone_mem_map 为 当前zone管理内存的起始 struct page *
		zone->zone_mem_map = mem_map + offset;
		zone->zone_start_mapnr = offset;
		
		// zone_start_paddr: 当前zone管理内存的起始物理地址； 被 free_area_init_core 传进来，为 0
		zone->zone_start_paddr = zone_start_paddr;
		
		// 初始化当前 area 的 每一个 page
		for (i = 0; i < size; i++) {
			struct page *page = mem_map + offset + i;
			page->zone = zone;
			if (j != ZONE_HIGHMEM) {
				// 除了 hightmem，low内存已经全部被映射
				page->virtual = __va(zone_start_paddr);
				// 指向下一个 page 的物理地址
				zone_start_paddr += PAGE_SIZE;
			}
		}
		
		// 为下一个 zone 做准备
		offset += size;
		
		mask = -1;
		// 将当前zone管理的内存分为 10类，分别为 2^0, 2^1, ... ,2^9
		for (i = 0; i < MAX_ORDER; i++) {
			unsigned long bitmap_size;

			memlist_init(&zone->free_area[i].free_list);
			// -1, -2, -4, -8, -16, -32, ...
			mask += mask;
			//     i:      0            1            2             3              4              5
			//  mask:      -1          -2            -4           -8             -16            -32
			// ~mask: 0x00000000   0x00000001   0x00000003    0x00000007      0x0000000F     0x0000001F                         
			//  mask: 0xFFFFFFFF   0xFFFFFFFE   0xFFFFFFFC    0xFFFFFFF8      0xFFFFFFF0     0xFFFFFFE0
			//        1的倍数        2的倍数          4的倍数        8的倍数           16的倍数       32的倍数
			// 计算当前zone管理的内存，1个连续page、2个连续pages、4个连续pages、...、10个连续pages 的数量，并分别建立 bitmap
			// size 为当前zone管理的内存的page数
			
			size = (size + ~mask) & mask; // 对 0/2/4/8/16/32/... 向上取整
			
			// 当前zone拥有的连续 2^i 的 pages 的个数
			bitmap_size = size >> i;      // 除以 1, 2, 4, 8, 16, 32, ...
			
			// 对8向上取整，方便建立 bitmap
			bitmap_size = (bitmap_size + 7) >> 3; // 除以8向上取整
			bitmap_size = LONG_ALIGN(bitmap_size);
			
			// 给每个 order 分配 bitmap
			zone->free_area[i].map = (unsigned int *) alloc_bootmem_node(pgdat, bitmap_size);
		}
		
		build_zonelists(pgdat);
		
// 从低端内存的DMA区域之后分配内存，也就是 ZONE_NORMAL
alloc_bootmem_node

typedef struct zonelist_struct {
	zone_t * zones [MAX_NR_ZONES+1]; // NULL delimited
	int gfp_mask;
} zonelist_t;

static inline void build_zonelists(pg_data_t *pgdat)
{
	int i, j, k;
	
	// pgdat->node_zonelists[NR_GFPINDEX]   256 = 0x100
	for (i = 0; i < NR_GFPINDEX; i++) {
		zonelist_t *zonelist;
		zone_t *zone;

		zonelist = pgdat->node_zonelists + i;
		memset(zonelist, 0, sizeof(*zonelist));

		zonelist->gfp_mask = i;
		j = 0;
		k = ZONE_NORMAL;
		if (i & __GFP_HIGHMEM)
			k = ZONE_HIGHMEM;
		if (i & __GFP_DMA)
			k = ZONE_DMA;

		switch (k) {
			default:
				BUG();
			/*
			 * fallthrough:
			 */
			case ZONE_HIGHMEM:
				zone = pgdat->node_zones + ZONE_HIGHMEM;
				if (zone->size) {
#ifndef CONFIG_HIGHMEM
					BUG();
#endif
					zonelist->zones[j++] = zone;
				}
			case ZONE_NORMAL:
				zone = pgdat->node_zones + ZONE_NORMAL;
				if (zone->size)
					zonelist->zones[j++] = zone;
			case ZONE_DMA:
				zone = pgdat->node_zones + ZONE_DMA;
				if (zone->size)
					zonelist->zones[j++] = zone;
		}
		zonelist->zones[j++] = NULL;
	} 
}

// 没有HIGHMEM
每 8 个配置为一个类型的 zone。

i & 0x08 == true   -> DMA;
else NORMAL;

0x00-0x07 NORMAL; 0x08-0x0f DMA; 
0x10-0x17 NORMAL； 0x18-0x1f DMA; 
0x20-0x27 NORMAL; 0x28-0x2f DMA; 
0x30-0x37 NORMAL; 0x38-0x3f DMA; 
0x40-0x47 NORMAL; 0x48-0x4f DMA; 
0x50-0x57 NORMAL; 0x58-0x5f DMA; 
0x60-0x67 NORMAL; 0x68-0x6f DMA; 
0x70-0x77 NORMAL; 0x78-0x7f DMA;

// 有了 HIGHMEM； 
i & 0x10 == true && i & 0x08 == false  ->  HIGHMEM;  //  i & 0x10 == true 每 32 个 16个HIGHMEM;  i & 0x08 == false 16个HIGHMEM中 8个变为 DMA
i & 0x08 == true  ->  DMA;      每 16 个 8个 DMA
else NORMAL;

0x00-0x07 NORMAL; 0x08-0x0f DMA; 0x10-0x17 HIGHMEM； 0x18-0x1f DMA; 
0x20-0x27 NORMAL; 0x28-0x2f DMA; 0x30-0x37 HIGHMEM; 0x38-0x3f DMA; 
0x40-0x47 NORMAL; 0x48-0x4f DMA; 0x50-0x57 HIGHMEM; 0x58-0x5f DMA; 
0x60-0x67 NORMAL; 0x68-0x6f DMA; 0x70-0x77 HIGHMEM; 0x78-0x7f DMA;

alloc_pages => __alloc_pages(contig_page_data.node_zonelists+(gfp_mask), order);
order为0，表示分配 1page；为1，表示分配 2 pages；为3，表示分配 4 pages，... ，为9，表示分配 512 pages

zone->free_area[order].map 就是每个zone的每个order的bitmap
```

- free_area_init_core 的调用

  - 假设有 highmem(4G/PAE 64位地址)

  ```c
  // arch/i386/kernel/setup.c
  setup_arch
      // 从 empty_zero_page 的 前半页 获取 BIOS 传递的E820的物理地址范围
      // empty_zero_page (0x4000): 前半页 (0x4000-0x47FF) => BIOS传递信息；后半页 (0x4800-0x4FFF) => 内核启动参数
      setup_memory_region
      	// 将物理地址范围添加到 struct e820map e820; 全局变量
      	add_memory_region
      // 获取内核结尾物理地址的下一页
      start_pfn = PFN_UP(__pa(&_end));
  	// e820 最大物理页
  	max_pfn;
  	// #define VMALLOC_RESERVE	(unsigned long)(128 << 20)
      // #define MAXMEM		(unsigned long)(-PAGE_OFFSET-VMALLOC_RESERVE)
      // #define MAXMEM_PFN	PFN_DOWN(MAXMEM)
  	// MAXMEM_PFN = 0x38000000 >> 12 = 0x38000
  	max_low_pfn = MAXMEM_PFN;   // lowmem:  0x00000000-0x38000000 物理地址
  	highend_pfn = max_pfn;
  	highstart_pfn = MAXMEM_PFN; // highmem: 0x38000000-max_pfn
  	
  	// 先将 0x00000000-0x38000000的物理内存全部置为 reserved，表示已经被使用；再使用e820实际的物理地址范围free，确保只有e820指定的地址可以被分配
  	bootmap_size = init_bootmem(start_pfn, max_low_pfn);
  		// (unsigned long start, unsigned long pages)
  		init_bootmem_core(&contig_page_data, start, 0, pages)
              // （unsigned long mapstart, unsigned long start, unsigned long end)
              bootmem_data_t *bdata = pgdat->bdata;
  			// 计算 bitmap 大小，向上对 8 取整
  			unsigned long mapsize = ((end - start)+7)/8;
  			pgdat->node_next = pgdat_list;
  			pgdat_list = pgdat;
  			mapsize = (mapsize + (sizeof(long) - 1UL)) & ~(sizeof(long) - 1UL);
  			// 设置 contig_page_data.bdata.node_bootmem_map为 start_pfn，即内核结尾的下一页的虚拟地址；这是第一块可用的内存地址，用来存放低端内存的 bitmap
  			bdata->node_bootmem_map = phys_to_virt(mapstart << PAGE_SHIFT);
  			// 设置 contig_page_data.bdata.node_boot_start 为 0，表示管理内存的起始物理地址
  			bdata->node_boot_start = (start << PAGE_SHIFT);
  			// 表示管理的内存的 pages
  			bdata->node_low_pfn = end;
  			// 将低端内存bitmap全部置 1，表示 reserved
  			memset(bdata->node_bootmem_map, 0xff, mapsize);
  			return mapsize;
  	// 将 e820 的所有的低端物理内存全部 free，置 0
  	for (i = 0; i < e820.nr_map; i++) {
          if (e820.map[i].type != E820_RAM)
  			continue;
          
          curr_pfn = PFN_UP(e820.map[i].addr);
  		if (curr_pfn >= max_low_pfn)
  			continue;
          free_bootmem(PFN_PHYS(curr_pfn), PFN_PHYS(size));
          	test_and_clear_bit(i, bdata->node_bootmem_map)
      // #define HIGH_MEMORY	(1024*1024)
  	// 将 0x00100000-(内核结束地址的下一页+低端内存的bitmap结束) 置为已使用状态；bitmap使用的页向上取整
  	reserve_bootmem(HIGH_MEMORY, (PFN_PHYS(start_pfn) + bootmap_size + PAGE_SIZE-1) - (HIGH_MEMORY));
          reserve_bootmem_core
              eidx = (addr + size - bdata->node_boot_start + PAGE_SIZE-1)/PAGE_SIZE;
          	test_and_set_bit(i, bdata->node_bootmem_map)
  	// 保留物理页 0 (0x00000000-0x00001000)
  	reserve_bootmem(0, PAGE_SIZE);
      // 将虚拟地址 0xc0000000-0xF8000000 映射到 物理地址 0x00000000-0x38000000
      // 通过 页目录表、页表索引获取虚拟地址，通过 虚拟地址 - PAGE_OFFSET (0xc0000000) 得到物理地址
      paging_init
          pagetable_init
          	end = (unsigned long)__va(max_low_pfn*PAGE_SIZE);
          	pgd_base = swapper_pg_dir;
          	i = __pgd_offset(PAGE_OFFSET);
          	pgd = pgd_base + i;
          	for (; i < PTRS_PER_PGD; pgd++, i++) {
                  for (j = 0; j < PTRS_PER_PMD; pmd++, j++) {
                      for (k = 0; k < PTRS_PER_PTE; pte++, k++) {
                          vaddr = i*PGDIR_SIZE + j*PMD_SIZE + k*PAGE_SIZE;
                          *pte = mk_pte_phys(__pa(vaddr), PAGE_KERNEL);
  		// 重新加载内核的页目录表
          __asm__( "movl %%ecx,%%cr3\n" ::"c"(__pa(swapper_pg_dir)));
          __flush_tlb_all();
          unsigned long zones_size[MAX_NR_ZONES] = {0, 0, 0};
          // #define MAX_DMA_ADDRESS      (PAGE_OFFSET+0x1000000)
  		// MAX_DMA_ADDRESS = 0xC1000000
  		// max_dma = 0x1000 个 pages，大小为 0x1000000
          max_dma = virt_to_phys((char *)MAX_DMA_ADDRESS) >> PAGE_SHIFT;
          low = max_low_pfn;
          high = highend_pfn;
  		// ZONE_DMA 设置为 0x1000
          zones_size[ZONE_DMA] = max_dma;
          // ZONE_NORMAL 设置为 剩下的低端内存
          zones_size[ZONE_NORMAL] = low - max_dma;
  		// ZONE_HIGHMEM 设置为高端内存
          zones_size[ZONE_HIGHMEM] = high - low;
