<script setup>
import SideBar from "../components/SideBar.vue"
import Header from "../components/Header.vue"
import LineChart from "../components/Chart/LineChart.vue"
import radial from "../components/Chart/radialBarDashboard.vue"
import axios from "axios"
import { reactive, ref } from "@vue/reactivity"
import { onMounted, watch } from "@vue/runtime-core"

//reactive state
const jenis_mesin = ref([]);
const jenis_mesin_count = ref(0);
const mesin = ref([]);
const mesin_count = ref(0)
const user = ref([]);
const mesin_anomali = ref([]);
const data_anomali = reactive({
    id_mesin_anomali: "",
    data_sensor: [],
})
const id_mesin_anomali = ref("");
const data_sensor = ref([]);
const filter = reactive({
    chart: "all",
})

// get jenis mesin data and insert max 6 data to jenis_mesin state
async function getJenisMesin(){
    try {
        await axios.get('/api/jenis_mesin/all?limit=6').then((response)=>{
            jenis_mesin_count.value = response.data.length;
            jenis_mesin.value = response.data
        })
    } catch (error) {
        console.log(error)
    }
}

// get all monitoring data and insert max 6 data to mesin state
async function getAllMonitoring(){
    try {
        await axios.get("/api/mesin/all?limit=6").then((response)=>{
            mesin_count.value = response.data.length;
            mesin.value = response.data;

            let anomali = mesin.value.filter((item)=>item.sensor_ai.latest_data_ai[0].kondisi_kesehatan <= 50)

            if(anomali.length < 4){
                mesin_anomali.value = []
                mesin_anomali.value = anomali
            }else {
                mesin_anomali.value = []
                for(let i = 0; i <= 3; i++){
                    console.log(anomali[i])
                    mesin_anomali.value.push(anomali[i])
                    console.log(mesin_anomali.value)
                }
            }
        })
    } catch (error) {
        console.log(error)
    }
}

const getListUser = async()=>{
    try {
        await axios.get('/api/user/all').then((res)=>{
            user.value = res.data.filter((item)=>item.hak_akses !== 1);
        })
    } catch (error) {
        console.log(error)
    }
}

watch(()=>data_anomali.id_mesin_anomali, async function(){
    data_anomali.data_sensor = [];
    await getAllMonitoring();
    let filter = mesin.value.filter((item)=>item._id === data_anomali.id_mesin_anomali)
    data_anomali.data_sensor = filter[0].sensor_result.data_sensor;
})

watch(()=>filter.chart,async function(){
    data_anomali.data_sensor = [];
    await getAllMonitoring();
    let filter = mesin.value.filter((item)=>item._id === data_anomali.id_mesin_anomali)
    data_anomali.data_sensor = filter[0].sensor_result.data_sensor;
})


onMounted(async()=>{
    // setInterval(()=>{
    //     if(mesin_anomali.value.length > 0){
    //         console.log("Gagal")
    //     }
    // }, 3000)
    await getJenisMesin();
    await getAllMonitoring();
    await getListUser();
    data_anomali.id_mesin_anomali = mesin_anomali.value[0]._id
    data_anomali.data_sensor = mesin_anomali.value[0].sensor_result.data_sensor;
})
</script>

<style scoped>
    .b-shadow{
        box-shadow: 3px 5px 6px rgba(0, 0, 0, 0.25);
    }
</style>

<template>
    <div class="flex bg-gray-100">
        <div >
            <SideBar />
        </div>
        <div class="w-full overflow-y-auto h-screen">
            <Header />
            <section class="p-10 flex flex-col gap-9">
                <div class="heading">
                    <h1 class="font-bold text-gray-200 text-4xl">Dashboard</h1>
                    <h1 class="font-medium text-gray-200 text-2xl mt-1.5">Dashboard / </h1>
                </div>
                <div class="detail">
                    <div class="grid 2xl:grid-cols-4 lg:grid-cols-2 gap-7 grid-cols-1">
                        <div class="bg-light rounded-2xl b-shadow">
                            <div class="side w-10 h-full float-left rounded-lg" style="background-color: #004f9b"></div>
                            <div class="flex items-center px-7 justify-center">
                                <span class="number text-main_black" style="font-size: 70px">{{jenis_mesin_count}}</span>
                                <div class="desc px-7 text-lg font-bold text-main_black">Machine Types</div>
                            </div>
                        </div>
                        <div class="bg-light rounded-2xl b-shadow">
                            <div class="side w-10 h-full float-left rounded-lg" style="background-color: #004f9b"></div>
                            <div class="flex items-center px-7 justify-center">
                                <span class="number text-main_black" style="font-size: 70px">{{mesin_count}}</span>
                                <div class="desc px-7 text-lg font-bold text-main_black">Machine Monitoring</div>
                            </div>
                        </div>
                        <div class="bg-light rounded-2xl b-shadow">
                            <div class="side w-10 h-full float-left rounded-lg" style="background-color: #004f9b"></div>
                            <div class="flex items-center px-7 justify-center">
                                <span class="number text-main_black" style="font-size: 70px">{{mesin_anomali.length}}</span>
                                <div class="desc px-7 text-lg font-bold text-main_black">Machine Anomaly</div>
                            </div>
                        </div>
                        <div class="bg-light rounded-2xl b-shadow">
                            <div class="side w-10 h-full float-left rounded-lg" style="background-color: #004f9b"></div>
                            <div class="flex items-center px-7 justify-center">
                                <span class="number text-main_black" style="font-size: 70px">{{user.length > 0 ? user.length : 0}}</span>
                                <div class="desc px-7 text-lg font-bold text-main_black">Users</div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="mesin">
                    <div class="grid 2xl:grid-cols-2 gap-7">
                        <div class="bg-light b-shadow py-6 px-7 rounded-2xl">
                            <h3 class="font-bold text-xl text-center relative">
                                Machine Types
                                <span class="absolute right-0 text-sm mt-2 italic underline hover:cursor-pointer" @click="$router.push({name: 'JenisMesin'})">See More ></span>
                            </h3>
                            <div class="list mt-4 text-lg">
                                <div class="flex justify-between" v-for="(daftar_mesin, idx) in jenis_mesin" :key="idx">
                                    <p>
                                        <span>{{idx+1}}. </span>
                                        <span>{{daftar_mesin.jenis_mesin}}</span>
                                    </p>
                                </div>
                            </div>
                        </div>
                        <div class="bg-light b-shadow py-6 px-7 rounded-2xl">
                            <h3 class="font-bold text-xl text-center relative">
                                Monitoring
                                <span class="absolute right-0 text-sm mt-2 italic underline hover:cursor-pointer" @click="$router.push({name: 'Monitoring'})">See More ></span>
                            </h3>
                            <div class="list mt-4 text-lg">
                                <div v-for="(msn,i) in mesin" :key="i">
                                    <p>
                                        <span>{{i+1}}. </span>
                                        <span>{{msn.nama_mesin}} | {{msn.kode_mesin}}</span>
                                    </p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="kesehatan">
                    <div class="bg-light b-shadow py-6 px-7 rounded-2xl">
                            <h3 class="text-center font-bold text-xl relative">
                                Machine Health
                                <span class="absolute right-0 text-sm mt-2 italic underline hover:cursor-pointer" @click="$router.push({name: 'Monitoring'})">See More ></span>
                            </h3>
                            <div class="list mt-9 text-lg grid 2xl:grid-cols-4 lg:grid-cols-2 md:grid-cols-1 gap-8">
                                <div v-for="(msn,i) in mesin_anomali" :key="i" class="relative hover:cursor-pointer" @click="$router.push({name:'DetailMonitoring', params:{_id: msn._id}})">
                                    <radial :health="msn.sensor_ai.latest_data_ai[0].kondisi_kesehatan" :labels="msn.nama_mesin" :indikasi="msn.sensor_ai.latest_data_ai[0].indikasi_kerusakan"></radial>
                                    <p class="text-center font-bold text-2xl lg:-mt-20 md:mt-0">{{msn.nama_mesin}}</p>
                                    <p class="text-center mt-3 font-bold">{{msn.sensor_ai.kode_mesin}}</p>
                                    <p class="text-center mt-1">{{msn.sensor_ai.latest_data_ai[0].indikasi_kerusakan}}</p>
                                </div>
                            </div>
                    </div>
                </div>
                <div class="chart" v-if="mesin_anomali.length > 0">
                    <div class="bg-light b-shadow py-6 px-7 rounded-2xl">
                        <h3 class="text-left font-bold text-xl">Machine needs Attention</h3>
                        <div class="flex justify-between">
                            <h3 class="text-xl my-4">
                                <select v-model="data_anomali.id_mesin_anomali">
                                    <option v-for="anomali in mesin_anomali" :key="anomali._id" :value="anomali._id" @click="changeGraphic(anomali._id)">
                                        {{anomali.nama_mesin}} - {{anomali.kode_mesin}}
                                    </option>
                                </select>
                            </h3>
                            <div>
                                <select v-model="filter.chart" class="cursor-pointer focus:outline-none text-xl">
                                    <option value="all">All</option>
                                    <option value="percepatan">Acceleration</option>
                                    <option value="kecepatan">Velocity</option>
                                    <option value="suhu">Temperature</option>
                                </select>
                            </div>
                        </div>
                        <div v-if="data_anomali.data_sensor.length > 0">
                            <line-chart :data_sensor="data_anomali.data_sensor" :filter="filter.chart"></line-chart>
                        </div>
                    </div>
                </div>
            </section>
        </div>
    </div>
</template>