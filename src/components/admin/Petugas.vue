<template>
  <div>
    <div class="container mt-3">
      <div class="row">
        <div class="col-lg-12 grid-margin stretch-card">
          <div class="card">
            <div class="card-body">
              <p class="card-title float-left"><b>Data Petugas</b></p>
              <p class="card-description float-right">
                <b-button class="btn btn-warning btn-sm" v-b-modal.modalTatib v-on:click="Add"><i class="mdi mdi-plus btn-icon-prepend"></i> Add</b-button>
              </p>
              <div class="table-responsive">
                <b-table hover :items="user" :fields="fields">
                  <template v-slot:cell(level)="data">
                    <b-badge variant="warning">{{ data.item.level }}</b-badge>
                  </template>
                  <template v-slot:cell(action)="data">
                    <button type="button" class="btn btn-link text-secondary" v-b-modal.modalTatib v-on:click="Edit(data.item)"><i class="mdi mdi-pencil"></i></button>
                    <button type="button" class="btn btn-link text-secondary" @click="Drop(data.item.id)"><i class="mdi mdi-delete"></i></button>
                  </template>
                </b-table>
                <b-pagination v-model="currentPage" :per-page="perPage" :total-rows="rows" align="center" v-on:input="pagination"> </b-pagination>

                <b-toast id="loadingToast" title="Processing Data" no-auto-hide>
                  <b-spinner label="Spinning" variant="secondary"></b-spinner>
                  <strong class="text-secondary"> Loading...</strong>
                </b-toast>

                <!-- toast untuk tampilan message box -->
                <b-toast id="message" title="Message">
                  <strong class="text-success">{{ message }}</strong>
                </b-toast>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <b-modal id="modalTatib" @ok="Save">
      <template v-slot:modal-title> Form Masyarakat </template>
      <form ref="form">
        <div class="form-group">
          <label for="nik" class="col-form-label">NIK</label>
          <input type="number" name="nik" class="form-control" id="nik" placeholder="nik" v-model="nik" />
        </div>
        <div class="form-group">
          <label for="nama" class="col-form-label">Nama Petugas</label>
          <input type="text" name="nama" class="form-control" id="nama" placeholder="name" v-model="nama" />
        </div>
        <div class="form-group">
          <label for="telp" class="col-form-label">Telepon</label>
          <input type="number" name="telp" class="form-control" id="telp" placeholder="telepon" v-model="telp" />
        </div>
        <div class="form-group">
          <label for="username" class="col-form-label">Username</label>
          <input type="text" name="username" class="form-control" id="username" placeholder="username" v-model="username" />
        </div>
        <div class="form-group">
          <label for="password" class="col-form-label">Password</label>
          <input type="password" name="password" class="form-control" id="password" placeholder="Kata Sandi" v-model="password" />
        </div>
      </form>
    </b-modal>
  </div>
</template>

<script>
module.exports = {
  data: function() {
    return {
      search: '',
      id: '',
      nik: '',
      nama: '',
      telp: '',
      username: '',
      password: '',
      level: '',
      action: '',
      message: '',
      currentPage: 1,
      rows: 0,
      perPage: 10,
      key: '',
      user: [],
      fields: ['id', 'nik', 'nama', 'telp', 'username', 'level', 'action'],
    };
  },

  methods: {
    getData: function() {
      let conf = { headers: { Authorization: 'Bearer ' + this.key } };
      let offset = (this.currentPage - 1) * this.perPage;
      this.$bvToast.show('loadingToast');
      this.axios
        .get('/petugas/' + this.perPage + '/' + offset, conf)
        .then((response) => {
          if (response.data.success) {
            this.$bvToast.hide('loadingToast');
            this.user = response.data.data.user;
            this.rows = response.data.data.count;
          } else {
            this.$bvToast.hide('loadingToast');
            this.message = 'Gagal menampilkan data petugas.';
            this.$bvToast.show('message');
            // this.$router.push({name: "login"})
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },

    pagination: function() {
      if (this.search == '') {
        this.getData();
      } else {
        this.searching();
      }
    },

    Add: function() {
      this.action = 'insert';
      this.nik = '';
      this.nama = '';
      this.telp = '';
      this.username = '';
      this.password = '';
      this.level = '';
    },

    Edit: function(item) {
      this.action = 'update';
      this.id = item.id;
      this.nik = item.nik;
      this.nama = item.nama;
      this.telp = item.telp;
      this.username = item.username;
      this.password = item.password;
      this.level = item.level;
    },

    Save: function() {
      let conf = { headers: { Authorization: 'Bearer ' + this.key } };
      this.$bvToast.show('loadingToast');
      if (this.action === 'insert') {
        let form = new FormData();
        form.append('id', this.id);
        form.append('nik', this.nik);
        form.append('nama', this.nama);
        form.append('username', this.username);
        form.append('password', this.password);
        form.append('telp', this.telp);
        form.append('level', this.level);

        this.axios
          .post('/petugas', form, conf)
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
      } else {
        let form = {
          nik: this.nik,
          nama: this.nama,
          telp: this.telp,
          username: this.username,
          password: this.password,
          level: this.level,
        };
        this.axios
          .put('/petugas/' + this.id, form, conf)
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
    Drop(id) {
      let conf = { headers: { Authorization: 'Bearer ' + this.key } };
      if (confirm('Apakah anda yakin ingin menghapus data ini?')) {
        this.$bvToast.show('loadingToast');
        this.axios
          .delete('/petugas/' + id, conf)
          .then((response) => {
            this.getData();
            this.$bvToast.hide('loadingToast');
            this.message = response.data.message;
            this.$bvToast.show('message');
          })
          .catch((error) => {
            console.log(error);
          });
      }
    },
  },
  mounted() {
    this.key = localStorage.getItem('Authorization');
    this.getData();
  },
};
</script>
