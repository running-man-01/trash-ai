<template>
    <Busy v-if="!loaded" />
    <GMapMap
        v-else
        :center="center"
        :zoom="zoom"
        map-type-id="hybrid"
        style="width: 100%; height: 75vh"
    >
        <GMapCluster
            v-for="(obj, idx) in items"
            :key="'dobj' + idx"
        >
            <GMapMarker
                :position="obj.gps"
                :clickable="true"
                :draggable="false"
                @click="openMarker(obj.hash)"
            >
                <GMapInfoWindow
                    :closeclick="true"
                    @closeclick="openMarker(null)"
                    :opened="openedMarkerID === obj.hash"
                >
                    <div>
                        <Thumb :item="obj" />
                    </div>
                </GMapInfoWindow>
            </GMapMarker>
        </GMapCluster>
    </GMapMap>
</template>
<script lang="ts">
import * as m from '@/lib'

interface GmapState {
    summary: m.Summary | null
    center: m.Coordinate | null
    position: m.Coordinate | null
    items: m.Display[]
    zoom: number
    loaded: boolean
    openedMarkerID: string | null
}
import { defineComponent } from 'vue'

export default defineComponent({
    name: 'GMapSummary',
    data(): GmapState {
        return {
            summary: null,
            position: null,
            center: null,
            zoom: 15,
            loaded: false,
            openedMarkerID: null,
            items: [],
        }
    },
    setup() {
        const imgstore = m.useImageStore()
        return {
            imgstore,
        }
    },
    methods: {
        openMarker(id: string | null) {
            this.openedMarkerID = id
        },
    },
    async mounted() {
        this.summary = this.imgstore.summary
        let dlist = await m.imagedb.savedata.bulkGet(
            this.summary.gps.list.map((g) => g.hash),
        )
        dlist.forEach((d) => {
            this.items.push(d!.display)
        })
        this.center = this.summary?.gps.center
        this.position = this.summary?.gps.center
        this.loaded = true
    },
})
</script>

<style>
body {
    margin: 0;
}
</style>
