<template>
  <div>
    <h1>新增投票項目</h1>
    <input v-model="itemName" placeholder="投票項目名稱" />
    <button @click="addItem">新增</button>

    <h2>目前項目</h2>
    <ul>
      <li v-for="item in items" :key="item.id">
        {{ item.name }}
        <button @click="deleteItem(item.id)">刪除</button>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      itemName: '',
      items: []
    }
  },
  methods: {
    fetchItems() {
      axios.get('http://localhost:8080/api/admin/items')
        .then(res => this.items = res.data)
        .catch(err => alert('獲取投票項目失敗：' + err.message));
    },
    addItem() {
      if (!this.itemName) return alert('請輸入項目名稱');
      if (/[<>&"']/.test(this.itemName)) return alert('項目名稱包含非法字符');
      axios.post('http://localhost:8080/api/admin/items', null, {
        params: { name: this.itemName }
      })
        .then(() => {
          alert('新增成功');
          this.itemName = '';
          this.fetchItems();
        })
        .catch(err => alert('新增失敗：' + err.response?.data || err.message));
    },
    deleteItem(id) {
      if (!confirm('確定要刪除此投票項目？')) return;
      axios.delete(`http://localhost:8080/api/admin/items/${id}`)
        .then(() => {
          alert('刪除成功');
          this.fetchItems();
        })
        .catch(err => alert('刪除失敗：' + err.response?.data || err.message));
    }
  },
  mounted() {
    this.fetchItems();
  }
}
</script>