<template>
  <div>
    <div class="container-scroller">
      <div class="container-fluid page-body-wrapper full-page-wrapper">
        <div class="login">
          <div class="content-wrapper d-flex align-items-center auth">
            <div class="row w-100">
              <div class="col-lg-4 mx-auto">
                <div
                  class="auth-form-light p-4"
                  style="
                    border-radius: 3%;
                    background-color: #d8000f;
                    margin-left: -15%;
                    margin-right: 20%;
                    text-align: center;
                  "
                >
                  <div class="navbar-brand brand-logo">
                    <img src="../../public/yh.png" style="width: 150px" />
                  </div>
                  <!-- <h4 class="text-warning">Selamat datang!</h4>
                  <h6 class="font-weight-light">
                    Silahkan login terlebih dahulu
                  </h6> -->
                  <form v-on:submit.prevent="Login">
                    <b-form-group class="text-white text-center" id="lbl_username" label="Username" label-for="input_username">
                      <b-form-input id="input_username" v-model="username" placeholder="Alamat username" trim></b-form-input>
                    </b-form-group>

                    <b-form-group class="text-white text-center" id="lbl_password" label="Password" label-for="input_password">
                      <b-form-input type="password" id="input_password" v-model="password" placeholder="Kata sandi" trim></b-form-input>
                    </b-form-group>

                    <b-button block type="submit" class="btn btn-warning btn-sm">Login </b-button>
                  </form>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- content-wrapper ends -->
      </div>
      <b-toast id="loadingToast" title="Processing Data" no-auto-hide>
        <b-spinner label="Spinning" variant="secondary"></b-spinner>
        <strong class="text-secondary">Loading...</strong>
      </b-toast>
      <!-- toast untuk tampilan message box -->
      <b-toast id="message" title="Message">
        <strong class="text-success">{{ message }}</strong>
      </b-toast>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      id: '',
      nik: '',
      nama: '',
      telp: '',
      username: '',
      password: '',
      message: '',
    };
  },
  methods: {
    Login: function() {
      this.$bvToast.show('loadingToast');
      let username = this.username;
      let password = this.password;
      this.$store
        .dispatch('login', { username, password })
        .then((response) => {
          if (response.data.success) {
            localStorage.setItem('nama', response.data.data.user.nama);
            this.$bvToast.hide('loadingToast');
            this.message = response.data.message;
            this.$bvToast.show('message');
            this.$router.push('/petugas');
          } else {
            this.$bvToast.hide('loadingToast');
            this.message = 'email atau password anda salah';
            this.$bvToast.show('message');
            this.$router.push({ name: 'login' });
          }
        })
        .catch((err) => console.log(err));
    },

    Add: function() {
      this.action = 'insert';
      this.nik = '';
      this.nama = '';
      this.username = '';
      this.password = '';
      this.telp = '';
      this.level = '';
    },

    Save: function() {
      this.$bvToast.show('loadingToast');
      if (this.action === 'insert') {
        let form = new FormData();
        form.append('id', this.id);
        form.append('nik', this.nik);
        form.append('nama', this.nama);
        form.append('username', this.username);
        form.append('password', this.password);
        form.append('telp', this.telp);

        this.axios
          .post('/registerPetugas', form)
          .then((response) => {
            this.$bvToast.hide('loadingToast');
            this.$router.push('/login');
            this.message = response.data.message;
            this.$bvToast.show('message');
          })
          .catch((error) => {
            console.log(error);
          });
      } else {
        let form = {
          nik: this.nik,
          nama: this.nama,
          username: this.username,
          password: this.password,
          telp: this.telp,
        };
        this.axios
          .put('/login' + this.id, form)
          .then((response) => {
            this.$bvToast.hide('loadingToast');
            if (this.search == '') {
              this.getData();
            } else {
              this.searching();
            }
            this.message = response.data.message;
            this.$bvToast.show('message');
          })
          .catch((error) => {
            console.log(error);
          });
      }
    },
  },
};
</script>
