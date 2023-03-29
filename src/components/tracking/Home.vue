<template>
  <div>
    <div class="search-text form-group mt-1">
      <label><b>No Transmiter/ Device ID</b></label>
      <div class="row">
        <div class="col-9 pr-0">
          <input
            v-model="cariDeviceID"
            autocomplete="false"
            type="text"
            placeholder="Masukkan No Transmiter/ Device ID"
            class="form-control"
          />
        </div>
        <div class="col-3">
          <button
            style="width: 100%; border-radius: 5px"
            class="btn btn-primary lineHeight"
            @click="getKapal()"
          >
            <b>LACAK</b>
          </button>
        </div>
      </div>
    </div>
    <div
      v-if="showDetail"
      class="col-3 pr-0"
      style="
        position: absolute;
        bottom: 1.5%;
        z-index: 999;
        transition-timing-function: ease-in;
      "
    >
      <div class="card" style="box-shadow: 1px 3px 16px #9b9b9b;">
        <div class="card-header">
          <h4>Informasi Detail</h4>
        </div>
        <div class="card-body">
          <table class="table">
            <tr>
              <td style="text-align: right; padding-left: 0px">
                <img src="../../assets/ico_perusahaan.png" class="img-set" />
              </td>
              <td>{{ namaPerusahaan }}</td>
            </tr>
            <tr>
              <td style="text-align: right; padding-left: 0px">
                <img src="../../assets/ico_shipe.png" class="img-set" />
              </td>
              <td>{{ namaKapal }}</td>
            </tr>
            <tr>
              <td style="text-align: right; padding-left: 0px">
                <img src="../../assets/ico_location.png" class="img-set" />
              </td>
              <td>{{ dmsData }}</td>
            </tr>
            <tr>
              <td style="text-align: right; padding-left: 0px">
                <img src="../../assets/ico_time.jpeg" class="img-set" />
              </td>
              <td>{{ timeZoneConvert(tglUpdate) }}</td>
            </tr>
            <tr>
              <td style="text-align: right; padding-left: 0px">
                <img src="../../assets/provider.png" class="img-set" />
              </td>
              <td>{{ provider }}</td>
            </tr>
          </table>
        </div>
      </div>
    </div>
    <l-map class="map" style="position: absolute" :zoom="zoom" :center="center">
      <l-control position="topright" style="margin-top: 25px">
        <a
          href="javascript:void(0)"
          class="filtertrack"
          @click="signout"
          :title="username"
          style="color: #ffffff"
        >
          <i class="ri-user-fill" style="font-size: 17px"></i>
        </a>
      </l-control>
      <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
      <l-geo-json :geojson="geojson" :options-style="stylegeojson" />
      <l-marker
        v-if="kondisiLatLon"
        :lat-lng="[dataLat, dataLon]"
        :icon="iconOn"
      >
      </l-marker>
    </l-map>
  </div>
</template>

<script>
import axios from "axios";
import { LMap, LTileLayer, LGeoJson, LControl, LMarker } from "vue2-leaflet";
import L from "leaflet";
import mapjson from "../../assets/seazone.json";
import moment from "moment-timezone";
import LatLontoDms from "latlng-to-dms";

moment.tz.guess();

export default {
  name: "Home",
  components: {
    LMap,
    LTileLayer,
    LGeoJson,
    LControl,
    LMarker,
  },
  data() {
    return {
      activeMenu: null,
      username: null,
      idPerusahaan: null,
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a target="_blank" href="https://kapalpintar.co.id">KapalPintar</a> contributors<span class="mr-2"></span>',
      zoom: 5,
      geojson: null,
      stylegeojson: {
        weight: 2,
        color: "grey",
        fillColor: "#ccc",
        fillOpacity: 0,
        dashArray: "3, 10",
      },
      iconOn: L.icon({
        iconUrl: "/images/marker.png",
        iconSize: [35, 55],
      }),
      center: [-2.0419296, 113.327494],
      cariDeviceID: null,
      dataLat: null,
      dataLon: null,
      kondisiLatLon: false,
      namaPerusahaan: null,
      namaKapal: null,
      tglUpdate: null,
      dmsData: null,
      showDetail: false,
      provider: null,
    };
  },
  beforeCreate: function () {
    if (this.$session.exists()) {
      if (this.$session.get("level") == null) {
        this.$router.push("/lacak/home");
      } else {
        this.$router.push("/perusahaan");
      }
    } else {
      this.$router.push("/lacak");
    }
  },
  created() {
    this.getData();
    this.getJsonMap();
  },
  methods: {
    async getJsonMap() {
      this.showloadingBar();
      try {
        this.geojson = await mapjson;
        this.closeloadingBar();
      } catch (error) {
        console.log(error);
      }
    },
    async getData() {
      try {
        this.username = this.$session.get("username");
        this.idPerusahaan = this.$session.get("id_customer");
      } catch (error) {
        console.log(error);
      }
    },
    async getKapal() {
      try {
        this.dataLat = "";
        this.dataLon = "";
        this.showloadingBar();
        const response = await axios.get(
          `https://track.kapalpintar.co.id/api/kapal_location/${this.cariDeviceID}`
        );
        const result = response.data;

        if (result.status === 1) {
          this.dataLat = parseFloat(result.data["latitude"]);
          this.dataLon = parseFloat(result.data["longitude"]);
          this.namaPerusahaan = result.data["nama_perusahaan"];
          this.namaKapal = result.data["nama_kapal"];
          this.tglUpdate = result.data["timestamp"];
          const latlon =
            result.data["latitude"] + "," + result.data["longitude"];
          this.dmsData = LatLontoDms(latlon);
          this.provider = result.data["provider"];

          this.kondisiLatLon = true;
          this.closeloadingBar();
          this.center = await [this.dataLat, this.dataLon];
          setTimeout(() => {
            this.zoom = 6.5;
          }, 500);
          this.showDetail = true;
        } else {
          this.popupPesan("Maaf, data tidak ditemukan !");
        }
      } catch (error) {
        console.log(error);
      }
    },
    async signout() {
      this.$swal
        .fire({
          text: "Apakah yakin ingin keluar?",
          showDenyButton: true,
          confirmButtonText: `Ya`,
          denyButtonText: `Tidak`,
        })
        .then((result) => {
          if (result.isConfirmed) {
            this.$session.clear();
            this.$session.destroy();
            this.$router.go();
          }
        });
    },
    timeZoneConvert(date) {
      if (date != null) {
        var dateFormat = "DD-MM-YYYY HH:mm:ss";
        var myDate = moment(date, "YYYY-MM-DD HH:mm:ss").format(
          "YYYY-MM-DD HH:mm:ss"
        );

        var testDateUtc = moment.utc(myDate);
        var localDate = testDateUtc.local();

        var ketLocal = "";
        var strLocal = moment.tz.guess();
        if (strLocal == "Asia/Jakarta") {
          ketLocal = "WIB";
        } else if (strLocal == "Asia/Ujung_Pandang") {
          ketLocal = "WITA";
        } else if (strLocal == "Asia/Jayapura") {
          ketLocal = "WIT";
        }
        return localDate.format(dateFormat) + " " + ketLocal;
      }
    },
    showloadingBar() {
      this.$swal.fire({
        html: "<img src='../images/loading-bar.gif' style='width: 50px;'/><p>Loading...</p>",
        showConfirmButton: false,
      });
    },
    closeloadingBar() {
      this.$swal
        .fire({
          showConfirmButton: false,
        })
        .close();
    },
    popupPesan(msg) {
      this.$swal.fire({
        text: msg,
        icon: "warning",
        confirmButtonText: "OK",
      });
    },
  },
};
</script>

<style>
.akun {
  background: #0db3c6;
  color: #ffffff;
}
.filtertrack {
  padding: 14px;
  background: #0db3c6;
  border: 2px solid #0db3c6;
  border-radius: 50%;
  box-shadow: 1px 3px 16px #9b9b9b;
}
.form-control {
  border-radius: 5px;
}
.search-text {
  position: absolute;
  left: 20%;
  right: 20%;
  z-index: 999;
  background: #ffffff;
  padding: 20px;
  border-radius: 5px;
  top: 10px;
  box-shadow: 1px 3px 16px #9b9b9b;
}
.lineHeight {
  line-height: 1.8rem;
}

.table td,
.table th {
  border-top: 0px solid;
  border-bottom: 1px solid rgb(180, 180, 180);
}
.table {
  margin-bottom: 0rem;
}
.card-body {
  padding: 0px;
}
.img-set {
  width: 20px;
}
</style>