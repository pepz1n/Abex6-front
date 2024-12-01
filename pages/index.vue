<template>
  <v-container class="justify-center mt-5" style="height: 100vh; overflow-y: auto;">
    <v-file-input
      v-model="imageFile"
      accept="image/*"
      label="Selecione uma imagem"
      outlined
      @change="previewImage"
    />
    <v-img
      v-if="imagePreview"
      :src="imagePreview"
      alt="Preview da Imagem"
      max-height="300"
      class="mt-3"
    />
    <v-btn
      class="mt-3"
      color="primary"
      :disabled="loading || !imageFile"
      @click="uploadImage"
    >
      Enviar Imagem
    </v-btn>
    <v-progress-linear
      v-if="loading"
      indeterminate
      color="blue"
      class="mt-3"
    />
    <v-list v-if="history.length > 0" class="mt-5">
      <v-list-subheader>Histórico de Envios</v-list-subheader>
      <v-list-item
        v-for="(entry, index) in history"
        :key="index"
        class="d-flex"
      >
        <v-img
          :src="entry.image"
          max-height="50"
          max-width="50"
          class="mr-3"
        />
        <v-list-item-content>
          <v-list-item-title>{{ entry.label }}</v-list-item-title>
          <v-list-item-subtitle>{{ entry.percentage }}%</v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
    </v-list>
    <v-btn class="mt-3 ml-4" color="error" @click="clearHistory">
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
      history: [], 
      loading: false,
      classes: {
        0: "Healthy",
        1: "Septoria",
        2: "Stripe Rust",
      },
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
      formData.append("image", this.imageFile);

      try {
        const response = await this.$api.post("/predict", formData, {
          headers: { "Content-Type": "multipart/form-data" },
        });

        const predictions = response.prediction[0];

        this.results = predictions.map((probability, index) => ({
          label: this.classes[index],
          percentage: (probability * 100).toFixed(2),
        }));

        const predictedClass = response.predictedClass;
        const result = {
          label: this.classes[predictedClass],
          percentage: (predictions[predictedClass] * 100).toFixed(2),
          image: this.imagePreview,
        };

        this.history.unshift(result);
        this.saveHistory();
      } catch (error) {
        alert(error.message);
      } finally {
        this.loading = false;
      }
    },
    saveHistory() {
      if (import.meta.client) {
        localStorage.setItem("uploadHistory", JSON.stringify(this.history));
      }
    },
    loadHistory() {
      if (import.meta.client) {
        this.history = JSON.parse(localStorage.getItem("uploadHistory")) || [];
      }
    },
    clearHistory() {
      if (import.meta.client) {
        localStorage.removeItem("uploadHistory");
        this.history = [];
        alert("Histórico limpo com sucesso!");
      }
    },
  },
};
</script>

