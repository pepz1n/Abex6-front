<template>
  <v-container>
    <v-row>
      <v-col cols="12" sm="8" md="4" class="mx-auto">
        <v-card>
          <v-card-title class="text-h5">Register</v-card-title>
          
          <v-form v-model="valid" @submit.prevent="register">
            <v-text-field
              v-model="username"
              label="Nome"
              :rules="[rules.required]"
              outlined
              dense
            />
            <v-text-field
              v-model="email"
              label="Email"
              :rules="[rules.required, rules.email]"
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
            <v-btn :disabled="!valid" type="submit" color="primary">Register</v-btn>
          </v-form>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'

definePageMeta({
  layout: 'admin'
});

const router = useRouter()
const valid = ref(false)
const username = ref('')
const email = ref('')
const password = ref('')
const rules = {
  required: value => !!value || 'Obrigatório',
  email: value => /.+@.+\..+/.test(value) || 'Email Válido',
}

const register = async () => {
  try {
    const response = await axios.post('http://localhost:3333/auth/register', {
      name: username.value,
      email: email.value,
      password: password.value,
    })

    alert('Deu Certo')
    router.push('/')  
  } catch (error) {
    alert('Falhou')
  }
}
</script>
