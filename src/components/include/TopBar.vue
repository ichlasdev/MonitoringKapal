<template>
  <div>
    <div class="iq-top-navbar header-top-sticky" style="width: calc(100% - 60px);">
      <div class="iq-navbar-custom">
        <div class="iq-sidebar-logo">
          <div class="top-logo">
            <a href="index.html" class="logo">
              <img src="images/logo.png" class="img-fluid" alt="" />
              <span>XRay</span>
            </a>
          </div>
        </div>
        <nav class="navbar navbar-expand-lg navbar-light p-0">
          <div class="collapse navbar-collapse" id="navbarSupportedContent">
            <div class="row">
              <div class="col-lg-6">
                <img src="images/kapalpintar.jpeg" class="ml-3" style="width: 200px;">
              </div>
              <div class="col-lg-6 text-right">
                <h4 class="mt-3">{{nmPerusahaan}}</h4>
              </div>
            </div>
            <div class="navbar-nav ml-auto navbar-list">
              <li class="nav-item">
                <a href="/perusahaan" data-toggle="tooltip" data-placement="bottom" 
                  title="Perusahaan Page" style="font-size: 30px;"
                  :class="{
                    'link-active': activeMenu === 'perusahaan',
                    'text-dark': activeMenu !== 'perusahaan',
                  }"
                  class="link iq-waves-effect border-style">
                  <i class="ri-community-fill icon-menu-home"></i>
                </a>
              </li>
              <li class="nav-item" v-if="this.$session.get('level') == 'root'">
                <a href="/customers" title="Customer Page"
                  :class="{
                    'link-active': activeMenu === 'customer',
                    'text-dark': activeMenu !== 'customer',
                  }"
                  class="link iq-waves-effect text-dark"
                >
                  <i class="ri-folder-user-line icon-menu"></i> Customers
                </a>
              </li>
              <li class="nav-item iq-full-screen">
                <a href="#" class="iq-waves-effect" id="btnFullscreen">
                  <i class="ri-fullscreen-line"></i>
                </a>
              </li>
            </div>
          </div>
          <ul class="navbar-list">
            <li>
              <a
                href="#"
                class="search-toggle iq-waves-effect d-flex align-items-center"
              >
                <img
                  src="images/user/01.jpg"
                  class="img-fluid rounded mr-3"
                  alt="user"
                />
                <div class="caption">
                  <h6 class="mb-0 line-height">Account</h6>
                  <span class="font-size-12">Available</span>
                </div>
              </a>
              <div class="iq-sub-dropdown iq-user-dropdown">
                <div class="iq-card shadow-none m-0">
                  <div class="iq-card-body p-0 ">
                    <div class="bg-primary p-3">
                      <h5 class="mb-0 text-white line-height">{{this.$session.get('email')}}</h5>
                      <span class="text-white font-size-12">Available</span>
                    </div>
                    <a href="javascript:void(0)" data-toggle="modal" data-target="#my-akun" class="iq-sub-card iq-bg-primary-hover">
                      <div class="media align-items-center">
                        <div class="rounded iq-card-icon iq-bg-primary">
                          <i class="ri-file-user-line"></i>
                        </div>
                        <div class="media-body ml-3">
                          <h6 class="mb-0 ">My Profile</h6>
                          <p class="mb-0 font-size-12">View personal profile details.</p>
                        </div>
                      </div>
                    </a>
                    <div class="d-inline-block w-100 text-center p-3">
                      <a
                        class="bg-primary iq-sign-btn"
                        href="javascript:void(0)"
                        role="button"
                        @click="signout"
                        >Sign out<i class="ri-login-box-line ml-2"></i
                      ></a>
                    </div>
                  </div>
                </div>
              </div>
            </li>
          </ul>
        </nav>
      </div>
    </div>
    <div class="modal fade" id="my-akun" tabindex="-1" role="dialog" aria-hidden="true">
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Edit My Account</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <div class="form-group">
              <label for="exampleInputEmail1">Email</label>
              <input type="email" class="form-control" v-model="email" autocomplete="false" placeholder="Masukkan Email">
            </div>
            <div class="form-group">
              <label for="exampleInputPassword1">Password</label>
              <input :type="showPass" class="form-control" v-model="password" placeholder="Masukkan Password">
              <a href="javascript:void(0)" class="search-link mod setShow" @click="togglePass()">
                <i class="ri-eye-off-line"></i>
              </a>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
            <button type="button" class="btn btn-primary" data-dismiss="modal" @click="editMyProfile()">Save changes</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: "TopBar",
  components: {},
  data() {
    return {
      activeMenu: null,
      showPass: 'password',
      nmPerusahaan: null,
      email: null,
      password: null,
    };
  },
  created() {
    this.setTopBar()
  },
  methods: {
    async setTopBar(){
        try {
          if(this.$session.get('level')==='root'){
            this.nmPerusahaan = this.$session.get('nmperusahaan')
            this.email = this.$session.get('email')
          }else{
            const response = await axios.get(`https://track.kapalpintar.co.id/api/perusahaan/${this.$session.get('id')}`);
            this.nmPerusahaan = response.data['results'][0]['customer_name'];
            this.email = response.data['results'][0]['customer_name'];
          }
        } catch (error) {
          // console.log(error);
        }

        if(this.$route.path === '/perusahaan') {
          this.activeMenu = 'perusahaan'
        } else if(this.$route.path === '/customers') {
          this.activeMenu = 'customer'
        }
    },
    async editMyProfile() {
      if(this.email === null || this.password === null){
        this.$toast.open({
          message: 'Email dan Password tidak boleh kosong',
          type: 'error',
          position: 'top-right',
          duration: 5000
        });
      }else{
        try {
          await axios.put(`https://track.kapalpintar.co.id/api/editmyakun/${this.$session.get('id')}`, {
            email: this.email,
            password: this.password
          });
          this.$session.set('email', this.email)
          this.$swal.fire({
            title: 'Success',
            text: 'Data berhasil diubah',
            type: 'success',
            timer: 1500
          });
          this.setTopBar()
        } catch (error) {
          // console.log(error);
        }
      }
    },
    async signout(){
      this.$swal.fire({
        text: 'Apakah yakin ingin keluar?',
        showDenyButton: true,
        confirmButtonText: `Ya`,
        denyButtonText: `Tidak`,
      }).then((result) => {
        if (result.isConfirmed) {
          this.$session.clear()
          this.$session.destroy()
          this.$router.go('/')
        } 
      });
    },
    togglePass() {
      if (this.showPass === 'password') {
        this.showPass = 'text';
      } else {
        this.showPass = 'password';
      }
    }
  },
};
</script>

<style>
.link-active {
    color: #0c9cac!important;
}
@media (max-width: 1299px){
  .iq-top-navbar, body.sidebar-main .iq-top-navbar {
    width: calc(100% - 30px) !important;
    margin: 20px 15px;
  }
}
.border-style {
  border-right: 1px solid #213567;
  border-right-style: dashed;
}
.setShow {
  right: 7%;
  position: absolute;
  bottom: 15%;
  font-size: 25px;
}
</style>
