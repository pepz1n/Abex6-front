<template>
  <v-container>
    <v-row>
      <v-col cols="12" sm="8" md="4" class="mx-auto">
        <v-card>
          <v-card-title class="text-h5">Login</v-card-title>
          
          <v-form v-model="valid" @submit.prevent="login">
            <v-text-field
              v-model="username"
              label="Email"
              :rules="[rules.required]"
              outlined
              dense
            />
            <v-text-field
              v-model="password"
              label="Senha"
              type="password"
              :rules="[rules.required]"
              outlined
              dense
            />
            <v-btn :disabled="!valid" type="submit" color="primary">Login</v-btn>
            <v-btn class="ml-3" @click="router.push('/register')" color="warning"> Cadastro </v-btn>
          </v-form>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'

definePageMeta({
  layout: 'admin'
});

const router = useRouter()
const valid = ref(false)
const username = ref('')
const password = ref('')
const rules = {
  required: value => !!value || 'Obrigatório.',
}

const login = async () => {
  try {
    const response = await axios.post('http://localhost:3333/auth/login', {
      email: username.value,
      password: password.value,
    })

    const token = response.data.token
    localStorage.setItem('token', token)  

    router.push('/test')
  } catch (error) {
    alert('Login Falhou')
  }
}
</script>
