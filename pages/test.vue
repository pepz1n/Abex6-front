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
          :src="'data:image/jpeg;base64,' + entry.image"
          max-height="50"
          max-width="50"
          class="mr-3"
        />
        <v-list-item-content>
          <v-list-item-title>{{ entry.label.split(' ')[0] + classes[entry.label.split(' ')[1]] }}</v-list-item-title>
          <v-list-item-subtitle>{{ String(entry.percentage*100).split('.')[0] }}%</v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
    </v-list>
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
    this.loadUserImages();
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
        await this.$api.post("/predict/predict", formData, {
          headers: { "Content-Type": "multipart/form-data" },
        });

        await this.loadUserImages();
      } catch (error) {
        alert(error.message);
      } finally {
        this.loading = false;
      }
    },
    
    
    async loadUserImages() {
      try {
        const response = await this.$api.get("/auth/get-image-token");
        this.history = response.images.map((image) => ({
          label: image.label,
          percentage: image.percentage,
          image: image.image,
        }));
      } catch (error) {
        console.error("Erro ao carregar imagens:", error);
      }
    },
  },
};
</script>
