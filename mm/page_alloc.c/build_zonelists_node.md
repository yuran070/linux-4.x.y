build_zonelists_node
========================================

备用列表的各项是借助于zone_type参数排序的，该参数指定了最优先选择哪个内存域，
我们知道其值可能是ZONE_HIGHMEM、ZONE_NORMAL、ZONE_DMA或ZONE_DMA32之一。
nr_zones表示从备用列表中的哪个位置开始填充新项。由于列表中尙没有项，因此调用者传递了0.

path: mm/page_alloc.c
```
/*
 * Builds allocation fallback zone lists.
 *
 * Add all populated zones of a node to the zonelist.
 */
static int build_zonelists_node(pg_data_t *pgdat, struct zonelist *zonelist,
                int nr_zones)
{
    struct zone *zone;
    enum zone_type zone_type = MAX_NR_ZONES;

    /* 在每一步结束时，都将内存域类型减1。换句话说，设置为一个更昂贵的内存域类型。
     * 例如，如果开始的内存域是ZONE_HIGHMEM，减1后下一个内存域类型是ZONE_NORMAL。
     * 考虑一个系统，有内存域ZONE_HIGHMEM、ZONE_NORMAL、ZONE_DMA
     */
    do {
        zone_type--;
        zone = pgdat->node_zones + zone_type;
        /* 在build_zonelists_zone的每一步中，都对所选的内存域调用populated_zone，
         * 确认zone->present_pages大于0，即确认内存域中确实有页存在。倘若如此，
         * 则将指向zone实例的指针添加到zonelist->zones中的当前位置
         */
        if (populated_zone(zone)) {
            // 从pgdat节点上的廉价内存开始全部存放到zonelist->_zonerefs数组中
            zoneref_set_zone(zone, &zonelist->_zonerefs[nr_zones++]);
            check_highest_zone(zone_type);
        }
    } while (zone_type);

    // 后备列表的当前位置保存在nr_zone
    return nr_zones;
}
```

### enum zone_type

https://github.com/novelinux/linux-4.x.y/tree/master/include/linux/mmzone.h/enum_zone_type.md

### populated_zone

https://github.com/novelinux/linux-4.x.y/tree/master/include/linux/mmzone.h/populated_zone.md

### zoneref_set_zone

https://github.com/novelinux/linux-4.x.y/tree/master/mm/page_alloc.c/zoneref_set_zone.md
