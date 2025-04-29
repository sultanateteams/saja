<template>
  <div class="app">
    <h2>📷 Rasm yuklang</h2>
    <input type="file" @change="onImageChange" />

    <div v-if="imageSrc">
      <img
        :src="imageSrc"
        alt="Yuklangan rasm"
        style="max-width: 300px; margin-top: 10px"
      />
    </div>

    <div v-if="result">
      <h3>🔍 Natija:</h3>
      <pre>{{ result }}</pre>
    </div>
  </div>
</template>

<script>
import Tesseract from "tesseract.js";
import Quagga from "quagga";

export default {
  data() {
    return {
      imageSrc: null,
      result: null,
    };
  },
  methods: {
    onImageChange(event) {
      const file = event.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = (e) => {
        this.imageSrc = e.target.result;
        this.analyzeImage(this.imageSrc);
      };
      reader.readAsDataURL(file);
    },

    async analyzeImage(image) {
      this.result = { barcode: null, productCode: null };

      // 1. OCR orqali CODExxxxCODE ni o‘qish
      const ocr = await Tesseract.recognize(image, "eng");
      const text = ocr.data.text;
      console.log("text:  ", text);
      const match = text.match(/SAJA(\d+)SAJA/);
      this.result.productCode = match ? match[1] : "Topilmadi";

      // 2. Barcode o‘qish
      Quagga.decodeSingle(
        {
          src: image,
          numOfWorkers: 0,
          inputStream: {
            size: 800,
          },
          decoder: {
            readers: ["code_128_reader", "ean_reader"], // kerakli formatga qarab sozlang
          },
        },
        (data) => {
          if (data && data.codeResult) {
            this.result.barcode = data.codeResult.code;
          } else {
            this.result.barcode = "Topilmadi";
          }
        }
      );
    },
  },
};
</script>

<style scoped>
.app {
  max-width: 500px;
  margin: auto;
  padding: 20px;
  font-family: Arial;
}
</style>
