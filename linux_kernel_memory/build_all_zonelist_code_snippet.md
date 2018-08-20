```c
<mm/page_alloc.c>
static int __build_all_zonelists(void *data)
{
	int nid;
    ...
	for_each_online_node(nid) {
        pg_data_t *pgdat = NODE_DATA(nid);
        build_zonelists(pgdat);
    }
    ...
    return 0;
}

static void build_zonelists(pg_data_t *pgdat)
{
    int node, local_node;
    enum zone_type j;
    struct zonelist *zonelist;

    local_node = pgdat->node_id;

    zonelist = &pgdat->node_zonelists[ZONELIST_FALLBACK];
    j = build_zonelists_node(pgdat, zonelist, 0);

    /*
     * Now we build the zonelist so that it contains the zones
     * of all the other nodes.
     * We don't want to pressure a particular node, so when
     * building the zones for node N, we make sure that the
     * zones coming right after the local ones are those from
     * node N+1 (modulo N)
     */
    for (node = local_node + 1; node < MAX_NUMNODES; node++) {
        if (!node_online(node))
            continue;
        j = build_zonelists_node(NODE_DATA(node), zonelist, j);
    }
    for (node = 0; node < local_node; node++) {
        if (!node_online(node))
            continue;
        j = build_zonelists_node(NODE_DATA(node), zonelist, j);
    }

    zonelist->_zonerefs[j].zone = NULL;
    zonelist->_zonerefs[j].zone_idx = 0;
}

/*
 * Builds allocation fallback zone lists.
 *
 * Add all populated zones of a node to the zonelist.
 */
static int build_zonelists_node(pg_data_t *pgdat, struct zonelist *zonelist, int nr_zones)
{
    struct zone *zone;
    enum zone_type zone_type = MAX_NR_ZONES;

    // zonelist->zones[0] = ZONE_HIGHMEM;
	// zonelist->zones[1] = ZONE_NORMAL;
    // zonelist->zones[2] = ZONE_DMA;
    do {
        zone_type--;
        zone = pgdat->node_zones + zone_type;
        if (managed_zone(zone)) {
            zoneref_set_zone(zone, &zonelist->_zonerefs[nr_zones++]);
            check_highest_zone(zone_type);
        }
    } while (zone_type);

    return nr_zones;
}
```

