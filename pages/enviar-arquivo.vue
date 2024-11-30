<template>
  <v-container class="justify-center mt-5">
    <v-file-input
      v-model="imageFile"
      accept="image/*"
      label="Selecione uma imagem"
      outlined
      @change="previewImage"
    ></v-file-input>
    <v-img
      v-if="imagePreview"
      :src="imagePreview"
      alt="Preview da Imagem"
      max-height="300"
      class="mt-3"
    ></v-img>
    <v-progress-linear
      v-if="loading"
      indeterminate
      color="blue"
      class="mt-3"
    ></v-progress-linear>
    <v-list v-if="results.length > 0" class="mt-5">
      <v-list-subheader>Resultados</v-list-subheader>
      <v-list-item
        v-for="(result, index) in results"
        :key="index"
      >
        <v-list-item-content>
          <v-list-item-title>{{ result.label }}</v-list-item-title>
          <v-list-item-subtitle>{{ result.percentage }}%</v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
    </v-list>
    <v-btn class="mt-5" color="primary" @click="clearHistory">
      Limpar Histórico
    </v-btn>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      imageFile: null, 
      imagePreview: null, 
      results: [], 
      loading: false, 
    };
  },
  created() {
    this.loadHistory();
  },
  methods: {
    previewImage() {
      if (this.imageFile) {
        this.imagePreview = URL.createObjectURL(this.imageFile);
      }
    },
    async uploadImage() {
      if (!this.imageFile) return;

      this.loading = true;

      const formData = new FormData();
      formData.append("file", this.imageFile);

      try {
        const response = await this.$api.post("/ia/analyze", formData, {
          headers: { "Content-Type": "multipart/form-data" },
        });

        this.results = response.data.map((item) => ({
          label: item.label,
          percentage: (item.probability * 100).toFixed(2),
        }));

        this.saveHistory();
      } catch (error) {
        alert("Erro ao enviar a imagem. Tente novamente.");
      } finally {
        this.loading = false;
      }
    },
    saveHistory() {
      if (process.client) {
        const history = JSON.parse(localStorage.getItem("uploadHistory")) || [];
        history.push({ date: new Date().toISOString(), results: this.results });
        localStorage.setItem("uploadHistory", JSON.stringify(history));
      }
    },
    loadHistory() {
      if (process.client) {
        const history = JSON.parse(localStorage.getItem("uploadHistory")) || [];
        if (history.length > 0) {
          this.results = history[history.length - 1].results;
        }
      }
    },
    clearHistory() {
      if (process.client) {
        localStorage.removeItem("uploadHistory");
        this.results = [];
        alert("Histórico limpo com sucesso!");
      }
    },
  },
};
</script>
