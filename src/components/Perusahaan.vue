<template>
    <div id="content-page" class="content-page ml-0">
        <TopBar />
        <div class="container-fluid">
            <div class="row">
                <div class="col-lg-3">
                    <div class="iq-card iq-card-block iq-card-stretch iq-card-height">
                        <div class="iq-card-header d-flex justify-content-between">
                            <div class="iq-header-title">
                                <h4 class="card-title">Daftar Kapal</h4>
                            </div>
                            <span class="set-li" data-toggle="modal" data-target="#filter">
                                <a href="javascript:void(0)" data-toggle="modal" data-target="#filter" title="Filter Data">
                                    <i class="ri-filter-2-fill text-dark"></i>
                                </a>
                            </span>
                        </div>
                        <div class="iq-card-body pl-2 pr-0 pb-0">
                            <div class="form-group ml-1 mr-2">
                                <input :type="searchKapalBy != 'active_date' ? 'text' : 'date'"
                                    class="form-control" style="padding-right:135px"
                                    :placeholder="searchKapalBy == 'name' ? 'Cari Nama Kapal' : 'Cari SN Kapal'" 
                                    @keyup="cariNamaKapal" 
                                    v-model="CnmKapal"
                                >
                                <select class="form-control custom-select-search my-1 pl-0" 
                                    v-model="searchKapalBy" @change="CnmKapal = ''">
                                >
                                    <option value="name" selected>By Name</option>
                                    <option value="sn">By SN</option>
                                    <option value="active_date">By Active Date</option>
                                </select>
                                <a class="mod" href="#" @click="cariNamaKapal"><i class="ri-search-line"></i></a>
                            </div>
                            <ul class="doctors-lists m-0 p-0 iq-email-sender-list" style="height: 56vh;">
                                <li class="d-flex mb-2 align-items-center pr-1 set-list"
                                    v-for="(item, key) in kapal" 
                                    :key="key" 
                                    :class="{'active-li': item.id === selectedKapal && item.id !== null}"
                                    @click="selectedKapal = item.id ? item.id : null"
                                >
                                    <div class="user-img img-fluid">
                                        <img :src="!item.last_update || convertLastUpdate(item.last_update) < item.tglNow ? 'images/shipOff.png' : 'images/shipOn.png' "
                                            alt="story-img" class="rounded-circle avatar-40" />
                                    </div>
                                    <a href="javascript:void(0)" @click="getSingleKapal(item)" class="iq-email-title">
                                        <div class="media-support-info ml-3">
                                            <h6>{{item.name ? item.name : '-'}}</h6>
                                            <p class="mb-0 font-size-12">{{item.sn ? item.sn : '-'}}</p>
                                        </div>
                                    </a>
                                </li>
                            </ul>
                        </div>
                        <div class="iq-card-footer p-3" style="border-top:1px solid #eee">
                            <div class="row">
                                <div class="col-lg-6 text-left">
                                    Total Data
                                </div>
                                <div class="col-lg-6 text-right">
                                    <b>{{kapal.length}}</b>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-lg-9">
                    <div class="iq-card iq-card-block iq-card-stretch iq-card-height">
                        <div class="iq-card-body p-0">
                            <l-map class="map" style="height: 82vh; z-index: 0;" :zoom="zoom" :center="center">
                                <l-control-fullscreen />
                                <!-- tombol refresh data -->
                                <l-control position="topleft" 
                                    :class="{
                                        'hide-me': !stateRefresh,
                                        'leaflet-control': true
                                    }"
                                >
                                    <a href="javascript:void(0)" title="Back All Kapal" class="filtertrack" @click="getKapal(true)">
                                        <i class="ri-refresh-fill" style="font-size: 15px;"></i>
                                    </a>
                                </l-control>
                                <!-- tombol show modal tracking -->
                                <l-control position="topright"
                                    :class="{
                                        'hide-me': !stateTrack,
                                        'leaflet-control': true
                                    }"
                                >
                                    <a href="javascript:void(0)" title="Filter" data-toggle="modal" data-target="#tracing" class="filtertrack">
                                        <i class="ri-filter-2-fill"></i>
                                    </a>
                                </l-control>
                                <!-- tombol download data tracking -->
                                <l-control position="bottomleft"
                                    :class="{'hide-me': !stateDownload, 'leaflet-control': true}"
                                >
                                    <download-excel class="filtertrack" style="cursor: pointer"
                                        :data="markerRute" name="analisatracking.xls"
                                    >
                                        <i class="ri-file-download-fill" style="font-size: 15px;"></i>
                                        Download Analisa Tracking
                                    </download-excel>
                                </l-control>

                                <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
                                <l-geo-json
                                    :geojson="geojson" 
                                    :options-style="stylegeojson" 
                                />
                                <l-rotated-marker v-if="kapalSingle.length" :lat-lng="kapalSingle" :icon="icon"
                                    :rotationAngle="heading" ref="singleMarker">
                                    <l-popup class="map-popup">
                                        <table>
                                            <tr>
                                                <th width="50%">Nama Perusahaan</th>
                                                <th>:</th>
                                                <td>{{detailKapal.customer_name ? detailKapal.customer_name : '-'}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Nama Kapal</th>
                                                <th>:</th>
                                                <td>{{detailKapal.name ? detailKapal.name : '-'}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Vessel ID</th>
                                                <th>:</th>
                                                <td>{{detailKapal.id ? detailKapal.id : '-'}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">SN</th>
                                                <th>:</th>
                                                <td>{{detailKapal.sn ? detailKapal.sn : '-'}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Speed</th>
                                                <th>:</th>
                                                <td>{{detailKapal.speed ? detailKapal.speed : 0}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Heading</th>
                                                <th>:</th>
                                                <td>{{detailKapal.heading ? detailKapal.heading : '-'}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Latitude / Longitude</th>
                                                <th>:</th>
                                                <td>{{detailKapal.lat +' / '+ detailKapal.lon}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Atp Start / Atp End</th>
                                                <th>:</th>
                                                <td>{{(detailKapal.atp_start != null) ? detailKapal.atp_start : '-'}} / {{(detailKapal.atp_end != null) ? detailKapal.atp_end : '-'}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Provider</th>
                                                <th>:</th>
                                                <td>
                                                    {{detailKapal.provider}}
                                                </td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Last Update</th>
                                                <th>:</th>
                                                <td :class="(convertLastUpdate(detailKapal.last_update) < detailKapal.tglNow) ? 'text-danger' : ''">
                                                    {{(detailKapal.timestamp != null) ? timeZoneConvert(detailKapal.last_update) : '-'}}
                                                </td>
                                            </tr>
                                            <tr>
                                                <th width="50%">First Active</th>
                                                <th>:</th>
                                                <td>
                                                    {{first_active}}
                                                </td>
                                            </tr>
                                        </table>
                                        <hr>
                                        <div class="d-flex justify-content-between">
                                            <button class="btn btn-sm btn-outline-primary" data-toggle="modal" data-target="#tracing"
                                                @click="getTanggal(detailKapal.sn, detailKapal.provider)"
                                            >
                                                Tracing
                                            </button>
                                            <button class="btn btn-sm btn-outline-danger" 
                                                data-toggle="modal" data-target="#edit_kapal"
                                                @click="getKapalEdit(detailKapal.sn)"
                                            >
                                                Edit
                                            </button>
                                        </div>
                                    </l-popup>
                                </l-rotated-marker>
                                <!-- all marker -->
                                <l-rotated-marker v-for="(data, key) in markerKapal" :key="key"
                                    :lat-lng="[parseFloat(data.lat), parseFloat(data.lon)]"
                                    :icon="!data.last_update || convertLastUpdate(data.last_update) < data.tglNow ? iconOff : iconOn"
                                    :rotationAngle="data.heading ? parseInt(data.heading) : 0"
                                    @click="getFirstActive(data.sn, data.provider)">
                                    <l-popup class="map-popup">
                                        <table>
                                            <tr>
                                                <th width="50%">Nama Perusahaan</th>
                                                <th>:</th>
                                                <td>{{data.customer_name ? data.customer_name : '-'}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Nama Kapal</th>
                                                <th>:</th>
                                                <td>{{data.name ? data.name : '-'}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Vessel ID</th>
                                                <th>:</th>
                                                <td>{{data.id ? data.id : '-'}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">SN</th>
                                                <th>:</th>
                                                <td>{{data.sn ? data.sn : '-'}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Speed</th>
                                                <th>:</th>
                                                <td>{{data.speed ? data.speed : 0}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Heading</th>
                                                <th>:</th>
                                                <td>{{data.heading ? data.heading : '-'}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Latitude / Longitude</th>
                                                <th>:</th>
                                                <td>{{data.lat +' / '+ data.lon}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Atp Start / Atp End</th>
                                                <th>:</th>
                                                <td>{{(data.atp_start != null) ? data.atp_start : '-'}} / {{(data.atp_end != null) ? data.atp_end : '-'}}</td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Provider</th>
                                                <th>:</th>
                                                <td>
                                                    {{data.provider}}
                                                </td>
                                            </tr>
                                            <tr>
                                                <th width="50%">Last Update</th>
                                                <th>:</th>
                                                <td :class="convertLastUpdate(data.last_update) < data.tglNow ? 'text-danger' : ''">
                                                    {{(data.last_update != null) ? timeZoneConvert(data.last_update) : '-'}}
                                                </td>
                                            </tr>
                                            <tr>
                                                <th width="50%">First Active</th>
                                                <th>:</th>
                                                <td>
                                                   {{first_active}}
                                                </td>
                                            </tr>
                                        </table>
                                        <hr>
                                        <div class="d-flex justify-content-between">
                                            <button class="btn btn-sm btn-outline-primary" data-toggle="modal" 
                                                data-target="#tracing" @click="getTanggal(data.sn, data.provider)"
                                            >
                                                Tracing
                                            </button>
                                            <button class="btn btn-sm btn-outline-danger" 
                                                data-toggle="modal" data-target="#edit_kapal"
                                                @click="getKapalEdit(data.sn)"
                                            >  
                                                Edit
                                            </button>
                                        </div>
                                    </l-popup>
                                </l-rotated-marker>

                                <!-- rute marker -->
                                <l-rotated-marker v-for="(data, key) in markerRute" :key="key"
                                    :lat-lng="[parseFloat(data.Latitude), parseFloat(data.Longitude)]"
                                    :icon="(data.kecepatan != '0.0') ? iconOn: iconOff"
                                    :rotationAngle="data.Direction ? parseInt(data.Direction) : 0">

                                </l-rotated-marker>
                                <l-polyline v-if="lineLatLon" :lat-lngs="lineLatLon" :color="color"></l-polyline>
                            </l-map>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- filter modal -->
        <div class="modal fade" id="filter" tabindex="-1" aria-labelledby="filter" aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">Filter Data Kapal</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <div class="form-check mb-3">
                            <input v-model="filter" class="form-check-input" type="radio" id="default" 
                                value="default" checked>
                            <label class="form-check-label" for="default">
                                Order Default
                            </label>
                        </div>
                        <div class="form-check mb-3">
                            <input v-model="filter" class="form-check-input" type="radio" id="bynmkapal" 
                            value="bynmkapal">
                            <label class="form-check-label" for="bynmkapal">
                                Order By Nama Kapal
                            </label>
                        </div>
                        <div class="form-check mb-3">
                            <input v-model="filter" class="form-check-input" type="radio" id="bylastposisi" 
                            value="bylastposisi">
                            <label class="form-check-label" for="bylastposisi">
                                Order By Last Position
                            </label>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button type="button" class="btn btn-primary" @click="filterKapal()" data-dismiss="modal">Save changes</button>
                    </div>
                </div>
            </div>
        </div>
        <!-- tracing modal -->
        <div class="modal fade" ref="tracing" id="tracing" tabindex="-1" aria-labelledby="tracing" aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">Filter Data Tracking</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <div class="form-check">
                            <input v-model="traceOption" class="form-check-input" type="radio" id="last10day" value="last10day">
                            <label class="form-check-label" for="last10day">
                                Berapa Titik Lokasi Terakhir
                            </label>
                            <input type="number" class="form-control" style="width: 47%;" v-model="last10day">
                        </div>
                        <div class="form-check">
                            <input v-model="traceOption" class="form-check-input" type="radio" id="lasyhowday" value="lasyhowday">
                            <label class="form-check-label" for="lasyhowday">
                                Berapa Hari Terakhir ?
                            </label>
                            <input type="number" class="form-control" style="width: 47%;" placeholder="10" v-model="lasyhowday">
                        </div>
                        <div class="form-check">
                            <input v-model="traceOption" class="form-check-input" type="radio" id="filterdate" value="filterdate">
                            <label class="form-check-label" for="filterdate">
                                Filter Antara Tanggal
                            </label>
                            <div class="d-flex justify-content-between align-items-center">
                                <input type="date" class="form-control" style="width: 47%;" v-model="tgldari">
                                <span class="text-dark">S/D</span>
                                <input type="date" class="form-control" style="width: 47%;" v-model="tglsampai">
                            </div>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button type="button" class="btn btn-primary" @click="getRute()" data-dismiss="modal">Search</button>
                    </div>
                </div>
            </div>
        </div>
        <!-- edit kapal modal -->
        <div class="modal fade" id="edit_kapal" tabindex="-1" role="dialog" aria-hidden="true">
            <div class="modal-dialog modal-lg" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">Edit Kapal</h5>
                    </div>
                    <div class="modal-body">
                        <div class="row">
                            <div class="col-lg-6">
                                <div class="form-group">
                                    <label>Nama Customer</label>
                                    <multiselect v-model="customer" placeholder="Pilih customer"
                                        :options="listPerusahaan" :allow-empty="true" :close-on-select="true"
                                        deselect-label="Klik untuk batal memilih" select-label="Klik untuk memilih" selected-label="Terpilih"
                                        track-by="id" label="customer_name"
                                    ></multiselect>
                                </div>
                                <div class="form-group">
                                    <label>Vessel ID</label>
                                    <input v-model="editKapal.id" autocomplete="false" type="text" placeholder="Masukkan vessel ID" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>IMEI</label>
                                    <input v-model="editKapal.imei" autocomplete="false" type="text" placeholder="Masukkan nomor IMEI" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>Categori</label>
                                    <multiselect v-model="kategoriCustomer" placeholder="Pilih kategori customer" 
                                        :options="listTipeCustomer" :allow-empty="true" :close-on-select="true"
                                        deselect-label="Klik untuk batal memilih" select-label="Klik untuk memilih" selected-label="Terpilih"
                                        track-by="id" label="name"
                                    ></multiselect>
                                </div>
                                <div class="form-group">
                                    <label>Tipe Kapal</label>
                                    <multiselect v-model="tipeKapal" placeholder="Pilih tipe Kapal" 
                                        :options="listTipeKapal" :allow-empty="true" :close-on-select="true"
                                        deselect-label="Klik untuk batal memilih" select-label="Klik untuk memilih" selected-label="Terpilih"
                                        track-by="id" label="name"
                                    ></multiselect>
                                </div>
                            </div>
                            <div class="col-lg-6">
                                <div class="form-group">
                                    <label>ATP Start</label>
                                    <input v-model="editKapal.atp_start" autocomplete="false" type="date" placeholder="mm/dd/yyyy" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>ATP End</label>
                                    <input v-model="editKapal.atp_end" autocomplete="false" type="date" placeholder="mm/dd/yyyy" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>Device ID/SN</label>
                                    <input autocomplete="false" type="text" class="form-control" v-model="editKapal.sn" disabled>
                                </div>
                                <div class="form-group">
                                    <label>Nama Kapal</label>
                                    <input autocomplete="false" type="text" placeholder="Masukkan nama kapal" class="form-control" v-model="editKapal.name">
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button type="button" class="btn btn-primary" data-dismiss="modal" @click="updateKapal(editKapal.sn)">Save Changes</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import TopBar from './include/TopBar.vue'
import {
    LMap,
    LTileLayer,
    LGeoJson,
    LPopup,
    LPolyline,
    LControl
} from 'vue2-leaflet'
import L from 'leaflet';
import mapjson from '../assets/seazone.json'
import axios from 'axios';
import $ from 'jquery';
import moment from "moment-timezone";

L.Icon.Default.imagePath = '/';
L.Icon.Default.mergeOptions({
    iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
    iconUrl: require('leaflet/dist/images/marker-icon.png'),
    shadowUrl: require('leaflet/dist/images/marker-shadow.png'),
});

moment.tz.guess();

export default {
    name: "Perusahaan",
    components: {
        TopBar,
        LMap,
        LTileLayer,
        LGeoJson,
        LPopup,
        LPolyline,
        LControl
    },
    directives: {},
    data() {
        return {
            idakun: null,
            getUrl: null,
            filter: 'default',
            kapal: [],
            selectedKapal: null,
            markerKapal: [],
            markerRute: [],
            CnmKapal: null,
            searchKapalBy: 'name',
            tglHistori: null,
            deviceId: null,
            lineLatLon: [],
            color: 'gray',

            // state l-control visibility
            stateRefresh: false,
            stateTrack: false,
            stateDownload: false,

            // multiselect options
            listPerusahaan: [],
            listTipeCustomer: [],
            listTipeKapal: [],

            // form tracing kapal
            traceOption: null,
            tgldari: null,
            tglsampai: null,
            last10day: null,
            lasyhowday: null,

            // form edit kapal
            editKapal: {},
            customer: [],
            tipeKapal: [],
            kategoriCustomer: [],

            //data detail kapal
            detailKapal: {},
            first_active: "-",

            icon: null,
            heading: null,
            kapalSingle: [],

            //data maps
            url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
            attribution: '&copy; <a target="_blank" href="https://kapalpintar.co.id">KapalPintar</a> contributors<span class="mr-2"></span>',
            zoom: 5,
            geojson: null,
            stylegeojson: {
                weight: 2,
                color: 'grey',
                fillColor: '#ccc',
                fillOpacity: 0,
                dashArray: '3, 10',
            },
            center: [-5.3121961, 116.0877759],
            iconOn: L.icon({
                iconUrl: '/images/iconon.png',
                iconSize: [12, 12],
            }),
            iconOff: L.icon({
                iconUrl: '/images/iconoff.png',
                iconSize: [12, 12],
            }),
            provider:'-',
            checkedValue:'',
        };
    },
    beforeCreate: function () {
        if (this.$session.exists()) {
            if (this.$session.get("level") == null) {
                this.$router.push("/lacak/home");
            }else{
                if (this.$session.get('level') === 'root') {
                    this.$router.push("/perusahaan");
                }
            }
        } else {
            this.$router.push("/");
        }
    },
    created() {
        this.getJsonMap()
        this.getKapal()
        if(this.$session.get('level') === 'root') {
            this.getLists()
        }
    },
    mounted() {
        this.idakun = this.$session.get("level");
    },
    methods: {
        // created initiate
        async getJsonMap() {
            this.showloadingBar()
            try {
                this.geojson = await mapjson;
            } catch (error) {
                console.log(error);
            }
        },
        async getKapal(refresh_state = false) {
            try {
                this.markerKapal = [];
                this.markerRute = [];
                this.lineLatLon = [];
                this.showloadingBar();
                if (this.$session.get('level') === 'root') {
                    this.getUrl = 'https://track.kapalpintar.co.id/api/kapal_all';
                } else {
                    this.getUrl = `https://track.kapalpintar.co.id/api/kapal/${this.$session.get('id')}`;
                }
                
                if(refresh_state) {
                    this.heading = this.tglHistori = this.dari = this.sampai = this.deviceId = this.CnmKapal =  this.selectedKapal = null;
                    this.stateRefresh = this.stateTrack = this.stateDownload = false;
                    this.center = [-5.3121961, 116.0877759];
                    // if(this.$refs['singleMarker']) this.$refs['singleMarker'].mapObject.closePopup();
                }

                const response = await axios.get(this.getUrl);
                this.kapal = response.data;
                let addMarker = [];
                for (let i = 0; i < response.data.length; i++) {
                    if (response.data[i].lat != null && response.data[i].lat != '0.000000') {
                        addMarker.push(response.data[i]);
                    } 
                }

                this.markerKapal = addMarker;
                
                this.closeloadingBar();
            } catch (error) {
                console.log(error);
            }
        },
        async getLists() {
            try {
                const data_perusahaan = await axios.get('https://track.kapalpintar.co.id/api/data_perusahaan');
                const kategori_customer = await axios.get('https://track.kapalpintar.co.id/api/kategori_customers');
                const tipe_kapal = await axios.get('https://track.kapalpintar.co.id/api/tipe_kapal');

                this.listPerusahaan = data_perusahaan.data.results;
                this.listTipeCustomer = kategori_customer.data.results;
                this.listTipeKapal = tipe_kapal.data.results;
            } catch(err) {
                console.log(err);
            }
        },

        async filterKapal() {
            this.showloadingBar()
            try {
                this.markerKapal = [];
                this.markerRute = [];
                this.lineLatLon = [];
                this.kapal = [];
                if (this.$session.get('level') === 'root') {
                    if(this.filter == 'default'){
                        await this.getKapal(true);
                        return;
                    }else if(this.filter == 'bynmkapal'){
                        this.getUrl = 'https://track.kapalpintar.co.id/api/kapal_all_bynm';
                    }else if(this.filter == 'bylastposisi'){
                        this.getUrl = 'https://track.kapalpintar.co.id/api/kapal_all_lastposisi';
                    } else {
                        return;
                    }
                } else {
                    if(this.filter == 'default'){
                        await this.getKapal(true);
                        return;
                    }else if(this.filter == 'bynmkapal'){
                        this.getUrl = `https://track.kapalpintar.co.id/api/kapal_bynama/${this.$session.get('id')}`;
                    }else if(this.filter == 'bylastposisi'){
                        this.getUrl = `https://track.kapalpintar.co.id/api/kapal_lastposisi/${this.$session.get('id')}`;
                    } else {
                        return;
                    }
                }

                if(this.$refs['singleMarker']) this.$refs['singleMarker'].mapObject.closePopup();

                const response = await axios.get(this.getUrl);
                this.kapal = response.data;
                let addMarker = [];
                for (let i = 0; i < response.data.length; i++) {
                    if (response.data[i].lat != null && response.data[i].lat != '0.000000') {
                        addMarker.push(response.data[i]);
                    } 
                }

                this.markerKapal = addMarker;

                this.closeloadingBar();
            } catch (error) {
                console.log(error);
            }
        },
        async getSingleKapal(item) {
            try {
                this.markerKapal = [];
                this.markerRute = [];
                this.lineLatLon = [];
                this.tglHistori = null;
                this.first_active = 'Loading...';

                if (item.lat == '0.000000' || item.lat == null) {
                    this.popupPesan('Lokasi kapal tidak sesuai !')
                } else {
                    this.stateRefresh = this.stateTrack = true;
                    this.kapalSingle = [parseFloat(item.lat), parseFloat(item.lon)];
                    this.center = [parseFloat(item.lat), parseFloat(item.lon)];
                    this.icon = !item.timestamp || item.timestamp < item.tglNow ? this.iconOff : this.iconOn;
                    this.heading = item.heading ? parseInt(item.heading) : 0;
                    //popup
                    this.detailKapal = item;
                
                    await this.getTanggal(item.sn, item.provider);
                    this.$refs['singleMarker'].mapObject.openPopup();

                    //get first active
             
                    if (item.provider == 'PIV') {
                        this.getUrl = `https://track.kapalpintar.co.id/api/kapal/first_active/${item.sn.replace(/\s+/g, '').toLowerCase()}`;
                    } else if(item.provider == 'OB'){
                        this.getUrl = `https://track.kapalpintar.co.id/api/ocean_byte/first_active/${item.sn.replace(/\s+/g, '').toLowerCase()}`;
                    }
                    
                    const response = await axios.get(this.getUrl);
                
                    if (response.data.status == 0) {
                        this.first_active = "-";
                    } else {
                        this.first_active = this.timeZoneConvert(response.data.results.first_active);
                    }

                    this.provider = item.provider;
                }
                
            } catch (error) {
                console.log(error);
            }
        },
        async getFirstActive(sn, providerData){
            this.provider = providerData;
       
            this.first_active = 'Loading...';

            if (providerData == 'PIV') {
                this.getUrl = `https://track.kapalpintar.co.id/api/kapal/first_active/${sn.replace(/\s+/g, '').toLowerCase()}`;
            } else if(providerData == 'OB'){
                 this.getUrl = `https://track.kapalpintar.co.id/api/ocean_byte/first_active/${sn.replace(/\s+/g, '').toLowerCase()}`;
            }
            
            const response = await axios.get(this.getUrl);

            this.first_active = this.timeZoneConvert(response.data.results.first_active);
        },
        async cariNamaKapal() {
            try {
                this.kapalSingle = [];
                this.markerKapal = [];
                this.markerRute = [];
                this.lineLatLon = [];

                if (this.$session.get('level') === 'root') {
                    this.getUrl = 'https://track.kapalpintar.co.id/api/kapal/search_all';
                } else {
                    this.getUrl = `https://track.kapalpintar.co.id/api/kapal/search/${this.$session.get('id')}`;
                }

                if(this.$refs['singleMarker']) this.$refs['singleMarker'].mapObject.closePopup();

                const response = await axios.post(this.getUrl, {
                    search: this.CnmKapal,
                    by: (this.searchKapalBy ? this.searchKapalBy : 'name')
                });

                this.kapal = response.data;
                let addMarker = [];
                for (let i = 0; i < response.data.length; i++) {
                    if (response.data[i].lat != null && response.data[i].lat != '0.000000') {
                        addMarker.push(response.data[i]);
                    } 
                }
                this.markerKapal = addMarker;

                if (this.kapal.length == 0) {
                    this.popupPesan('Data kapal tidak ditemukan');
                }
                
            } catch (error) {
                console.log(error);
            }
        },
        async getTanggal(sn, providerData) {
            this.showloadingBar();
            try {
                var urlGetTgl = '';
                if (providerData == 'PIV') {
                    urlGetTgl = `https://track.kapalpintar.co.id/api/histori_tgl/${sn}`;
                } else if(providerData == 'OB'){
                    urlGetTgl = `https://track.kapalpintar.co.id/api/ocean_byte/histori_tgl/${sn}`;
                }
                const response = await axios.get(urlGetTgl);
                this.tglHistori = response.data;
                this.tgldari = response.data.length ? response.data[0].tanggal : null;
                this.tglsampai = new Date().toISOString().slice(0, 10);
                this.deviceId = sn;
                this.closeloadingBar();
            } catch (error) {
                this.closeloadingBar();
                console.log(error);
            }
        },
        async getRute() {
            this.showloadingBar();
            
            try {
                let data = {dari: this.tgldari, sampai: this.tglsampai, jumltitik: 10, tipefilter: null};

                if(this.traceOption == 'last10day') {
                    data.jmltitik = this.last10day;
                    data.tipefilter = this.traceOption;
                } else if(this.traceOption == 'lasyhowday') {
                    let day = parseInt(this.lasyhowday) ? parseInt(this.lasyhowday) : 10;
                    data.dari = new Date(new Date().getTime() - (day * 24 * 60 * 60 * 1000)).toISOString().slice(0, 10);
                    data.tipefilter = this.traceOption;
                } else if(this.traceOption == 'filterdate') {
                    data.dari = this.tgldari;
                    data.sampai = this.tglsampai;
                    data.tipefilter = this.traceOption;
                } else {
                    this.closeloadingBar();
                    this.$swal.fire({
                        title: 'Pilih Filter',
                        text: 'Silahkan pilih filter terlebih dahulu',
                        type: 'warning',
                        confirmButtonText: 'OK'
                    });

                    return;
                }

                
                // if(this.$refs['singleMarker']) this.$refs['singleMarker'].mapObject.closePopup();
                var urlgettrack = '';
                if (this.provider == 'OB') {
                    urlgettrack = `https://track.kapalpintar.co.id/api/ocean_byte/device_histori/${this.deviceId}`;
                } else {
                    urlgettrack = `https://track.kapalpintar.co.id/api/histori_kapal/${this.deviceId}`;
                }
                const response = await axios.post(urlgettrack,data);
                
                if (response.data['histori'].length == 0) {
                    this.popupPesan('Data tracking tidak ditemukan !');
                    return;
                }else{
                    $(".leaflet-popup-close-button")[0].click();
                    this.stateRefresh = this.stateTrack = this.stateDownload = true;
                    this.markerKapal = [];
                    this.kapalSingle = [];
                    this.markerRute = response.data['histori'];
                    this.lineLatLon = response.data['polyline'];
                }
                
                this.closeloadingBar()
            } catch (error) {
                console.log(error);
            }
        },

        // edit kapal
        async getKapalEdit(sn) {
            try {
                this.showloadingBar();

                const kapal_by_sn = await axios.get(`https://track.kapalpintar.co.id/api/kapal_sn/${sn}`);
                // reconfig editKapal properties
                this.editKapal = kapal_by_sn.data[0];
                this.editKapal.tipeForm = 'Edit';
                if(this.editKapal.atp_start) this.editKapal.atp_start = this.formatISODate(this.editKapal.atp_start);
                if(this.editKapal.atp_end) this.editKapal.atp_end = this.formatISODate(this.editKapal.atp_end);

                if(this.editKapal.customer) {
                    const customer = await axios.get(`https://track.kapalpintar.co.id/api/perusahaan/${this.editKapal.customer}`);
                    this.customer = customer.data.results[0];
                }
                if(this.editKapal.type_id) {
                    const tipe_kapal = await axios.get(`https://track.kapalpintar.co.id/api/tipe_kapal/${this.editKapal.type_id}`);
                    this.tipeKapal = tipe_kapal.data.results[0];
                }
                if(this.editKapal.category_id) {
                    const kategori_customer = await axios.get(`https://track.kapalpintar.co.id/api/kategori_customers/${this.editKapal.category_id}`);
                    this.kategoriCustomer = kategori_customer.data.results[0];
                }

                this.closeloadingBar();
            } catch(err) {
                this.closeloadingBar()
                console.log(err)
            }
        },
        async updateKapal(sn) {
            if(!sn) {
                this.$swal.fire({
                    title: 'Kapal tidak ditemukan',
                    text: 'Silahkan coba lagi',
                    type: 'warning',
                    confirmButtonText: 'OK'
                });
                return;
            }

            try {
                this.showloadingBar();

                this.editKapal.id_customer = this.customer.id ? this.customer.id : null;
                this.editKapal.type_id = this.tipeKapal.id ? this.tipeKapal.id : null;
                this.editKapal.category_id = this.kategoriCustomer.id ? this.kategoriCustomer.id : null;

                await axios.put(`https://track.kapalpintar.co.id/api/updatepemilikkapal/${sn}`, this.editKapal);

                this.searchKapalBy = 'name';
                this.CnmKapal = this.editKapal.name;
                await this.cariNamaKapal();
                this.$swal.fire({
                    title: 'Berhasil',
                    text: 'Data berhasil diubah',
                    type: 'success',
                    confirmButtonText: 'OK'
                });
            } catch(err) {
                this.closeloadingBar();
                console.log(err);
            }
        },

        showloadingBar() {
            this.$swal.fire({
                html: "<img src='images/loading-bar.gif' style='width: 50px;'/><p>Loading...</p>",
                showConfirmButton: false,
            });
        },
        closeloadingBar() {
            this.$swal.fire({
                showConfirmButton: false,
            }).close();
        },
        popupPesan(msg){
            this.$swal.fire({
                text: msg,
                icon: 'warning',
                confirmButtonText: 'OK'
            });
        },
        formatISODate(date) {
            let d = new Date(date);
            
            return d.toISOString().split('T')[0];
        },
        timeZoneConvert(date){
            if (date != null) {
                var dateFormat = 'DD-MM-YYYY HH:mm:ss';
                var myDate =  moment(date,"DD-MM-YYYY HH:mm:ss").format("YYYY-MM-DD HH:mm:ss");

                var testDateUtc = moment.utc(myDate);
                var localDate = testDateUtc.local();
                
                var ketLocal = '';
                var strLocal = moment.tz.guess();
                if (strLocal == 'Asia/Jakarta') {
                    ketLocal = 'WIB';
                } else if(strLocal == 'Asia/Ujung_Pandang'){
                    ketLocal = 'WITA';
                }else if(strLocal == 'Asia/Jayapura'){
                    ketLocal = 'WIT';
                }
                return localDate.format(dateFormat)+' '+ ketLocal;
            }
        },
        convertLastUpdate(date){
            if (date != null) {
                var dateFormat = 'DD-MM-YYYY';
                var myDate =  moment(date,"DD-MM-YYYY HH:mm:ss").format("YYYY-MM-DD HH:mm:ss");

                var testDateUtc = moment.utc(myDate);
                var localDate = testDateUtc.local();
            
                return localDate.format(dateFormat);
            } 
        }
    },
};
</script>

<style scoped>
li .iq-email-title{
    width: 100%;
}
.doctors-lists li{
    padding-bottom: 5px;
    padding-top: 5px;
}
.doctors-lists li:hover{
    background: #ebf4f5;
    border-radius: 50px;
}
.active-li {
    background: #ceebee;
    border-radius: 50px;
}
.map{
    border-radius: 25px;
}
.mod{
    position: absolute;
    right: 30px;
    top: 82px;
    font-size: 15pt;
}
.custom-select-search {
    position: absolute;
    height: 40px;
    right: 55px;
    top: 78px;
    max-width: 100px;
    border: 0px;
    outline: 0px;
}
.set-li {
    border-left: 1px solid grey;
    padding-left: 10px;
    border-left-style: dashed;
}
.filtertrack {
    padding: 10px;
    color: #000;
    background: #fff;
    border: 2px solid #89a9b2;
    border-radius: 3px;
}
.map-popup {
    min-width: 300px;
}
.set-list {
    padding: 10px 0 10px 10px;
    border-bottom: 1px solid #efefef;
    border-bottom-style: dashed;
}
.hide-me {
    visibility: hidden;
}
</style>
