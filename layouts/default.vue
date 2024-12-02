<template>
  <v-card>
    <v-layout>
      <v-main style="height: 100vh; background-color: black;" theme="dark">
        <v-row class="mt-3 ml-4" align="center" justify="start">
          <v-col>
            <v-btn class="mt-3" color="white" disabled>
              {{ userName }}
            </v-btn>
          </v-col>
          
          <v-col>
            <v-btn class="mt-3 ml-4" color="error" @click="logout">
              Logout
            </v-btn>
          </v-col>
        </v-row>

        <slot />
      </v-main>
    </v-layout>
  </v-card>
</template>

<script>
import axios from 'axios';

export default {
  name: 'default',
  data() {
    return {
      userName: '',
    };
  },
  created() {
    this.getUserName();
  },
  methods: {
    async getUserName() {
      try {
        const response = await this.$api.get('/auth/decode-token');
        this.userName = response.user_name;
      } catch (error) {
        console.error("Erro ao obter o nome do usuário:", error);
      }
    },

    logout() {
      localStorage.removeItem('token'); 

      this.$router.push('/'); 
    },
  },
};
</script>
