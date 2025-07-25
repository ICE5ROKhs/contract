<template>
  <div class="flex">
    <Test />
    <Sidebar />
    <div class="contract-list">
      <h2>合同定稿列表</h2>
      <div class="tab-buttons">
        <!-- 可以在这里添加标签按钮 -->
      </div>
      
      <!-- 搜索栏 -->
      <div class="search-bar">
        <input 
          type="text" 
          v-model="searchQuery" 
          placeholder="输入合同名称或编号搜索..." 
          @keyup.enter="searchContracts"
        />
        <button @click="searchContracts" class="search-button">
          <span class="search-icon">🔍</span>
        </button>
      </div>
      
      <div v-if="loading" class="loading">加载中...</div>
      <div v-else>
        <!-- 待定稿合同列表 -->
        <table v-if="activeTab === 'draft'" class="contract-table">
          <thead>
            <tr>
              <th>合同编号</th>
              <th>合同名称</th>
              <th>申请日期</th>
              <th>操作</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="contract in filteredDraftContracts" :key="contract.id">
              <td>{{ contract.ContractID }}</td>
              <td>{{ contract.Title }}</td>
              <td>{{ formatDate(contract.LastModifiedDate) }}</td>
              <td>
                <button @click="viewContract(contract.ContractID)">查看</button>
              </td>
            </tr>
          </tbody>
        </table>
        
        <div v-if="activeTab === 'draft' && filteredDraftContracts.length === 0" class="no-data">
          {{ searchQuery ? '没有找到匹配的合同' : '暂无待定稿合同' }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Sidebar from '../components/sidebar.vue';
import { useAuthStore } from '../common/auth';
import axios from 'axios';

export default {
  components: {
    Sidebar
  },
  data() {
    return {
      draftContracts: [],
      loading: true,
      activeTab: 'draft', // 默认显示待定稿列表
      searchQuery: ''
    };
  },
  computed: {
    filteredDraftContracts() {
      if (!this.searchQuery) return this.draftContracts;
      const query = this.searchQuery.toLowerCase();
      return this.draftContracts.filter(contract => 
        contract.ContractID.toLowerCase().includes(query) || 
        contract.Title.toLowerCase().includes(query)
      );
    }
  },
  created() {
    this.fetchContracts();
  },
  methods: {
    async fetchContracts() {
      try {
        this.loading = true;
        const authStore = useAuthStore(); // 获取 authStore
        const username = authStore.currentUser.username; // 从 authStore 获取用户名

        // 获取待定稿合同
        const draftRes = await axios.get("/finalize/list", {
          params: {
            username: username // 将用户名作为请求参数传递
          }
        });

        this.draftContracts = draftRes.data.rows;
        this.draftContracts.forEach(contract => {
          contract.approver = draftRes.data.rowsApprover;
        });

        this.loading = false;
      } catch (error) {
        console.error('获取合同列表失败:', error);
        this.loading = false;
      }
    },
    viewContract(contractId) {
      this.$router.push(`/FinalizeContract/${contractId}`);
    },
    formatDate(dateString) {
      if (!dateString) return 'N/A';
      return new Date(dateString).toLocaleDateString();
    },
    searchContracts() {
      // 搜索逻辑已经在计算属性中实现
    }
  }
};
</script>

<style scoped>
.flex {
  display: flex;
}

.contract-list {
  margin-top: 20px;
  margin-left: 200px; 
  padding: 20px;
  flex-grow: 1;
}

.tab-buttons {
  margin-bottom: 20px;
}

.tab-buttons button {
  padding: 8px 16px;
  margin-right: 10px;
  background-color: #f2f2f2;
  border: 1px solid #ddd;
  border-radius: 4px 4px 0 0;
  cursor: pointer;
}

.tab-buttons button.active {
  background-color: #4CAF50;
  color: white;
  border-color: #4CAF50;
}

/* 搜索栏样式 */
.search-bar {
  display: flex;
  margin-bottom: 20px;
}

.search-bar input {
  flex: 1;
  padding: 10px 15px;
  border: 1px solid #ddd;
  border-radius: 4px 0 0 4px;
  font-size: 14px;
  outline: none;
}

.search-bar input:focus {
  border-color: #4CAF50;
}

.search-button {
  padding: 0 15px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.search-button:hover {
  background-color: #45a049;
}

.search-icon {
  font-size: 16px;
}

.contract-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 15px;
}

.contract-table th, .contract-table td {
  border: 1px solid #ddd;
  padding: 8px 12px;
  text-align: left;
}

.contract-table th {
  background-color: #f2f2f2;
}

.contract-table tr:nth-child(even) {
  background-color: #f9f9f9;
}

.contract-table tr:hover {
  background-color: #f1f1f1;
}

button {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}

.loading, .no-data {
  text-align: center;
  padding: 20px;
  color: #666;
}
</style>