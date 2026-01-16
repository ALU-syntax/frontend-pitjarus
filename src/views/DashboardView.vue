<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'
import api from '../api'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import {
    Select,
    SelectContent,
    SelectItem,
    SelectTrigger,
    SelectValue
} from '@/components/ui/select'
import { Calendar } from '@/components/ui/calendar'
import {
    Popover,
    PopoverContent,
    PopoverTrigger
} from '@/components/ui/popover'

import { CalendarDate, fromDate, getLocalTimeZone, today } from '@internationalized/date'
import type { DateValue } from '@internationalized/date'
import { Label } from '@/components/ui/label'
import { toast } from 'vue-sonner'
import BarChart from "@/components/BarChart.vue"
import {
    Table,
    TableBody,
    TableCell,
    TableHead,
    TableHeader,
    TableRow,
} from "@/components/ui/table"


interface Store {
    store_id: number
    store_name: string
}

const stores = ref<Store[]>([])
const selectedStores = ref<number[]>([])

const columns = ref<string[]>([])
const rows = ref<any[]>([])

/**
 * Date Range
 */
const dateFrom = ref<DateValue | null>(null)
const dateTo = ref<DateValue | null>(null)

/**
 * Format date untuk label
 */
function formatDate(date: DateValue | null) {
    if (!date) return ""
    return date.toDate(getLocalTimeZone()).toLocaleDateString("id-ID", {
        day: "2-digit",
        month: "short",
        year: "numeric",
    })
}

/**
 * Label untuk button
 */
const dateFromLabel = computed(() => {
    return dateFrom.value ? formatDate(dateFrom.value as DateValue) : "Pilih tanggal awal"
})

const dateToLabel = computed(() => {
    return dateTo.value ? formatDate(dateTo.value as DateValue) : "Pilih tanggal akhir"
})

const loadStores = async () => {
    const res = await api.get('/store')
    stores.value = res.data
}

watch([dateFrom, dateTo], ([from, to]) => {
    if (from && to) {
        // bandingkan tanggal
        if (to.compare(from) < 0) {
            toast.error("Tanggal akhir tidak boleh lebih kecil dari tanggal awal")

            // reset dateTo
            dateTo.value = null
        }
    }
})

function isDateDisabled(date: CalendarDate) {
    if (!dateFrom.value) return false

    return date.compare(dateFrom.value) < 0
}

const chartData = ref([]);

async function getDataReportAreaSummary(selectedStores: [], dateFrom: DateValue, dateTo: DateValue) {
    const res = await api.post('/report/area-summary', {
        store_ids: selectedStores,
        start_date: dateFrom,
        end_date: dateTo
    })

    const reportValue = res.data.map(item => ({
        name: item.area_name,
        value: Number(item.compliance) // convert string ke number
    }));

    chartData.value = reportValue;
}

async function getDataReportBrandSummary(selectedStores: [], dateFrom: DateValue, dateTo: DateValue) {
    const res = await api.post('/report/brand-area-summary', {
        store_ids: selectedStores,
        start_date: dateFrom,
        end_date: dateTo
    });

    columns.value = res.data.columns
    rows.value = res.data.rows
}

const loadChart = async () => {
    if (!selectedStores.value.length) {
        toast.warning('Gagal!', {
            description: "Store belum dipilih, Pilih store minimal 1"
        });
        return;
    }

    if (!dateFrom.value) {
        toast.warning('Gagal!', {
            description: "Pilih Tanggal sebelumnya terlebih dahulu"
        });
        return;
    }

    if (!dateTo.value) {
        toast.warning('Gagal!', {
            description: "Pilih Tanggal Akhir terlebih dahulu"
        });
        return;
    }

    fetchData(selectedStores.value as [], dateFrom.value as DateValue, dateTo.value as DateValue);
    
}

function fetchData(selectedStores: [], dateFrom: DateValue, dateTo: DateValue) {
    getDataReportAreaSummary(selectedStores, dateFrom, dateTo);
    getDataReportBrandSummary(selectedStores, dateFrom, dateTo);

    toast.success('Berhasil!', {
        description: "Data berhasil difetching"
    });

}

onMounted(async () => {
    loadStores().then(() => {
        // fething data per hari ini ketika awal page di load
        const storeIdsOnly = stores.value.map(store => store.store_id)
        fetchData(storeIdsOnly as [], today(getLocalTimeZone()), today(getLocalTimeZone()));
    });
});
</script>

<template>
    <Card class="space-y-4 m-10">
        <CardContent>
            <div class="grid grid-cols-12 gap-4 pt-5">

                <!-- Store -->
                <div class="col-span-12 md:col-span-4 lg:col-span-4 flex flex-col gap-3">
                    <Label class="px-1">Store</Label>

                    <Select v-model="selectedStores" multiple>
                        <SelectTrigger class="w-full">
                            <SelectValue placeholder="Pilih Store" />
                        </SelectTrigger>

                        <SelectContent>
                            <SelectItem v-for="store in stores" :key="store.store_id" :value="store.store_id">
                                {{ store.store_name }}
                            </SelectItem>
                        </SelectContent>
                    </Select>
                </div>

                <!-- Date From -->
                <div class="col-span-12 md:col-span-4 lg:col-span-3 flex flex-col gap-3">
                    <Label class="px-1">Select Date From</Label>

                    <Popover>
                        <PopoverTrigger as-child>
                            <Button variant="outline" class="w-full justify-between font-normal">
                                {{ dateFromLabel }}
                                <ChevronDownIcon />
                            </Button>
                        </PopoverTrigger>

                        <PopoverContent class="w-auto overflow-hidden p-0" align="start">
                            <Calendar v-model="dateFrom" />
                        </PopoverContent>
                    </Popover>
                </div>

                <!-- Date To -->
                <div class="col-span-12 md:col-span-4 lg:col-span-3 flex flex-col gap-3">
                    <Label class="px-1">Select Date To</Label>

                    <Popover>
                        <PopoverTrigger as-child>
                            <Button variant="outline" class="w-full justify-between font-normal">
                                {{ dateToLabel }}
                                <ChevronDownIcon />
                            </Button>
                        </PopoverTrigger>

                        <PopoverContent class="w-auto overflow-hidden p-0" align="start">
                            <Calendar v-model="dateTo" :is-date-disabled="isDateDisabled" />
                        </PopoverContent>
                    </Popover>
                </div>

                <!-- Button -->
                <div class="col-span-12 md:col-span-4 lg:col-span-2 flex flex-col gap-3 mt-6">
                    <Button class="w-full" @click="loadChart">
                        View
                    </Button>
                </div>
            </div>
        </CardContent>
    </Card>

    <!-- Chart -->
    <Card class="space-y-4 m-10">
        <CardHeader>
            <CardTitle>Compliance by Area</CardTitle>
        </CardHeader>

        <CardContent>
            <BarChart :data="chartData" />
        </CardContent>
    </Card>

    <!-- TABLE -->
    <Card class="m-10">
        <CardHeader>
            <CardTitle>Brand Compliance by Area</CardTitle>
        </CardHeader>

        <CardContent>
            <Table>
                <TableHeader>
                    <TableRow>
                        <TableHead v-for="col in columns" :key="col">
                            {{ col }}
                        </TableHead>
                    </TableRow>
                </TableHeader>

                <TableBody>
                    <TableRow v-for="(row, index) in rows" :key="index">
                        <TableCell v-for="col in columns" :key="col">
                            {{ col === 'BRAND' ? row.brand : row[col] }}
                        </TableCell>
                    </TableRow>
                </TableBody>
            </Table>
        </CardContent>
    </Card>


</template>
