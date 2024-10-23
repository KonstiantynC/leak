<template>
  <RouterLink to="register">
    <div class="app-logo auth-logo">
      <VNodeRenderer :nodes="themeConfig.app.logo" />
      <h1 class="app-logo-title">
        {{ themeConfig.app.title }}
      </h1>
    </div>
  </RouterLink>

  <VRow
    no-gutters
    class="auth-wrapper"
  >
    <VCol
      md="8"
      class="d-none d-md-flex align-center justify-center position-relative"
    >
      <!-- here your illustrator -->
      <div class="d-flex align-center justify-center pa-10">
        <img
          :src="authThemeImg"
          class="auth-illustration w-100"
          alt="auth-illustration"
        >
      </div>
      <VImg
        :src="authThemeMask"
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
      <VCard
        flat
        :max-width="500"
        class="mt-12 mt-sm-0 pa-5 pa-lg-7"
      >
        <VCardText>
          <h4 class="text-h4 mb-1">
            Adventure starts here 🚀
          </h4>
          <p class="mb-0">
            Make your app management easy and fun!
          </p>
        </VCardText>

        <VCardText>
          <VForm @submit.prevent="registerForm">
            <VRow>
              <!-- Username -->
              <VCol cols="12">
                <VTextField
                  v-model="form.username"
                  :rules="usernameRules"
                  autofocus
                  label="Username"
                  placeholder="Johndoe"
                  :error="!!errors.username"
                  :error-messages="errors.username"
                />
              </VCol>

              <!-- email -->
              <VCol cols="12">
                <VTextField
                  v-model="form.email"
                  label="Email"
                  type="email"
                  :rules="emailRules"
                  placeholder="johndoe@email.com"
                />
              </VCol>

              <!-- password -->
              <VCol cols="12">
                <VTextField
                  v-model="form.password"
                  label="Password"
                  placeholder="············"
                  :rules="passwordRules"
                  :type="isPasswordVisible ? 'text' : 'password'"
                  :append-inner-icon="!isPasswordVisible ? 'ri-eye-off-line' : 'ri-eye-line'"
                  @click:append-inner="isPasswordVisible = !isPasswordVisible"
                />
              </VCol>
              <VCol cols="12">  
                <VTextField
                  v-model="form.confirm_password"
                  label="Confirm password"
                  placeholder="············"
                  :error="!!errors.confirm_password"
                  :error-messages="errors.confirm_password"
                  :type="isConfirmPasswordVisible ? 'text' : 'password'"
                  :append-inner-icon="!isConfirmPasswordVisible ? 'ri-eye-off-line' : 'ri-eye-line'"
                  @click:append-inner="isConfirmPasswordVisible = !isConfirmPasswordVisible"
                />
              </VCol>
              <VCol>
                <div class="d-flex align-center my-6">
                  <VCheckbox
                    id="privacy-policy"
                    v-model="form.privacyPolicies"
                    inline
                  />
                  <VLabel
                    for="privacy-policy"
                    style="opacity: 1;"
                  >
                    <span class="me-1 text-high-emphasis">I agree to</span>
                    <a
                      href="javascript:void(0)"
                      class="text-primary"
                    >privacy policy & terms</a>
                  </VLabel>
                </div>

                <VBtn
                  block
                  type="submit"
                >
                  Sign up
                </VBtn>
              </VCol>

              <!-- create account -->
              <VCol cols="12">
                <div class="text-center text-base">
                  <span class="d-inline-block">Already have an account?</span> <RouterLink
                    class="text-primary d-inline-block"
                    :to="{ name: 'login' }"
                  >
                    Sign in instead
                  </RouterLink>
                </div>
              </VCol>

              <VCol cols="12">
                <div class="d-flex align-center">
                  <VDivider />
                  <span class="mx-4 text-high-emphasis">or</span>
                  <VDivider />
                </div>
              </VCol>

              <!-- auth providers -->
              <VCol
                cols="12"
                class="text-center"
              >
                <AuthProvider />
              </VCol>
            </VRow>
          </VForm>
          <p v-if="errorMessage">{{ errorMessage }}</p>
        </VCardText>
      </VCard>
    </VCol>
  </VRow>
</template>

<script setup>
import AuthProvider from '@/views/pages/authentication/AuthProvider.vue'
import authV2RegisterIllustrationBorderedDark from '@images/pages/auth-v2-register-illustration-bordered-dark.png'
import authV2RegisterIllustrationBorderedLight from '@images/pages/auth-v2-register-illustration-bordered-light.png'
import authV2RegisterIllustrationDark from '@images/pages/auth-v2-register-illustration-dark.png'
import authV2RegisterIllustrationLight from '@images/pages/auth-v2-register-illustration-light.png'
import authV2RegisterMaskDark from '@images/pages/auth-v2-register-mask-dark.png'
import authV2RegisterMaskLight from '@images/pages/auth-v2-register-mask-light.png'
import { VNodeRenderer } from '@layouts/components/VNodeRenderer'
import { themeConfig } from '@themeConfig'
import axios from 'axios'
import { useRouter } from 'vue-router'

const authThemeMask = useGenerateImageVariant(authV2RegisterMaskLight, authV2RegisterMaskDark)
const authThemeImg = useGenerateImageVariant(authV2RegisterIllustrationLight, authV2RegisterIllustrationDark, authV2RegisterIllustrationBorderedLight, authV2RegisterIllustrationBorderedDark, true)

definePage({
  meta: {
    layout: 'blank',
    unauthenticatedOnly: true,
  },
})
const form = ref({
  username: '',
  email: '',
  password: '',
  confirm_password: '',
  privacyPolicies: false,
})

const router = useRouter()
const errors = ref({
  username: '',
})
const errorMessage = ref('');

const validateUsername = async (username) => {
  if (!username) {
    errors.value.username = 'Username is required'
    return false
  }

  if (username.length < 3 || username.length > 20) {
    errors.value.username = 'Username must be between 3 and 20 characters'
    return false
  }

  const regex = /^[a-zA-Z0-9._-]+$/
  if (!regex.test(username)) {
    errors.value.username = 'Invalid username format'
    return false
  }

  errors.value.username = ''
  return true
}

const usernameRules = [
  (v) => !!v || 'Username is required',
]

const emailRules = [
  (v) => !!v || 'Email is required',
  (v) => /.+@.+\..+/.test(v) || 'Email must be valid',
]
const passwordRules = [
  (v) => !!v || 'Password is required',
]

const validateConfirmPassword = () => {
  if (form.value.confirm_password !== form.value.password) {
    errors.value.confirm_password = 'Passwords do not match';
    return false;
  } else {
    errors.value.confirm_password = '';
    return true;
  }
};

const validateForm = async () => {
  let isValid = true

  // Validate username
  if (!(await validateUsername(form.value.username))) isValid = false

  // Validate email
  if (!form.value.email) {
    errors.value.email = 'Email is required'
    isValid = false
  } else if (!/.+@.+\..+/.test(form.value.email)) {
    errors.value.email = 'Invalid email format'
    isValid = false
  } else {
    errors.value.email = ''
  }

  // Validate password
  if (!form.value.password) {
    errors.value.password = 'Password is required'
    isValid = false
  } else if (form.value.password.length < 6) {
    errors.value.password = 'Password must be at least 6 characters'
    isValid = false
  } else {
    errors.value.password = ''
  }

  if (!validateConfirmPassword()) isValid = false;

  return isValid
}

const registerForm = async () => {
  const isValid = await validateForm();
  if (isValid) {
    try {
      const response = await axios.post(
        'http://127.0.0.1:8000/api/register/',
        {
          username: form.value.username,
          email: form.value.email,
          password: form.value.password,
          password_confirm: form.value.confirm_password
        },
        {
          headers: {
            'Content-Type': 'application/json'
          }
        }
      );
      const data = response.data;

      console.log('User registered successfully:', data);

      router.push({ path: 'login' });

    } catch (error) {
      errorMessage.value = error.response.data.error || 'Registration failed.';
      console.error('Error registering user:', error);
    }
  }
};



const isPasswordVisible = ref(false)
const isConfirmPasswordVisible = ref(false)
</script>

<style lang="scss">
@use "@core/scss/template/pages/page-auth.scss";
</style>
