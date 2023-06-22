<template>
    <div id="content-page" class="content-page ml-0">
        <TopBar />
        <div class="container-fluid">
            <div class="row">
                <div class="col-lg-3">
                    <div class="iq-card iq-card-block iq-card-stretch iq-card-height">
                        <div class="iq-card-header d-flex justify-content-between">
                            <div class="iq-header-title">
                                <h4 class="card-title">Customers</h4>
                            </div>
                            <button class="btn btn-outline-primary btn-sm" 
                                data-toggle="modal" data-target="#form_customer" 
                                @click="statusForm = 'create'"
                            >
                                Tambah
                            </button>
                        </div>
                        <div class="iq-card-body pl-2 pr-0 pt-2 pb-0">
                            <div class="form-group ml-1 mr-2">
                                <input type="text" class="form-control" 
                                    style="padding-right:30px"
                                    placeholder="Cari Nama Customer" 
                                    v-model="CnmCustomer"
                                    @keyup="searchCustomer()"
                                >
                                <a class="search-link mod" href="#" @click="searchCustomer()">
                                    <i class="ri-search-line"></i>
                                </a>
                            </div>
                            <div class="row" v-if="perusahaan">
                                <div class="col-lg-4 pt-1"></div>
                                <div class="col-lg-8 text-right mb-m10">
                                    <button type="button" 
                                        class="btn btn-sm btn-outline-success mr-2"
                                        data-toggle="modal" data-target="#form_customer"
                                        @click="editCustomer()"
                                    >
                                        Edit Data
                                    </button>
                                    <button type="button" 
                                        class="btn btn-sm btn-outline-danger mr-2"
                                        @click="deleteCustomer(); akun.email = null"
                                    >
                                        Hapus
                                    </button>
                                    <button type="button" 
                                        class="btn btn-sm btn-outline-dark mr-2"
                                        @click="perusahaan = kapal = liCustomerActive = null"
                                    >
                                        Cancel
                                    </button>
                                </div>
                            </div>
                            <hr class="mb-2"/>
                            <ul class="doctors-lists m-0 p-0 iq-email-sender-list" style="height: 56vh;">
                                <li class="set-list d-flex mb-2 align-items-center pr-1"
                                    v-for="item in customer" 
                                    :key="item.id" 
                                    :class="{'active-li': item.id === liCustomerActive}"
                                    @click="getSingleCustomer(item.id)"
                                >
                                    <div class="user-img img-fluid">
                                        <img src="images/customer.png" alt="customer icon" class="rounded-circle avatar-40" />
                                    </div>
                                    <a href="javascript:void(0)" class="iq-email-title">
                                        <div class="media-support-info ml-3">
                                            <h6>{{item.customer_name ? item.customer_name : '-'}}</h6>
                                            <p class="mb-0 font-size-12">{{item.contact ? item.contact : '-'}}</p>
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
                                    <b>{{customer.length}}</b>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <!-- bila informasi customer tidak ada yang di-select -->
                <div v-if="!perusahaan && !kapal" class="col-lg-9">
                    <div class="mt-5 text-center" width="100%">
                        <h5 class="mb-5" style="margin-top: 10%; color: #ffc727">
                            Data customer belum dipilih !
                        </h5>
                        <img src="images/5667051.png" class="mt-2">
                    </div>
                </div>
                <!-- bila ada customer yang di-select - start -->
                <div class="col-lg-6" v-if="perusahaan">
                    <div class="row m-0">
                        <div class="iq-card iq-user-profile-block w-100" style="height: 83%;">
                            <div class="iq-card-body" style="padding: 10px;">
                                <div class="user-details-block">
                                    <div class="user-profile text-center">
                                        <img src="images/user/111.png" alt="profile-img" class="avatar-130 img-fluid">
                                    </div>
                                    <div class="text-center mt-3">
                                        <h5>{{ perusahaan.customer_name }}</h5>
                                        <p :class="{
                                            'text-danger': !akun.email,
                                            'text-primary': akun.email
                                        }">
                                            {{ !akun.email ? '-' : akun.status }}
                                        </p>
                                        <p v-if="!akun.email" class="text-danger">Customer belum mempunyai akun! 
                                            <button data-toggle="modal" data-target="#form_akun"
                                                data-backdrop="static" data-keyboard="false"
                                                class="btn btn-sm btn-outline-danger"
                                                @click="statusAkun = 'create'; akun.password = null"
                                            >
                                                Create !
                                            </button>
                                        </p>
                                        <p v-if="akun.email" class="text-primary">Customer sudah mempunyai akun! 
                                            <button data-toggle="modal" data-target="#form_akun"
                                                data-backdrop="static" data-keyboard="false"
                                                class="btn btn-sm btn-outline-primary"
                                                @click="statusAkun = 'edit'; editAkun(perusahaan.id);"
                                            >
                                                Edit !
                                            </button>
                                        </p>
                                    </div>
                                    <hr style="margin-bottom: 0rem;">
                                    <div class="iq-card-body pb-0" style="padding: 15px;">
                                        <h6 class="card-title text-primary">Data Perusahaan</h6>
                                        <div class="row">
                                            <div class="col-lg-4">
                                                <div class="media-support-info ml-3">
                                                    <h6>NPWP</h6>
                                                    <p class="mb-0">{{ perusahaan.npwp ? perusahaan.npwp : '-' }}</p>
                                                    <h6>Contact</h6>
                                                    <p class="mb-0">{{ perusahaan.contact ? perusahaan.contact : '-' }}</p>
                                                    <h6>Agent</h6>
                                                    <p class="mb-0">{{ perusahaan.agent ? perusahaan.agent : '-' }}</p>
                                                </div>
                                            </div>
                                            <div class="col-lg-4">
                                                <div class="media-support-info ml-3">
                                                    <h6>Email</h6>
                                                    <p class="mb-0">{{ perusahaan.email ? perusahaan.email : '-' }}</p>
                                                    <h6>Phone (Office/Handphone)</h6>
                                                    <p class="mb-0">
                                                        {{ perusahaan.phone ? perusahaan.phone : '-' }} / 
                                                        {{ perusahaan.contact_hp ? perusahaan.contact_hp : '-' }}
                                                    </p>
                                                    <h6>Fax</h6>
                                                    <p class="mb-0">{{ perusahaan.fax ? perusahaan.fax : '-' }}</p>
                                                </div>
                                            </div>
                                            <div class="col-lg-4">
                                                <div class="media-support-info ml-3">
                                                    <h6>Addres</h6>
                                                    <p class="mb-0">
                                                        {{ perusahaan.address ? perusahaan.address : '-' }}
                                                    </p>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="iq-card-body pt-0 mt-4" style="padding: 15px;">
                                        <h6 class="card-title text-primary">Data Kapal</h6>
                                        <div class="row">
                                            <div class="col-md-4">
                                                <div class="training-block d-flex align-items-center">
                                                    <div class="rounded-circle iq-card-icon iq-bg-primary">
                                                        <i class="ri-ship-fill"></i>
                                                    </div>
                                                    <div class="ml-3">
                                                        <h5>Jumlah Kapal</h5>
                                                        <p class="mb-0">{{ kapalTotal.total }} armada</p>
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="col-md-4">
                                                <div class="training-block d-flex align-items-center">
                                                    <div class="rounded-circle iq-card-icon iq-bg-warning">
                                                        <i class="ri-ship-2-fill"></i>
                                                    </div>
                                                    <div class="ml-3">
                                                        <h5>Kapal On</h5>
                                                        <p class="mb-0">{{ kapalTotal.on }} armada</p>
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="col-md-4">
                                                <div class="training-block d-flex align-items-center">
                                                    <div class="rounded-circle iq-card-icon iq-bg-danger">
                                                        <i class="ri-ship-2-line"></i>
                                                    </div>
                                                    <div class="ml-3">
                                                        <h5>Kapal Off</h5>
                                                        <p class="mb-0">{{ kapalTotal.off }} armada</p>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-lg-3" v-if="kapal">
                    <div class="iq-card iq-card-block iq-card-stretch iq-card-height">
                        <div class="iq-card-header d-flex justify-content-between">
                            <div class="iq-header-title">
                                <h4 class="card-title">Kapal</h4>
                            </div>
                            <button class="btn btn-outline-primary btn-sm" data-toggle="modal" data-target="#form_kapal"
                                @click="selectedKapal = {} ;statusKapal = 'create'"
                            >
                                Tambah
                            </button>
                        </div>
                        <div class="iq-card-body pl-2 pr-0 pt-2 pb-0">
                            <div class="form-group ml-1 mr-2">
                                <input type="text" autocomplete="off" placeholder="Cari Nama Kapal" class="form-control" style="padding-right: 30px;">
                                <a href="javascript:void(0)" class="search-link mod">
                                    <i class="ri-search-line"></i>
                                </a>
                            </div>
                            <ul id="listkapal" class="doctors-lists m-0 p-0 iq-email-sender-list" style="height: 64vh;">
                                <li class="d-flex mb-2 align-items-center justify-content-center text-dark" v-if="!kapal || !kapal.length">
                                    Belum ada data kapal!
                                </li>
                                <li class="d-flex mb-2 align-items-center pr-1"
                                    v-for="(item, key) in kapal"
                                    :key="item.id"
                                >
                                    <div class="media-support-info ml-3">
                                        <h6 :class="{'text-danger': !item.timestamp || item.timestamp < item.tglNow}">{{ item.name }}</h6>
                                        <p class="mb-0 font-size-12">{{ item.sn }}</p>
                                    </div>
                                    <ul class="iq-social-media" style="display: contents; line-height: 0px;">
                                        <li>
                                            <a href="javascript:void(0)" data-toggle="modal" data-target="#form_kapal" 
                                                @click="editKapal(key); statusKapal = 'edit';"
                                            >
                                                <i class="ri-edit-box-line"></i>
                                            </a>
                                        </li>
                                        <li>
                                            <a href="javascript:void(0)" @click="formKapal = kapal[key]; deleteKapal();">
                                                <i class="ri-delete-bin-2-line"></i>
                                            </a>
                                        </li>
                                        <li>
                                            <a href="javascript:void(0)" data-toggle="modal" data-target="#detail_kapal" 
                                                @click="selectedKapal = kapal[key]">
                                                <i class="ri-file-list-2-line"></i>
                                            </a>
                                        </li>
                                    </ul>
                                </li>
                            </ul>
                        </div>
                    </div>
                </div>
                <!-- bila ada customer yang di-select - end -->
            </div>
        </div>
        <!-- modal section -->
        <!-- modal form customer - digunakan untuk create dan edit customer -->
        <div class="modal fade" id="form_customer" data-backdrop="static" tabindex="-1" role="dialog" aria-hidden="true">
            <div class="modal-dialog modal-lg" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" v-if="statusForm == 'create'">Tambah Customer</h5>
                        <h5 class="modal-title" v-if="statusForm == 'edit'">Edit Customer</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <div class="row">
                            <div class="col-md-6">
                                <div class="form-group">
                                    <label>Nama Customer</label>
                                    <input v-model="form.customer_name" autoconplete="false" type="text" placeholder="Masukkan nama customer" class="form-control"> 
                                </div>
                                <div class="form-group">
                                    <label>NPWP</label>
                                    <input v-model="form.npwp" autocomplete="false" type="text" placeholder="Masukkan nomor NPWP" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>Email</label>
                                    <input v-model="form.email" autocomplete="false" type="email" placeholder="Masukkan email" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>No Telephone</label>
                                    <input v-model="form.phone" autocomplete="false" type="text" placeholder="Masukkan nomor telepon" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>No Handphone</label>
                                    <input v-model="form.contact_hp" autocomplete="false" type="text" placeholder="Masukkan nomor handphone" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>Contact</label>
                                    <input v-model="form.contact" autocomplete="false" type="text" placeholder="Masukkan contact" class="form-control">
                                </div>
                            </div>
                            <div class="col-md-6">
                                <div class="form-group">
                                    <label>No Fax</label>
                                    <input v-model="form.fax" autocomplete="false" type="text" placeholder="Masukkan nomor fax" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>Agent</label>
                                    <input v-model="form.agent" autocomplete="false" type="text" placeholder="Masukkan nama agent" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>Provinsi</label>
                                    <input v-model="form.province" autocomplete="false" type="text" placeholder="Masukkan nama provinsi" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>Kota</label>
                                    <input v-model="form.city" autocomplete="false" type="text" placeholder="Masukkan nama kota" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>Postal Kode</label>
                                    <input v-model="form.postal_code" autocomplete="false" type="text" placeholder="Masukkan kode pos" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>Alamat</label>
                                    <textarea v-model="form.address" rows="2" cols="3" placeholder="Masukkan alamat" class="form-control"></textarea>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button type="button" class="btn btn-primary" tabindex="-1"
                            data-dismiss="modal" @click="createCustomer();"
                            v-if="statusForm == 'create'"
                        >
                            Simpan
                        </button>
                        <button type="button" class="btn btn-primary" tabindex="-1"
                            data-dismiss="modal" @click="updateCustomer()"
                            v-if="statusForm == 'edit'"
                        >
                            Edit
                        </button>
                    </div>
                </div>
            </div>
        </div>
        <!-- modal form akun -->
        <div class="modal fade" id="form_akun" data-backdrop="static" tabindex="-1" role="dialog" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" v-if="statusAkun == 'create'">Create Account Customer</h5>
                        <h5 class="modal-title" v-if="statusAkun == 'edit'">Edit Account Customer</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <div class="form-group">
                            <label>Email</label>
                            <input v-model="akun.email" type="email" autoccomplete="false" class="form-control" placeholder="Masukkan email">
                        </div>
                        <div class="form-group">
                            <label>Password</label>
                            <input v-model="akun.password" :type="showPass" autoccomplete="false" class="form-control" placeholder="Masukkan password">
                            <a href="javascript:void(0)" class="search-link setShow" 
                                @click="showPass = (showPass == 'password' ? 'test' : 'password');"
                            >
                                <i class="ri-eye-off-line"></i>
                            </a>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button type="button" class="btn btn-primary" data-dismiss="modal"
                            @click="statusAkun == 'create' ? createAkun() : updateAkun();"
                        >
                            Save changes
                        </button>
                    </div>
                </div>
            </div>
        </div>
        <!-- form kapal -->
        <div class="modal fade" id="form_kapal" data-backdrop="static" tabindex="-1" role="dialog" aria-hidden="true">
            <div class="modal-dialog modal-lg" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" v-if="statusKapal == 'create'">Tambah Kapal</h5>
                        <h5 class="modal-title" v-if="statusKapal == 'edit'">Edit Kapal</h5>
                    </div>
                    <div class="modal-body">
                        <div class="row">
                            <div class="col-lg-6">
                                <div class="form-group">
                                    <label>Vessel ID</label>
                                    <input v-model="formKapal.id" autocomplete="false" type="text" placeholder="Masukkan vessel ID" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>IMEI</label>
                                    <input v-model="formKapal.imei" autocomplete="false" type="text" placeholder="Masukkan nomor IMEI" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>Categori</label>
                                    <!-- <select v-model="formKapal.category_id" class="form-control">
                                        <option value="" selected disabled>Pilih kategori customer</option>
                                        <option v-for="item in categoryCustomer" :key="item.id" :value="item.id">{{ item.name }}</option>
                                    </select> -->
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
                                    <input v-model="formKapal.atp_start" autocomplete="false" type="date" placeholder="mm/dd/yyyy" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>ATP End</label>
                                    <input v-model="formKapal.atp_end" autocomplete="false" type="date" placeholder="mm/dd/yyyy" class="form-control">
                                </div>
                                <div class="form-group">
                                    <label>Device ID/SN</label>
                                    <input autocomplete="false" type="text" placeholder="Masukkan Device ID/SN" class="form-control"
                                        v-model="formKapal.sn" :disabled="statusKapal == 'edit' ? true : false">
                                    <button type="button" class="btn btn-outline-danger btn-sm mr-2 setcarisn" 
                                        @click="getDeviceID()" :disabled="statusKapal == 'edit' ? true : false"
                                    >
                                        Cari
                                    </button>
                                </div>
                                <div class="form-group">
                                    <label>Nama Kapal</label>
                                    <input autocomplete="false" type="text" placeholder="Masukkan nama kapal" class="form-control"
                                        v-model="formKapal.name" :disabled="statusKapal == 'create' ? true : false"
                                    >
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal" @click="formKapal = {}">Close</button>
                        <button type="button" class="btn btn-primary" data-dismiss="modal" @click="createKapal()" v-if="statusKapal == 'create'">Save Changes</button>
                        <button type="button" class="btn btn-primary" data-dismiss="modal" @click="updateKapal()" v-if="statusKapal == 'edit'">Save Changes</button>
                    </div>
                </div>
            </div>
        </div>
        <!-- detail kapal -->
        <div class="modal fade" id="detail_kapal" tabindex="-1" role="dialog" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">Detail Kapal {{ selectedKapal.name }}</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <div class="row">
                            <div class="col-lg-6">
                                <ul class="m-0 p-0">
                                    <li class="d-flex mb-4 align-items-center">
                                        <div class="media-support-info">
                                            <h6>IMEI Kapal</h6>
                                            <a href="javascript:void(0)">
                                                {{ selectedKapal.imei ? selectedKapal.imei : '-' }}
                                            </a>
                                        </div>
                                    </li>
                                    <li class="d-flex mb-4 align-items-center">
                                        <div class="media-support-info">
                                            <h6>Vessel ID</h6>
                                            <a href="javascript:void(0)">
                                                {{ selectedKapal.id ? selectedKapal.id : '-' }}
                                            </a>
                                        </div>
                                    </li>
                                    <li class="d-flex mb-4 align-items-center">
                                        <div class="media-support-info">
                                            <h6>Waktu Terakhir</h6>
                                            <a href="javascript:void(0)">
                                                {{ selectedKapal.last_update ? selectedKapal.last_update : '-' }}
                                            </a>
                                        </div>
                                    </li>
                                    <li class="d-flex mb-4 align-items-center">
                                        <div class="media-support-info">
                                            <h6>Speed</h6>
                                            <a href="javascript:void(0)">
                                                {{ selectedKapal.speed ? selectedKapal.speed : '-' }}
                                            </a>
                                        </div>
                                    </li>
                                </ul>
                            </div>
                            <div class="col-lg-6">
                                <ul class="m-0 p-0">
                                    <li class="d-flex mb-4 align-items-center">
                                        <div class="media-support-info">
                                            <h6>SN</h6>
                                            <a href="javascript:void(0)">
                                                {{ selectedKapal.sn ? selectedKapal.sn : '-' }}
                                            </a>
                                        </div>
                                    </li>
                                    <li class="d-flex mb-4 align-items-center">
                                        <div class="media-support-info">
                                            <h6>Status Kapal</h6>
                                            <a href="javascript:void(0)">active</a>
                                        </div>
                                    </li>
                                    <li class="d-flex mb-4 align-items-center">
                                        <div class="media-support-info">
                                            <h6>Latitude/Longitude</h6>
                                            <a href="javascript:void(0)">
                                                {{ (selectedKapal.lat ? selectedKapal.lat : '-') +' / '+ (selectedKapal.lon ? selectedKapal.lon : '-') }}
                                            </a>
                                        </div>
                                    </li>
                                    <li class="d-flex mb-4 align-items-center">
                                        <div class="media-support-info">
                                            <h6>Atp Start/End</h6>
                                            <a href="javascript:void(0)">
                                                {{ (selectedKapal.atp_start ? selectedKapal.atp_start : '-') +' / '+ (selectedKapal.atp_end ? selectedKapal.atp_end : '-') }}
                                            </a>
                                        </div>
                                    </li>
                                </ul>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import TopBar from './include/TopBar.vue';
import axios from 'axios';

export default {
    name: "Customer",
    components: {
        TopBar
    },
    directives: {},
    data() {
        return {
            customer: [],
            CnmCustomer: null,

            // form customer
            form: {},
            statusForm: 'create', // edit atau create

            // form akun
            akun: {
                // dinamic - based input form
                email: null,
                password: null,

                // kinda static :wink 😉
                level: 'user',
                status: 'aktif',
                id_customer: null,
                tgl_buat: this.getCurrentFormatedDate(),
            },
            statusAkun: 'create',

            // form kapal
            selectedKapal: {},
            formKapal: {},
            kategoriCustomer: [],
            tipeKapal: [],
            statusKapal: 'create',

            // multiselect options
            listTipeKapal: [],
            listTipeCustomer: [],

            // individual data
            perusahaan: null,
            kapal: null,
            kapalTotal: [],

            // attribute element / component
            liCustomerActive: null,
            showPass: 'password',
        };
    },
    beforeCreate: function () {
        if (this.$session.exists()) {
            if (this.$session.get("level") == null) {
                this.$router.push("/lacak/home");
            }else{
                if (this.$session.get('level') === 'root') {
                    this.$router.push("/customers");
                }
            }
        } else {
            this.$router.push("/");
        }
    },
    created() {
        if(this.$session.get('level') === 'root') {
            this.getCustomer();
        }
    },
    mounted() {
        this.idakun = this.$session.get("level");
    },
    methods: {
        // customer section
        async getCustomer() {
            try {
                this.showloadingBar();
                const customer = await axios.get(`https://track.kapalpintar.co.id/api/data_perusahaan`);
                const tipeKapal = await axios.get(`https://track.kapalpintar.co.id/api/tipe_kapal`);
                const categoryCustomer = await axios.get(`https://track.kapalpintar.co.id/api/kategori_customers`);

                this.customer = customer.data.results;
                this.listTipeKapal = tipeKapal.data.results;
                this.listTipeCustomer = categoryCustomer.data.results;
                this.closeloadingBar();
            } catch (error) {
                console.log(error);
            }
        },
        async searchCustomer() {
            try {
                const response = await axios.post('https://track.kapalpintar.co.id/api/data_perusahaan/search', {
                    search: this.CnmCustomer
                });
                this.customer = response.data;
            } catch (error) {
                console.log(error);
            }
        },
        async getSingleCustomer(id) {
            try {
                this.showloadingBar();

                if(id === this.liCustomerActive) {
                    this.perusahaan = this.kapal = this.liCustomerActive = null;

                    this.closeloadingBar();
                    return;
                }

                this.activeListCustomer(id);
                const perusahaan = await axios.get(`https://track.kapalpintar.co.id/api/perusahaan/${id}`);
                const akun = await axios.get(`https://track.kapalpintar.co.id/api/akuncustomer/${id}`);
                const kapal = await axios.get(`https://track.kapalpintar.co.id/api/kapal/${id}`);

                this.perusahaan = perusahaan.data.results[0];

                this.akun.email = null;
                if(akun.data.results[0]) this.akun.email = akun.data.results[0].email;

                this.kapal = kapal.data;
                this.kapalTotal = this.countKapal();

                this.closeloadingBar();
            }catch (err) {
                console.log(err);
            }
        },
        async createCustomer() {
            try {
                this.showloadingBar();
                await axios.post('https://track.kapalpintar.co.id/api/addcustomer', this.form);
                this.closeloadingBar();

                this.$swal.fire({
                    title: 'Berhasil',
                    text: 'Data customer berhasil ditambahkan',
                    type: 'success',
                    confirmButtonText: 'Ok'
                });
                this.CnmCustomer = this.form.customer_name;
                this.searchCustomer();

                this.form = {};
            } catch (err) {
                console.log(err);
            }
        },
        async editCustomer() {
            try {
                this.showloadingBar();
                this.statusForm = 'edit';
                const response = await axios.get(`https://track.kapalpintar.co.id/api/singlecustomer/${this.perusahaan.id}`);
                this.form = response.data.results[0];

                this.closeloadingBar();
            } catch(err) {
                this.closeloadingBar();
                console.log(err);
            }
        },
        async updateCustomer() {
            try {
                this.showloadingBar();
                await axios.put(`https://track.kapalpintar.co.id/api/editcustomer/${this.perusahaan.id}`, this.form);
                this.getSingleCustomer(this.form.id);
                this.closeloadingBar();

                this.$swal.fire({
                    title: 'Berhasil',
                    text: 'Data customer berhasil diedit',
                    type: 'success',
                    confirmButtonText: 'Ok'
                });

                this.form = {};
            } catch (err) {
                console.log(err);
            }
        },
        async deleteCustomer() {
            this.$swal.fire({
                title: 'Hapus data customer?',
                text: "Data customer akan dihapus secara permanen",
                type: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                confirmButtonText: 'Hapus'
            }).then(async (result) => {
                if (result.value) {
                    try {
                        this.showloadingBar()
                        await axios.delete(`https://track.kapalpintar.co.id/api/customer/${this.perusahaan.id}`, {
                            headers: {
                                'Access-Control-Allow-Origin': '*',
                            }
                        });
                        this.closeloadingBar();

                        this.CnmCustomer = this.perusahaan = this.akun.email = this.kapal = null;
                        this.kapalTotal = [];
                        this.getCustomer();
                        this.$swal.fire({
                            title: 'Berhasil',
                            text: 'Data customer berhasil dihapus',
                            type: 'success',
                            confirmButtonText: 'Ok'
                        });
                    } catch (err) {
                        this.closeloadingBar();
                        console.log(err);
                    }
                }
            });
        },

        // akun section
        async createAkun() {
            try {
                this.showloadingBar();
                this.akun.id_customer = this.perusahaan.id;
                await axios.post('https://track.kapalpintar.co.id/api/addakun', this.akun);
                this.akun.password = null;

                this.getSingleCustomer(this.akun.id_customer);
                this.$swal.fire({
                    title: 'Berhasil',
                    text: 'Data akun berhasil ditambahkan',
                    type: 'success',
                    confirmButtonText: 'Ok'
                });
            } catch(err) {
                this.closeloadingBar();
                console.log(err);
            }
        },
        async editAkun(id) {
            try {
                this.showloadingBar();

                const response = await axios.get(`https://track.kapalpintar.co.id/api/singleakun/${id}`);
                this.akun.password = response.data.results[0].password;

                this.closeloadingBar();
            } catch(err) {
                this.closeloadingBar();
                console.log(err)
            }
        },
        async updateAkun() {
            try {
                this.showloadingBar();
                this.akun.id_customer = this.perusahaan.id;
                await axios.put(`https://track.kapalpintar.co.id/api/editakun/${this.akun.id_customer}`, this.akun);
                this.akun.password = null;

                this.getSingleCustomer(this.akun.id_customer);
                this.$swal.fire({
                    title: 'Berhasil',
                    text: 'Data akun berhasil ditambahkan',
                    type: 'success',
                    confirmButtonText: 'Ok'
                });
            } catch(err) {
                this.closeloadingBar();
                console.log(err);
            }
        },

        // kapal section
        async getDeviceID() {
            try {
                this.showloadingBar();
                const response = await axios.get(`https://track.kapalpintar.co.id/api/getdeviceid/${this.formKapal.sn}`);
                if(!response.data.results.length) {
                    this.$swal.fire({
                        title: 'Device ID/SN yang dimasukan tidak ditemukan',
                        text: 'Harap cek kembali input yang dimasukkan!',
                        type: 'warning',
                        confirmButtonText: 'Ok'
                    });
                } else {
                    this.statusKapal = 'edit';
                    this.formKapal.name = response.data.results[0].name;
                    this.statusKapal = 'create';
                }

                this.closeloadingBar();
            } catch(err) {
                console.log(err);
                this.closeloadingBar();
            }
        },
        async createKapal() {
            try {
                this.showloadingBar();

                this.formKapal.tipeForm = "Tambah";
                console.log(this.formKapal);
                if(this.formKapal.sn) {
                    const response = await axios.put(`https://track.kapalpintar.co.id/api/updatepemilikkapal/${this.formKapal.sn}`, this.formKapal);

                    this.closeloadingBar();
                    this.getSingleCustomer(this.perusahaan.id);
                    if(response.data.status == 1) {
                        this.$swal.fire({
                            title: 'Berhasil',
                            text: 'Data kapal berhasil ditambahkan',
                            type: 'success',
                            confirmButtonText: 'Ok'
                        });
                    } else if(response.data.status == 2) {
                        this.$swal.fire({
                            title: 'Gagal',
                            text: response.data.msg,
                            type: 'danger',
                            confirmButtonText: 'Ok'
                        });
                    } else {
                        this.$swal.fire({
                            title: 'Gagal',
                            text: 'Terjadi kesalahan, silahkan coba lagi',
                            type: 'danger',
                            confirmButtonText: 'Ok'
                        });
                        setTimeout(() => {
                            window.location.reload();
                        }, 3000);
                    }
                } else {
                    this.$swal.fire({
                        title: 'Device ID/SN yang dimasukan tidak ditemukan',
                        text: 'Harap cek kembali input yang dimasukkan!',
                        type: 'warning',
                        confirmButtonText: 'Ok'
                    });
                }
            } catch(err) {
                this.closeloadingBar();
                console.log(err);
            }
        },
        async editKapal(key) {
            try {
                this.showloadingBar();

                this.formKapal = this.kapal[key];
                this.formKapal.atp_start = this.kapal[key].atp_startYMD;
                this.formKapal.atp_end = this.kapal[key].atp_endYMD;
    
                if(this.formKapal.category_id) {
                    const kategori_customer = await axios.get(`https://track.kapalpintar.co.id/api/kategori_customers/${this.formKapal.category_id}`);
                    this.kategoriCustomer = kategori_customer.data.results[0];
                }
                if(this.formKapal.type_id) {
                    const tipe_kapal = await axios.get(`https://track.kapalpintar.co.id/api/tipe_kapal/${this.formKapal.type_id}`);
                    this.tipeKapal = tipe_kapal.data.results[0];
                }

                this.closeloadingBar();
            } catch(err) {
                this.closeloadingBar();
                console.log(err)
            }
        },
        async updateKapal() {
            try {
                this.showloadingBar();
                this.formKapal.tipeForm = 'Edit';
                this.formKapal.customer_id = this.kategoriCustomer ? this.kategoriCustomer : null;
                this.formKapal.type_id = this.tipeKapal ? this.tipeKapal : null;

                await axios.put(`https://track.kapalpintar.co.id/api/updatepemilikkapal/${this.formKapal.sn}`, this.formKapal);

                this.formKapal = {};
                await this.getSingleCustomer(this.perusahaan.id);
                this.$swal.fire({
                    title: 'Berhasil',
                    text: 'Data kapal berhasil diedit',
                    type: 'success',
                    confirmButtonText: 'Ok'
                });
            } catch(err) {
                this.closeloadingBar();
                console.log(err);
            }
        },
        async deleteKapal() {
            this.$swal.fire({
                title: 'Hapus data kapal',
                text: 'Apakah anda yakin ingin menghapus data kapal ini?',
                type: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                confirmButtonText: 'Ya, hapus!'
            }).then(async (result) => {
                if(result.value) {
                    try {
                        this.showloadingBar();
                        this.formKapal.tipeForm = 'Delete';
                        this.formKapal.id_customer = null;

                        await axios.put(`https://track.kapalpintar.co.id/api/updatepemilikkapal/${this.formKapal.sn}`, this.formKapal);
                        this.formKapal = {};
                        await this.getSingleCustomer(this.perusahaan.id);
                        this.$swal.fire({
                            title: 'Berhasil',
                            text: 'Data kapal berhasil dihapus',
                            type: 'success',
                            confirmButtonText: 'Ok'
                        });
                    } catch (err) {
                        this.closeloadingBar();
                        console.log(err);
                    }
                }
            })
        },

        // trait
        activeListCustomer(id) {
            this.liCustomerActive = id ? id : null;
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
        countKapal() {
            let result = {total: 0, on: 0, off: 0};

            for (const key in this.kapal) {
                if (Object.hasOwnProperty.call(this.kapal, key)) {
                    const e = this.kapal[key];

                    if(e.timestamp >= e.tglNow) {
                        result.on++
                    } else {
                        result.off++
                    }

                    result.total++;
                }
            }

            return result;
        },
        getCurrentFormatedDate() {
            const d = new Date();
            let d_arr = d.toLocaleDateString('id-ID').split('/');

            return d_arr[2] +'-'+ d_arr[1] +'-'+ d_arr[0]; // YYYY-m-d
        }
    }
}
</script>

<style scoped>
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
.mod {
    position: absolute;
    right: 30px;
    top: 70px;
    font-size: 15pt;
}
.setShow {
    position: absolute;
    right: 7%;
    bottom: 15%;
    font-size: 25px;
}
.media-support-info {
    flex: 1 1 0%;
}
.setcarisn {
    position: absolute;
    right: 3%;
    bottom: 31.7%;
}
.mb-m10 {
    margin-bottom: -10px;
}
.set-list {
    padding: 10px 0 10px 10px;
    border-bottom: 1px solid #efefef;
    border-bottom-style: dashed;
}
</style>