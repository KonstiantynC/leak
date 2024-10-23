<template>
  <a href="javascript:void(0)">
    <div class="app-logo auth-logo">
      <VNodeRenderer :nodes="themeConfig.app.logo" />
      <h1 class="app-logo-title">
        {{ themeConfig.app.title }}
      </h1>
    </div>
  </a>

  <VRow no-gutters class="auth-wrapper">
    <VCol
      md="8"
      class="d-none d-md-flex align-center justify-center position-relative"
    >
      <div class="d-flex align-center justify-center pa-10">
        <img
          :src="authV2LoginIllustration"
          class="auth-illustration w-100"
          alt="auth-illustration"
        />
      </div>
      <VImg
        :src="authV2LoginMask"
        class="d-none d-md-flex auth-footer-mask"
        alt="auth-mask"
      />
    </VCol>
    <VCol
      cols="12"
      md="4"
      class="auth-card-v2 d-flex align-center justify-center"
      style="background-color: rgb(var(--v-theme-surface));"
    >
      <VCard flat :max-width="500" class="mt-12 mt-sm-0 pa-5 pa-lg-7">
        <VCardText>
          <h4 class="text-h4 mb-1">
            Welcome to <span class="text-capitalize">{{ themeConfig.app.title }}! 👋🏻</span>
          </h4>

          <p class="mb-0">
            Please sign-in to your account and start the adventure
          </p>
        </VCardText>

        <VCardText>
          <VForm @submit.prevent="Login">
            <VRow>
              <!-- username -->
              <VCol cols="12">
                <VTextField
                  v-model="form.username"
                  autofocus
                  label="Username"
                  :rules="usernameRules"
                  placeholder="johndoe"
                />
              </VCol>

              <!-- password -->
              <VCol cols="12">
                <VTextField
                  v-model="form.password"
                  label="Password"
                  :rules="passwordRules"
                  placeholder="············"
                  :type="isPasswordVisible ? 'text' : 'password'"
                  :append-inner-icon="isPasswordVisible ? 'ri-eye-off-line' : 'ri-eye-line'"
                  @click:append-inner="isPasswordVisible = !isPasswordVisible"
                />

                <!-- remember me checkbox -->
                <div class="d-flex align-center justify-space-between flex-wrap my-6 gap-x-2">
                  <VCheckbox
                    v-model="form.remember"
                    label="Remember me"
                    />

                    <RouterLink
                    class="text-primary"
                    href="javascript:void(0)"
                    :to="{ name: 'forgot-password'}"
                    >
                    Forgot Password?
                  </RouterLink>
                </div>

                <!-- login button -->
                <VBtn block type="submit">
                  Login
                </VBtn>
              </VCol>
              


              <!-- create account -->
              <VCol cols="12" class="text-body-1 text-center">
                <span class="d-inline-block">
                  New on our platform?
                </span>
                <RouterLink
                  class="text-primary ms-1 d-inline-block text-body-1"
                  href="javascript:void(0)"
                  :to="{ name: 'register' }"
                >
                  Create an account
                </RouterLink>
              </VCol>

              <VCol cols="12" class="d-flex align-center">
                <VDivider />
                <span class="mx-4 text-high-emphasis">or</span>
                <VDivider />
              </VCol>

              <!-- auth providers -->
              <VCol cols="12" class="text-center">
                <AuthProvider />
              </VCol>
            </VRow>
          </VForm>
        </VCardText>
      </VCard>
    </VCol>
  </VRow>
</template>

<script setup>
import { useGenerateImageVariant } from '@/@core/composable/useGenerateImageVariant'
import AuthProvider from '@/views/pages/authentication/AuthProvider.vue'
import authV2LoginIllustrationBorderedDark from '@images/pages/auth-v2-login-illustration-bordered-dark.png'
import authV2LoginIllustrationBorderedLight from '@images/pages/auth-v2-login-illustration-bordered-light.png'
import authV2LoginIllustrationDark from '@images/pages/auth-v2-login-illustration-dark.png'
import authV2LoginIllustrationLight from '@images/pages/auth-v2-login-illustration-light.png'
import authV2LoginMaskDark from '@images/pages/auth-v2-login-mask-dark.png'
import authV2LoginMaskLight from '@images/pages/auth-v2-login-mask-light.png'
import { VNodeRenderer } from '@layouts/components/VNodeRenderer'
import { themeConfig } from '@themeConfig'
import axios from 'axios'
import { ref } from 'vue'
import { useRouter } from 'vue-router'


definePage({
  meta: {
    layout: 'blank',
    public: true,
  },
})

const router = useRouter()
const form = ref({
  username: '',
  password: '',
  remember: false,
})

const isPasswordVisible = ref(false)
const errorMessage = ref('')

const usernameRules = [
  (v) => !!v || 'Username is required',
]

const passwordRules = [
  (v) => !!v || 'Password is required',
]

const Login = async () => {
  if (!form.value.username || !form.value.password) {
    console.error('Username and password are required.');
    return;
  }

  try {
    const response = await axios.post(
      `${import.meta.env.VITE_APP_BASE_URL}/api/token/`, 
      {
        username: form.value.username,
        password: form.value.password,
      },
      {
        headers: {
          accept: 'application/json',
          'Content-Type': 'application/json',
        },
      }
    );

    const { refresh, access } = response.data;

    localStorage.setItem('access_token', access);
    localStorage.setItem('refresh_token', refresh);
    axios.defaults.headers.common['Authorization'] = `Bearer ${access}`;

    router.push('/');
  } catch (error) {
    console.error('Authorization error:', error.response?.data || error.message);

    errorMessage.value =
      error.response?.data?.detail || 'Invalid credentials. Please try again.';
  }
};



const authV2LoginMask = useGenerateImageVariant(authV2LoginMaskLight, authV2LoginMaskDark)
const authV2LoginIllustration = useGenerateImageVariant(authV2LoginIllustrationLight, authV2LoginIllustrationDark, authV2LoginIllustrationBorderedLight, authV2LoginIllustrationBorderedDark, true)
</script>

<style lang="scss">
@use "@core/scss/template/pages/page-auth";
</style>
