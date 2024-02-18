<template>
  <div class="container">
    <header class="d-flex justify-content-center py-3">
      <ul class="nav nav-pills">
        <li class="nav-item">
          <a href="#" class="nav-link active" aria-current="page">List</a>
        </li>
        <li class="nav-item"><a href="#" data-bs-toggle="modal" data-bs-target="#addModal" class="nav-link">Add Data</a></li>
      </ul>
    </header>

    <body>
      <table class="table">
        <thead>
          <tr>
            <th scope="col" class="text-center">No</th>
            <th scope="col" class="text-center">Nama</th>
            <th scope="col" class="text-center">Jurusan</th>
            <th scope="col" class="text-center">Aksi</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(dataMhs, index) in dataMahasiswa" :key="index">
            <th class="text-center" scope="row">{{ index+1 }}</th>
            <td class="text-center">{{ dataMhs['nama'] }}</td>
            <td class="text-center">{{ dataMhs['jurusan'] }}</td>
            <td class="text-center">
              <button type="button" class="btn btn-primary me-2" @click="getData(dataMhs)">
                Edit
              </button>

              <button type="button" class="btn btn-danger" @click="hapusData(dataMhs['id'])">Hapus</button>
            </td>
          </tr>
        </tbody>
      </table>
    </body>

    <!-- Modal Add-->
    <div class="modal fade" id="addModal" data-bs-backdrop="static" data-bs-keyboard="false" tabindex="-1" aria-labelledby="staticBackdropLabel" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content">

          <div class="modal-header">
            <h1 class="modal-title fs-5" id="staticBackdropLabel">Input Data Mahasiswa</h1>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          
          <div class="modal-body">
            <form @submit.prevent="submitForm">

              <div class="mb-3">
                <label class="form-label">Nama</label>
                <input type="text" class="form-control" v-model="inp_nama_mhs">
              </div>
              
              <div class="mb-3">
                <label class="form-label">Jurusan</label>
                <input type="text" class="form-control" v-model="inp_jurusan_mhs" >
              </div>

              <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal" id="btnClose">Close</button>
                <button type="submit" class="btn btn-primary" :disabled="inp_nama_mhs == '' || inp_jurusan_mhs == ''">Submit Data</button>
              </div>

            </form>
          </div>
          
        </div>
      </div>
    </div>
    <!-- Modal Add End -->

    <!-- Modal Edit-->
    <button data-bs-toggle="modal" data-bs-target="#editModal" id="showBtnEdit" style="display: none;"></button>
    <div class="modal fade" id="editModal" data-bs-backdrop="static" data-bs-keyboard="false" tabindex="-1" aria-labelledby="staticBackdropLabel" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content">

          <div class="modal-header">
            <h1 class="modal-title fs-5" id="staticBackdropLabel">Edit Data Mahasiswa</h1>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          
          <div class="modal-body">
            <form @submit.prevent="editForm">

              <div class="mb-3">
                <label class="form-label">Nama</label>
                <input type="text" class="form-control" v-model="inp_edit_id_mhs" style="display: none;">
                <input type="text" class="form-control" v-model="inp_edit_nama_mhs">
              </div>
              
              <div class="mb-3">
                <label class="form-label">Jurusan</label>
                <input type="text" class="form-control" v-model="inp_edit_jurusan_mhs" >
              </div>

              <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal" id="btnEditClose">Close</button>
                <button type="submit" class="btn btn-primary" :disabled="inp_edit_nama_mhs == '' || inp_edit_jurusan_mhs == ''">Edit Data</button>
              </div>

            </form>
          </div>
          
        </div>
      </div>
    </div>
    <!-- Modal Edit End -->

  </div>
</template>

<script setup lang="ts">
  import axios from 'axios'
  import { onMounted, ref } from 'vue'
  import $ from 'jquery'
  import Swal from 'sweetalert2'
 
  const url = 'http://localhost:3000/mhs';

  const dataMahasiswa = ref([]);

  const inp_nama_mhs    = ref("");
  const inp_jurusan_mhs = ref("");

  const inp_edit_nama_mhs    = ref("");
  const inp_edit_jurusan_mhs = ref("");
  const inp_edit_id_mhs      = ref("");

  const getMhsAll = async () => {
    try {
      const response      = await axios.get(url+'/all');
      dataMahasiswa.value = response.data.data;
    } catch (error) {
      console.error(error);
    }
  }

  const clearInput = () => {
    inp_nama_mhs.value    = "";
    inp_jurusan_mhs.value = "";

    inp_edit_id_mhs.value      = "";
    inp_edit_nama_mhs.value    = "";
    inp_edit_jurusan_mhs.value = "";
  }

  const submitForm = async () => {
    await axios.post(url+'/insert', {
      nama: inp_nama_mhs.value,
      jurusan: inp_jurusan_mhs.value
    })
    .then(function (response) {
      // console.log(response);
      clearInput();
      getMhsAll();
      $("#btnClose").trigger("click");
    })
    .catch(function (error) {
      console.log(error);
    });
  }

  const hapusData = (id :string) => {
    Swal.fire({
      title: "Apakah Ingin Menghapus Mahasiswa Ini?",
      showDenyButton: true,
      showCancelButton: false,
      confirmButtonText: "Ya!",
      denyButtonText: "Tidak"
    }).then(async (result) => {
      /* Read more about isConfirmed, isDenied below */
      if (result.isConfirmed) {
        await axios.post(url+'/delete', {
          id: id,
        })
        .then(function (response) {
          Swal.fire("Owkayy!", "Berhasil Dihapus", "success");
          getMhsAll();
        })
        .catch(function (error) {
          Swal.fire("Whoops!", "Ada yang error nih.", "error");
        });
      } else if (result.isDenied) {
        Swal.fire("Owkayy!", "Tidak Jadi Dihapus", "info");
      }
    });
  }

  const getData = async (objMhs :any) => {
    $("#showBtnEdit").trigger("click");
    inp_edit_id_mhs.value         = objMhs.id;
    inp_edit_nama_mhs.value       = objMhs.nama;
    inp_edit_jurusan_mhs.value    = objMhs.jurusan;
  }

  const editForm = async () => {
    await axios.post(url+'/update', {
      id: inp_edit_id_mhs.value,
      nama_upd: inp_edit_nama_mhs.value,
      jurusan_upd: inp_edit_jurusan_mhs.value
    })
    .then(function (response) {
      clearInput();
      getMhsAll();
      $("#btnEditClose").trigger("click");
    })
    .catch(function (error) {
      console.log(error);
    });
  }

  onMounted(() => {
    getMhsAll();
  })
</script>