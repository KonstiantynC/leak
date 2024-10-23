<script setup>
import { useTheme } from 'vuetify'
import ScrollToTop from '@core/components/ScrollToTop.vue'
import initCore from '@core/initCore'
import {
  initConfigStore,
  useConfigStore,
} from '@core/stores/config'
import { hexToRgb } from '@core/utils/colorConverter'
import { useRouter } from 'vue-router'
import axios from 'axios'

const router = useRouter()
onMounted(async () => {
  const accessToken = localStorage.getItem('access_token')
  const refreshToken = localStorage.getItem('refresh_token')

  if (accessToken) {
    try {
      axios.defaults.headers.common['Authorization'] = `Bearer ${accessToken}`
    } catch (error) {
      console.error('Access token is invalid, refreshing...')
      await refreshAccessToken(refreshToken)
    }
  }
})

const refreshAccessToken = async (refreshToken) => {
  if (!refreshToken) {
    console.error('No refresh token available.')
    router.push('/login')
  }

  try {
    const response = await axios.post(
      `${import.meta.env.VITE_APP_BASE_URL}/api/token/refresh/`,
      {
        refresh: refreshToken,
      },
      {
        headers: {
          accept: 'application/json',
          'Content-Type': 'application/json',
        },
      }
    )

    const { access } = response.data  
    localStorage.setItem('access_token', access)
    axios.defaults.headers.common['Authorization'] = `Bearer ${access}`
  } catch (error) {
    console.error('Error refreshing access token:', error.response?.data || error.message)
    router.push('/login') 
  }
}

const { global } = useTheme()

// ℹ️ Sync current theme with initial loader theme
initCore()
initConfigStore()

const configStore = useConfigStore()
</script>

<template>
  <VLocaleProvider :rtl="configStore.isAppRTL">
    <!-- ℹ️ This is required to set the background color of active nav link based on currently active global theme's primary -->
    <VApp :style="`--v-global-theme-primary: ${hexToRgb(global.current.value.colors.primary)}`">
      <RouterView />

      <ScrollToTop />
    </VApp>
  </VLocaleProvider>
</template>
