<template>
  <div>
    <h1>線上投票</h1>
    <div v-for="item in items" :key="item.id">
      <label>
        <input type="checkbox" :value="item.id" v-model="selected" />
        {{ item.name }}
      </label>
    </div>
    <input v-model="voterName" placeholder="你的名字" />
    <button @click="submitVote">投票</button>

    <h2>投票結果</h2>
    <ul>
      <li v-for="result in results" :key="result.id">
        {{ result.name }}：{{ result.voteCount }} 票
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      items: [],
      results: [],
      selected: [],
      voterName: ''
    }
  },
  methods: {
    fetchData() {
      axios.get('http://localhost:8080/api/admin/items')
        .then(res => this.items = res.data)
        .catch(err => alert('獲取投票項目失敗：' + err.message));
      axios.get('http://localhost:8080/api/votes')
        .then(res => this.results = res.data)
        .catch(err => alert('獲取投票結果失敗：' + err.message));
    },
    submitVote() {
      if (!this.selected.length || !this.voterName) {
        return alert('請選擇至少一個投票項目並填寫名字');
      }
      if (/[<>&"']/.test(this.voterName)) {
        return alert('名字包含非法字符');
      }
      axios.post('http://localhost:8080/api/vote', null, {
        params: {
          voterName: this.voterName,
          itemIds: this.selected
        },
        paramsSerializer: params => {
          return Object.entries(params)
            .map(([key, value]) => {
              if (Array.isArray(value)) {
                return value.map(val => `${key}=${encodeURIComponent(val)}`).join('&');
              }
              return `${key}=${encodeURIComponent(value)}`;
            })
            .join('&');
        }
      })
        .then(() => {
          alert('投票成功');
          this.voterName = '';
          this.selected = [];
          this.fetchData();
        })
        .catch(err => alert('投票失敗：' + err.response?.data || err.message));
    }
  },
  mounted() {
    this.fetchData();
  }
}
</script>