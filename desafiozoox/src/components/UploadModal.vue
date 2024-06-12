<template>
  <div v-if="isOpen" class="modal-overlay" @click.self="close">
    <div 
      class="modal-content" 
      @dragover.prevent 
      @dragenter.prevent 
      @drop.prevent="handleFileDrop"
    >
      <h2>Escolha Um Arquivo A Ser Importado</h2>
      <div v-if="selectedFile" class="file-info">
        <p>{{ selectedFile.name }}</p>
        <div class="progress-bar">
          <div class="progress" :style="{ width: uploadProgress + '%' }"></div>
        </div>
        <p>Envio do Arquivo: {{ uploadProgress }}%</p>
      </div>
      <div v-else class="upload-area">
        <input 
          ref="fileInput" 
          type="file" 
          @change="handleFileUpload" 
          accept=".csv, .xlsx" 
          hidden
        >
        <p>Arraste um arquivo CSV, XLSX até aqui</p>
        <p>Ou se preferir</p>
        <button @click="triggerFileUpload" class="button_upload">Selecione um arquivo do seu dispositivo</button>
      </div>
      <div class="div_button_cancel">
        <button @click="close" class="button_cancel">Cancelar</button>
      </div>
    </div>
  </div>
</template>

<script>
import Papa from 'papaparse';

export default {
  props: {
    isOpen: {
      type: Boolean,
      required: true
    }
  },
  data() {
    return {
      selectedFile: null,
      uploadProgress: 0
    };
  },
  methods: {
    triggerFileUpload() {
      this.$refs.fileInput.click();
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      this.selectedFile = file;
      this.uploadFile(file);
    },
    handleFileDrop(event) {
      const file = event.dataTransfer.files[0];
      this.selectedFile = file;
      this.uploadFile(file);
    },
    uploadFile(file) {
      const formData = new FormData();
      formData.append('file', file);

      Papa.parse(file, {
        header: true,
        complete: (results) => {
          this.uploadProgress = 100;
          this.$emit('file-uploaded', results.data);
          this.resetUpload(); 
          this.$emit('close'); 
        }
      });

      const uploadInterval = setInterval(() => {
        if (this.uploadProgress < 100) {
          this.uploadProgress += 10;
        } else {
          clearInterval(uploadInterval);
        }
      }, 200);
    },
    close() {
      this.$emit('close');
      this.resetUpload();
    },
    resetUpload() {
      this.selectedFile = null;
      this.uploadProgress = 0;
    }
  }
};
</script>

<style>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  padding: 20px;
  border-radius: 10px;
  text-align: center;
  background-color: #ffffff;
}

.upload-area {
  border: 2px dashed #666666;
  padding: 40px;
  margin: 20px 0;
  border-radius: 10px;
}

.file-info {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.progress-bar {
  width: 100%;
  background: #e0e0e0;
  border-radius: 5px;
  overflow: hidden;
  margin-top: 10px;
}

.progress {
  height: 5px;
  background: #2196f3;
}

button {
  margin-top: 10px;
}

.button_upload {
  border: 3px solid #0273c3;
  text-transform: uppercase;
  padding: 6px;
  gap: 10px;
  border-radius: 20px;
  color: #057bc7;
  cursor: pointer;
  background-color: #ffffff;
}

.button_cancel {
  border: 3px solid #333333;
  padding: 6px;
  gap: 10px;
  border-radius: 20px;
  color: #333333;
  cursor: pointer;
  background-color: #ffffff;
}

.div_button_cancel {
  display: flex;
  flex-direction: row;
  justify-content: flex-end;
}
</style>
