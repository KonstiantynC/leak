<template>
  <div>
    <div v-if="data.length"> 
      <VCardText>Domains List:</VCardText>
      <ul>
        <VCard 
          class="mb-6 add_card"
          v-for="item in paginatedData" 
          :key="item.id"
        >
          <VCardText>
            {{ item.domain }}
          </VCardText>
          <div>
            <VBtn 
              block 
              type="button" 
              class="button"
              @click="addDomain(item)"
            >
              Add
            </VBtn>
          </div>
        </VCard> 
      </ul>
      <v-pagination
        v-model="currentPage"
        :length="pageCount"
        @input="onPageChange"
        class="my-4"
      />
    </div>
  </div>
</template>

<script setup>
import axios from 'axios';
import { computed, onMounted, ref } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter()
const data = ref([]); 
const itemsPerPage = 10; 
const currentPage = ref(1);
const totalItems = ref(0); 

const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  return data.value.slice(start, start + itemsPerPage);
});

const pageCount = computed(() => {
  return Math.ceil(totalItems.value / itemsPerPage);
});

const onPageChange = (page) => {
  currentPage.value = page;
  getDomains(); 
};

const getDomains = async () => {
  data.value = []; 
  let page = 1;
  let totalFetched = 0; 
  try {
    while (true) {
      const token = localStorage.getItem('access_token');
      const response = await axios.get(`${import.meta.env.VITE_APP_BASE_URL}/api/domains/`, {
        headers: {
        'Authorization': `Bearer ${token}`, 
        'Content-Type': 'application/json',
      },
        params: {
          page: page,
          per_page: itemsPerPage,
        },
      });
      
      const results = response.data.results;

      
      if (!results.length) {
        break;
      }

      data.value.push(...results); 
      totalFetched += results.length; 
      page++; 
    }
    
    totalItems.value = totalFetched; 
  } catch (error) {
    console.error('Ошибка при запросе:', error);
  }
};

const addDomain = async (item) => {
  try {
    const payload = {
      domain: item.domain,
      external_id: item.id,
      is_internal: item.is_internal
    };

    const response = await axios.post(
      `${import.meta.env.VITE_APP_BASE_URL}/api/me/domains/`, 
      payload,
      {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${localStorage.getItem('access_token')}`,
        }
      }
    );

    console.log('Домен добавлен:', response.data); 
    await getDomains(); 
  } catch (error) {
    console.error('Ошибка при добавлении домена:', error.response?.data || error.message);
  }
};

onMounted(() => {
  const token = localStorage.getItem('access_token');
  if (!token) {
    router.push('/login');
  } else {
    getDomains(); 
  }
});
</script>

<style scoped>
.add_card {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  padding: 0 10px;
}
</style>
