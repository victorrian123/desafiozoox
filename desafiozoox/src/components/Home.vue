<template>
  <div class="container">
    <div class="container-all-code">
      <div class="breadcrumbs">
        <p class="title">Início</p>
        <span v-if="tableData.length > 0" class="separator">></span>
        <p v-if="tableData.length > 0" class="breadcrumb active">Detalhe do CSV</p>
      </div>

      <div class="values-on-tabled">
        <div>Seus Arquivos</div>
        <div v-if="tableData.length === 0" class="button_csv" @click="openModal">
          <svg width="15px" height="15px" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
            <line fill="none" stroke="#000000" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" x1="12" x2="12" y1="19" y2="5" />
            <line fill="none" stroke="#000000" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" x1="5" x2="19" y1="12" y2="12" />
          </svg>
          Importar CSV
        </div>
        <div v-else class="button-group">
          <div class="button_csv" @click="exportSelected">
            <svg width="15px" height="15px" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" fill="none">
              <path stroke="#000000" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 5v8.5M15 7l-3-3-3 3m-4 5v5a2 2 0 002 2h10a2 2 0 002-2v-5" />
            </svg>
            Exportar Tabela
          </div>
          <div class="button_csv" @click="resetTable">
            <svg width="15px" height="15px" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
              <path fill="none" stroke="#000000" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 12H5M12 19l-7-7 7-7" />
            </svg>
            Voltar
          </div>
        </div>
        <UploadModal :isOpen="isModalOpen" @close="closeModal" @file-uploaded="handleFileUploaded" />
      </div>
      <div class="table-created">
        <div v-if="tableData.length > 0" class="table-group-export">
          <table>
            <thead>
              <tr>
                <th class="checkbox-column">
                  <input type="checkbox" @change="selectAll($event)" />
                </th>
                <th v-for="(header, index) in tableHeaders" :key="index">
                  {{ header }}
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(row, rowIndex) in tableData" :key="rowIndex">
                <td class="checkbox-column">
                  <input type="checkbox" v-model="selectedRows" :value="rowIndex" />
                </td>
                <td v-for="(header, colIndex) in tableHeaders" :key="colIndex">
                  {{ row[header] }}
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <div v-else-if="exportLogs.length > 0" class="table-group-export">
          <table>
            <thead>
              <tr>
                <th>Nome</th>
                <th>Número De Colunas</th>
                <th>Número De Linhas</th>
                <th>Exportado</th>
                <th>Criado Em</th>
                <th>Download</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(log, logIndex) in exportLogs" :key="logIndex">
                <td>{{ log.nome }}</td>
                <td>{{ log.colunas }}</td>
                <td>{{ log.linhas }}</td>
                <td>{{ log.exportado }}</td>
                <td>{{ log.criado }}</td>
                <td>
                  <a :href="log.link" download>
                    <svg width="25px" height="25px" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                      <path fill="none" stroke="#000000" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 5v8.5M15 7l-3-3-3 3m-4 5v5a2 2 0 002 2h10a2 2 0 002-2v-5" />
                    </svg>
                  </a>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <div v-else class="table-empty">
          <p>Sem tabelas importadas</p>
          <p>Importe clicando no botão acima "Importar csv" para começar</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import UploadModal from "./UploadModal.vue";
import Papa from "papaparse";

export default {
  components: {
    UploadModal,
  },
  data() {
    return {
      isModalOpen: false,
      tableData: [],
      tableHeaders: [],
      selectedRows: [],
      exportLogs: [],
    };
  },
  mounted() {
    this.loadExportLogs();
  },
  methods: {
    openModal() {
      this.isModalOpen = true;
    },
    closeModal() {
      this.isModalOpen = false;
    },
    handleFileUploaded(data) {
      const filteredData = data.filter((row) => {
        return Object.values(row).some(
          (value) => value !== null && value !== undefined && value !== "",
        );
      });

      if (filteredData.length > 0) {
        this.tableHeaders = Object.keys(filteredData[0]);
      } else {
        this.tableHeaders = [];
      }
      this.tableData = filteredData;
    },
    resetTable() {
      this.tableData = [];
      this.tableHeaders = [];
      this.selectedRows = [];
    },
    exportSelected() {
      const selectedData = this.tableData
        .filter((_, index) => this.selectedRows.includes(index))
        .filter((row) => {
          return Object.values(row).some(
            (value) => value !== null && value !== undefined && value !== "",
          );
        });

      const csv = Papa.unparse(selectedData);
      const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
      const link = document.createElement("a");
      const filename = `tabela_exportada_${new Date().toISOString()}.csv`;
      link.href = URL.createObjectURL(blob);
      link.setAttribute("download", filename);
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      this.saveExportLog(filename, selectedData);
    },
    selectAll(event) {
      if (event.target.checked) {
        this.selectedRows = this.tableData.map((_, index) => index);
      } else {
        this.selectedRows = [];
      }
    },
    saveExportLog(filename, data) {
      const log = {
        nome: filename,
        colunas: `${this.tableHeaders.length}`,
        linhas: `${data.length}`,
        exportado: `Sim - ${data.length} linhas e ${this.tableHeaders.length} colunas`,
        criado: new Date().toLocaleDateString(),
        link: URL.createObjectURL(new Blob([Papa.unparse(data)], { type: "text/csv;charset=utf-8;" }))
      };

      let logs = JSON.parse(localStorage.getItem('exportLogs')) || [];
      logs.push(log);
      localStorage.setItem('exportLogs', JSON.stringify(logs));
      this.loadExportLogs();
    },
    loadExportLogs() {
      const logs = JSON.parse(localStorage.getItem('exportLogs')) || [];
      this.exportLogs = logs;
    }
  },
};
</script>

<style>
.container {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 20px;
  padding: 50px;
}

.breadcrumbs {
  display: flex;
  align-items: center;
  font-size: 14px;
}

.breadcrumb {
  color: #7ebcf2;
  text-decoration: none;
}

.breadcrumb:hover {
  text-decoration: underline;
}

.separator {
  margin: 0 8px;
  color: #6c757d;
}

.breadcrumb.active {
  color: #007dc6;
  text-decoration: none;
  pointer-events: none;
}

.title {
  color: #7ebcf2;
  font-weight: bold;
}

.button_csv {
  border: 1px solid #333333;
  padding: 3px;
  gap: 5px;
  border-radius: 20px;
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #333333;
  cursor: pointer;
}

.container-all-code {
  display: flex;
  flex-direction: column;
  width: 100%;
}

.values-on-tabled {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
}

.table-created {
  background-color: #ffffff;
  box-shadow: 0px 6px 36px rgba(112, 144, 176, 0.09);
  border-radius: 20px;
  width: 100%;
  height: 500px;
  margin-top: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.table-empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: #ffffff;
}

.table-group-export {
  width: 100%;
  height: 100%;
  overflow-x: auto;
}

.item-table {
  display: flex;
  flex-direction: row;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background-color: #ffffff;
}

th {
  padding: 10px;
  text-align: left;
  border: 1px solid #ddd;
}

td {
  padding: 10px;
  text-align: left;
  border: 1px solid #ddd;
}

th {
  font-weight: bold;
}

tbody tr:hover {
  background-color: #f1f1f1;
}

.button-group {
  display: flex;
  flex-direction: row;
  gap: 10px;
}

.checkbox-column {
  text-align: center;
}

.table-group-export::-webkit-scrollbar {
  width: 10px;
  height: 10px;
}

.table-group-export::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
}

.table-group-export::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 10px;
}

.table-group-export::-webkit-scrollbar-thumb:hover {
  background: #555;
}

</style>
