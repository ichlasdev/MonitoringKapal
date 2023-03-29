<template>
  <section class="sign-in-page">
    <div class="container sign-in-page-bg mt-5 p-0">
      <div class="row no-gutters">
        <div class="col-md-6 text-center">
          <div class="sign-in-detail text-white">
            <a class="sign-in-logo mb-3" href="#">
              <h1 class="text-white">Lacak Kapal</h1>
              <h4 class="text-white">Sistem Lacak Kapal</h4>
            </a>
            <img src="images/tacking_kapal.png" class="img" alt="" srcset="" />
          </div>
        </div>
        <div class="col-md-6 position-relative">
          <div class="sign-in-from">
            <h1 class="mb-0">Sign in</h1>
            <p>
              Enter your username address and password to access lacak
              monitoring.
            </p>

            <div class="mt-4 form-group">
              <label for="exampleInputEmail1">Username</label>
              <input
                type="email"
                class="form-control mb-0"
                v-model="email"
                placeholder="Username"
              />
            </div>
            <div class="form-group">
              <label for="exampleInputPassword1">Password</label>
              <!-- <a href="#" class="float-right">Forgot password?</a> -->
              <input
                type="password"
                class="form-control mb-0"
                v-model="password"
                placeholder="Password"
              />
            </div>
            <div class="d-inline-block w-100">
              <div
                class="custom-control custom-checkbox d-inline-block mt-2 pt-1"
              >
                <input
                  type="checkbox"
                  class="custom-control-input"
                  id="customCheck1"
                />
                <label class="custom-control-label" for="customCheck1"
                  >Remember Me</label
                >
              </div>
              <button
                type="button"
                class="btn btn-primary float-right"
                @click="sign"
              >
                Sign in
              </button>
            </div>
            <div class="sign-info">
              <span class="dark-color d-inline-block line-height-2">
                <a
                  href="/"
                  class="btn btn-warning"
                >
                  <b>Go To Sistem Monitoring</b>
                </a>
              </span>
              <ul class="iq-social-media">
                <li>
                  <a href="javascript:void(0)"
                    ><i class="ri-facebook-box-line"></i
                  ></a>
                </li>
                <li>
                  <a href="javascript:void(0)"
                    ><i class="ri-twitter-line"></i
                  ></a>
                </li>
                <li>
                  <a
                    href="https://www.instagram.com/kapalpintar/"
                    target="_blank"
                    ><i class="ri-instagram-line"></i
                  ></a>
                </li>
              </ul>
              
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import axios from "axios";
import { required, email } from "vuelidate/lib/validators";

export default {
  name: "Login",
  components: {},
  data() {
    return {
      email: null,
      password: null,
    };
  },
  beforeCreate: function () {
    if (this.$session.exists()) {
      if (this.$session.get("level") == null) {
        this.$router.push("/lacak/home");
      }else{
        this.$router.push("/perusahaan");
      }
    } else {
      this.$router.push("/lacak");
    }
  },
  created() {},
  mounted() {},
  methods: {
    async sign() {
      this.$swal.fire({
        html: "<img src='images/loading-bar.gif' style='width: 50px;'/><p>Loading...</p>",
        showConfirmButton: false,
      });
      try {
        const response = await axios.post(
          "https://track.kapalpintar.co.id/api/lacak/login",
          {
            email: this.email,
            password: this.password,
          }
        );
       
        if (response.data["status"] === 0) {
          this.$swal.fire({
            icon: "error",
            title: "Oops...",
            text: response.data["msg"],
          });
        } else if (response.data["status"] === 1) {
          this.$session.start();
          this.$session.set("id_customer", response.data["results"][0]["id_customer"]);
          this.$session.set("username", response.data["results"][0]["username"]);
          
          this.$router.go();
        }
      } catch (error) {
        console.log(error);
      }
    },
  },
  validations: {
    email: {
      required,
      email,
    },
  },
};
</script>

<style scoped>
.img {
  height: 400px;
}
.btn-blue-muda {
  background: #2b2871;
}
</style>
