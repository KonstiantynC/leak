<template>
  <div>
    <div class="" v-if="data.length">
      <VCard 
        class="mb-6 domain_item"
        v-for="item in data" 
        :key="item.id"
      >
        <VCardText>
          {{ item.domain }}
        </VCardText>
        <VBtn 
          class="button_btn"
          type="button"
          @click="() => handleCredentialsClick(item.external_id)"
          >
          Credentials
        </VBtn>
        <VBtn 
        class="button_btn" 
        type="button"
        @click="() => handleLogsClick(item.external_id)"
        >
          logs
        </VBtn>
      </VCard>
      <div v-if="showCredentials" class="grid-container_log">
        <VCard class="mb-6 vcard-logs" >
          <VCardText>
            <VTable>
              <thead>
                <tr>
                  <th>Credential Count</th>
                  <th>Hostname</th>
                </tr>
              </thead>
              <tbody>
                <tr 
                  v-for="(item, index) in paginatedResources"
                  :key="index"
                >
                  <td style="word-break: break-word;">{{ item.credential_count }}</td>
                  <td style="word-break: break-word;">{{ item.hostname }}</td>
                </tr>
              </tbody>
            </VTable>
          </VCardText>
          <v-pagination
            v-model="currentPageResources"
            :length="pageCountResources"
            @input="onPageChangeResources"
            class="my-4"
          />
        </VCard>
        <VCard class="mb-6 vcard-logs"> 
          <VCardText>
            <VTable>
              <thead>
                <tr>
                  <th>Credential Count</th>
                  <th>Login</th>
                </tr>
              </thead>
              <tbody>
                <tr 
                  v-for="(item, index) in paginatedLogins"
                  :key="index"
                >
                  <td style="word-break: break-word;">{{ item.count }}</td>
                  <td style="word-break: break-word;">{{ item.login }}</td>
                </tr>
              </tbody>
            </VTable>
          </VCardText>
          <v-pagination
            v-model="currentPageLogins"
            :length="pageCountLogins"
            @input="onPageChangeLogins"
            class="my-4"
          />
        </VCard>
      </div>
      <VCard v-if="showCredentials" class="mb-6">
        <VCardText>Statistics: number of credentials: {{ totalItems }}</VCardText>
      </VCard>
      <VCard 
        class="mb-6"  
      >
        <VCardText v-if="showCredentials" class="vcard-logs">
          <VTable>
            <thead>
              <tr>
                <th>Connected Resources</th>
                <th>Login</th>
                <th>Password</th>
                <th>Connected Sources</th>
              </tr>
            </thead>
            <tbody>
              <tr 
                v-for="item in paginatedCreds" 
                :key="item.id"
              >
                <td style="word-break: break-word;">
                  <div style="white-space: pre-line;">
                    <div 
                      v-for="(resource, index) in item.connected_resources" 
                      :key="index"
                    >
                      {{ resource.hostname }}
                    </div>
                  </div>
                </td>
                <td style="word-break: break-word;">{{ item.login }}</td>
                <td style="word-break: break-word;">{{ item.password }}</td>
                <td style="word-break: break-word;">
                  <div style="white-space: pre-line;">
                    <div 
                      v-for="(source, index) in item.connected_sources" 
                      :key="index"
                    >
                      {{ source.filename }}
                    </div>
                  </div>
                </td>
              </tr>
            </tbody>
          </VTable>
          <v-pagination
            v-model="currentPage"
            :length="pageCount"
            @input="onPageChange"
            class="my-4"
          />
        </VCardText>
      </VCard>
      <div v-if="showLogs" class="grid-container_logs">
        <VCard 
          class="mb-6 vcard-logs" 
          v-for="(item, index) in dataLogs.results" 
          :key="item.id"
        >
          <VCardText>Compromised machine: {{ index + 1 }}</VCardText>
          <VCardText class="data-logs_item">
            <div class="">machine_id: {{ item.machine_id}}</div>
            <div class="">computer_name: {{ item.computer_name }}</div>
            <div class="">hardware_id: {{ item.hardware_id }}</div>
            <div class="">os: {{ item.os }}</div>
            <div class="">machine_user: {{ item.machine_user }}</div>
            <div class="">ip_address: {{ item.ip_address }}</div>
            <div class="">country: {{ item.country }}</div>
            <div class="">log_date: {{ item.log_date }}</div>
            <div class="btn-log">
            </div>
          </VCardText>
          <VBtn
            class="button_log_btn" 
            type="button" 
            @click="() => getLog(item.id)"
          >
            Get Log
          </VBtn>
        </VCard>
      </div>
      <div v-if="showLog" class="grid-container_log">
        <VCard>
          <VCardText>
            <div class="">machine_id: {{ dataLog?.machine_id}}</div>
            <div class="">computer_name: {{ dataLog?.computer_name }}</div>
            <div class="">hardware_id: {{ dataLog?.hardware_id }}</div>
            <div class="">os: {{ dataLog?.os }}</div>
            <div class="">machine_user: {{ dataLog?.machine_user }}</div>
            <div class="">ip_address: {{ dataLog?.ip_address }}</div>
            <div class="">country: {{ dataLog?.country }}</div>
            <div class="">log_date: {{ dataLog?.log_date }}</div>
            <!-- <div class="">tree: {{ dataLog?.tree }}</div> -->
          </VCardText>
        </VCard>
        <VCard class="archive-card">
          <VBtn 
            class="button_btn " 
            type="button"
          >
            Download archive
          </VBtn>
        </VCard>
      </div>
    </div>
  </div>
</template>


<script setup>
import axios from 'axios';
import { onMounted, ref } from 'vue';
import { useRouter } from 'vue-router';


const router = useRouter()

const data = ref([]); 
const dataCreds = ref([]); 
const dataLogs = ref([]); 
const dataLogins = ref([]); 
const dataResources = ref([]); 
const dataLog = ref([]); 
const showCredentials = ref(false);
const showLogs = ref(false);
const showLog = ref(false);

// for credentials
const currentPage = ref(1);
const itemsPerPage = 10; 
const totalItems = ref(0); 

// for logins
const currentPageLogins = ref(1);
const itemsPerPageLogins = 10;
const totalItemsLogins = ref(0);

// for resources
const currentPageResources = ref(1);
const itemsPerPageResources = 10;
const totalItemsResources= ref(0);


// creds pagination
const paginatedCreds = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  return dataCreds.value.slice(start, start + itemsPerPage);
});

const pageCount = computed(() => {
  return Math.ceil(totalItems.value / itemsPerPage);
});

const onPageChange = (page, external_id) => {
  currentPage.value = page;
  getDomainCreds(external_id);
};


// logins Pagination
const paginatedLogins = computed(() => {
  const start = (currentPageLogins.value - 1) * itemsPerPageLogins;
  return dataLogins.value.slice(start, start + itemsPerPageLogins);
});

const pageCountLogins = computed(() => {
  return Math.ceil(totalItemsLogins.value / itemsPerPageLogins);
});

const onPageChangeLogins = (pageLogins, external_id) => {
  currentPageLogins.value = pageLogins;
  getDomainLogins(external_id);
};

// Resources Pagination
const paginatedResources = computed(() => {
  const start = (currentPageResources.value - 1) * itemsPerPageResources;
  return dataResources.value.slice(start, start + itemsPerPageResources);
});

const pageCountResources = computed(() => {
  return Math.ceil(totalItemsResources.value / itemsPerPageResources);
});

const onPageChangeResources = (pageResources, external_id) => {
  currentPageResources.value = pageResources;
  getDomainResources(external_id);
};



const toggleCredentials = async (external_id) => {
  try {
    showCredentials.value = !showCredentials.value;
    showLogs.value = false; 
    showLog.value = false; 

    if (showCredentials.value) {
      currentPage.value = 1; 
      currentPageLogins.value = 1; 
      currentPageResources.value = 1; 
      await getDomainCreds(external_id); 
    } else {
      data.value = []; 
    }
  } catch (error) {
    console.error('Ошибка при переключении credentials:', error);
  }
};

const toggleLogs = async (external_id) => {
  try {
    showLogs.value = !showLogs.value;
    showCredentials.value = false; 
    showLog.value = false;
    if (showLogs.value) {
      await getDomainLogs(external_id); 
    }
  } catch (error) {
    console.error('Ошибка при переключении logs:', error);
  }
};

const handleLogsClick = async (external_id) => {
  try {
    await toggleLogs(external_id);
    await getDomainLogins(external_id);
    await getDomainResourses(external_id);
  } catch (error) {
    console.error('Ошибка при выполнении обработчиков:', error);
  }
};
const handleCredentialsClick = async (external_id) => {
  try {
    await toggleCredentials(external_id);
    await getDomainLogins(external_id);
    await getDomainResourses(external_id);
  } catch (error) {
    console.error('Ошибка при выполнении обработчиков:', error);
  }
};


const getDomainUsers = async () => {
  try {
    const token = localStorage.getItem('access_token');
    const response = await axios.get(`${import.meta.env.VITE_APP_BASE_URL}/api/me/domains`, {
      headers: {
        'Authorization': `Bearer ${token}`, 
        'Content-Type': 'application/json',
      },
    });
    data.value = response.data; 
  } catch (error) {
    console.error('Ошибка при запросе:', error);
  }
};

const getDomainCreds = async (external_id) => {
  dataCreds.value = []; 
  let page = 1; 
  let totalFetchedCreds = 0;

  try {
    while (page <= 5) {
      const token = localStorage.getItem('access_token');
      const response = await axios.get(`${import.meta.env.VITE_APP_BASE_URL}/api/domain/${external_id}/credentials`, {
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
      
      dataCreds.value.push(...results);
      totalFetchedCreds += results.length;
      page++; 
    }

    totalItems.value = totalFetchedCreds; 
    console.log('totalItems', totalItems)
  } catch (error) {
    console.error('Ошибка при запросе:', error.response?.data || error);
  }
};

const getDomainLogs = async (external_id) => {
  try {
    const token = localStorage.getItem('access_token');
    const response = await axios.get(`${import.meta.env.VITE_APP_BASE_URL}/api/domain/${external_id}/logs`, {
      headers: {
        'Authorization': `Bearer ${token}`, 
        'Content-Type': 'application/json',
      },
  });
    dataLogs.value = response.data;

  } catch (error) {
    console.error('Ошибка при запросе:', error.response?.data || error);
  }
};

const getDomainLogins = async (external_id) => {
  dataLogins.value = []; 
  let pageLogins = 1; 
  let totalFetchedLogins = 0;
  try {
    while (pageLogins <= 5) {
      const token = localStorage.getItem('access_token');
      const response = await axios.get(`${import.meta.env.VITE_APP_BASE_URL}/api/domain/${external_id}/logins`, {
        headers: {
          'Authorization': `Bearer ${token}`,
          'Content-Type': 'application/json',
        },
        params: {
          page: pageLogins,
          per_page: itemsPerPageLogins,
        },
      });

      const results = response.data.results;
      if (!results.length) {
        break; 
      }
      
      dataLogins.value.push(...results);
      totalFetchedLogins += results.length;
      pageLogins++; 
    }

    totalItemsLogins.value = totalFetchedLogins;
    
  } catch (error) {
    console.error('Ошибка при запросе:', error.response?.data || error);
  }
};

const getDomainResourses = async (external_id) => {
  dataResources.value = []; 
  let pageResources = 1; 
  let totalFetchedResources = 0;
  try {
    while (pageResources <= 5) {
      const token = localStorage.getItem('access_token');
      const response = await axios.get(`${import.meta.env.VITE_APP_BASE_URL}/api/domain/${external_id}/resources`, {
        headers: {
          'Authorization': `Bearer ${token}`,
          'Content-Type': 'application/json',
        },
        params: {
          page: pageResources,
          per_page: itemsPerPageResources,
        },
      });

      const results = response.data.results;
      if (!results.length) {
        break; 
      }
      
      dataResources.value.push(...results);
      totalFetchedResources += results.length;
      pageResources++; 
    }

    totalItemsResources.value = totalFetchedResources;

  } catch (error) {
    console.error('Ошибка при запросе:', error.response?.data || error);  
  }
};

const getLog = async (id) => {
  try {
    const token = localStorage.getItem('access_token');
    const response = await axios.get(`${import.meta.env.VITE_APP_BASE_URL}/api/log/${id}`, {
      headers: {
        'Authorization': `Bearer ${token}`, 
        'Content-Type': 'application/json',
      },
  });
    dataLog.value = response.data; 
    showLogs.value = false;
    showCredentials.value = false;
    showLog.value = true;

  } catch (error) {
    console.error('Ошибка при запросе:', error.response?.data || error);
  }
};


const main = async () => {
  await getDomainUsers(); 
};


onMounted(() => {
  const token = localStorage.getItem('access_token');
  if (!token) {
    console.warn('Користувач не авторизований, редирект на логін');
    router.push('/login');
  }
});
main();

</script>

<style scoped>
table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

th, td {
  border: 1px solid #ccc;
  padding: 12px;
  text-align: left;
}

th {
  font-weight: bold;
}

td {
  vertical-align: top; 
}

.grid-container_logs {
  display: grid;
  grid-template-columns: repeat(4, 1fr); 
  gap: 16px; 
}

.vcard-logs {
  display: flex;
  flex-direction: column; 
  justify-content: space-between; 
  padding-bottom: 15px;
}
.grid-container_log{
  display: grid;
  grid-template-columns: repeat(2, 1fr); 
  gap: 16px; 
}

.VCard { 
  min-width: 200px;
}

.domain_item {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
  padding: 0 10px;
}

.button_btn {
  width: 200px !important;
  margin: auto;
}

.button_log_btn {
  margin-top: auto; 
  margin-left: auto; 
  margin-right: auto; 
  display: flex;
  justify-content: center; 
  width: 200px !important;
}

.btn-log {
  display: flex;
  justify-content: center;
  margin: 35px 0 0 0;
  margin-top: auto;
}

.archive-card {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
