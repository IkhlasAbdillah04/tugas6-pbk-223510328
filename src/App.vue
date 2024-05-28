<template>
  <div class="app-container">
    <div class="container">
      <form @submit.prevent="save" class="product-form">
        <input type="text" v-model="form.nama" placeholder="Nama Produk" />
        <select v-model="form.kategori">
          <option disabled value="">Pilih Kategori</option>
          <option v-for="kategori in kategoriList" :key="kategori" :value="kategori">{{ kategori }}</option>
        </select>
        <div class="form-buttons">
          <button type="button" @click="load" class="load-button">Load</button>
          <button type="submit" class="save-button">Save</button>
        </div>
      </form>

      <div class="table-wrapper">
        <table class="product-table">
          <thead>
            <tr>
              <th>Nama</th>
              <th>Kategori</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="produk in produkList" :key="produk.id">
              <td>{{ produk.nama }}</td>
              <td>{{ produk.kategori }}</td>
              <td>
                <button @click="edit(produk)" class="edit-button">Edit</button>
                <button @click="del(produk.id)" class="delete-button">Delete</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      nama: '',
      kategori: '',
    });

    const produkList = ref([]);
    const kategoriList = ref(['Elektronik', 'Pakaian', 'Perabotan', 'Buku', 'Makanan', 'Minuman']);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/produk');
        produkList.value = response.data.map(produk => ({
          ...produk,
          id: parseInt(produk.id) // Pastikan ID adalah angka
        }));
      } catch (error) {
        console.error('Error loading products', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/produk/${form.id}`
          : 'http://localhost:3000/produk';
        const method = form.id ? 'put' : 'post';

        if (!form.id) {
          // Menentukan ID baru
          const maxId = produkList.value.reduce((max, produk) => Math.max(max, produk.id), 0);
          form.id = (maxId + 1).toString();
        }

        const response = await axios[method](url, form);

        if (form.id && method === 'put') {
          produkList.value = produkList.value.map(produk =>
            produk.id === parseInt(response.data.id) ? response.data : produk
          );
        } else {
          produkList.value.push({
            ...response.data,
            id: parseInt(response.data.id)
          });
        }

        form.id = null;
        form.nama = '';
        form.kategori = '';
      } catch (error) {
        console.error('Error saving product', error);
      }
    }

    async function del(id) {
      try {
        await axios.delete(`http://localhost:3000/produk/${id}`);
        produkList.value = produkList.value.filter(produk => produk.id !== id);
      } catch (error) {
        console.error('Error deleting product', error);
      }
    }

    function edit(produk) {
      form.id = produk.id.toString();
      form.nama = produk.nama;
      form.kategori = produk.kategori;
    }

    onMounted(load);

    return { form, produkList, save, edit, del, load, kategoriList };
  },
};
</script>

<style>
.app-container {
  font-family: Arial, sans-serif;
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  margin-top: 80px;
}

.container {
  background: #f9f9f9;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  max-width: 100%;
}

.product-form {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 20px;
}

.product-form input,
.product-form select {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.form-buttons {
  display: flex;
  justify-content: space-between;
}

.form-buttons button {
  padding: 10px;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.load-button {
  background: #007bff;
}

.save-button {
  background: #28a745;
}

.table-wrapper {
  overflow-x: auto;
}

.product-table {
  width: 100%;
  border-collapse: collapse;
}

.product-table th,
.product-table td {
  padding: 12px;
  border: 1px solid #ddd;
  text-align: left;
}

.product-table th {
  background: #f1f1f1;
}

.edit-button,
.delete-button {
  padding: 5px 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.edit-button {
  background: #ffc107;
  color: white;
}

.delete-button {
  background: #dc3545;
  color: white;
}
</style>
