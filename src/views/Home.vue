<template>
  <div class="home">
    <NewClient
      v-if="showClientModal"
      @close="openClientModal"
      :itemData="selectedRow"
    />
    <div class="home-datatable">
      <DataTable
        :columns="[
          'ID',
          'Nome',
          'Sobrenome',
          'Data nascimento',
          'E-mail',
          'Cidade',
          'Estado',
          'Complemento',
          'Telefone',
          'Editar',
        ]"
        :data="items"
        :hasEmitAddNew="true"
        :selectEntireRow="true"
        :hasButtonDownload="true"
        @add-new="openClientModal"
        @remove-all="removeAllItems"
        @open-edit="openModalEdit"
        @downloadContent="downloadAllClients"
      />
    </div>
  </div>
</template>
<script>
import DataTable from "../components/DataTable/DataTable.vue";
import NewClient from "../components/NewClient/NewClient.vue";
export default {
  name: "HomeComponent",
  components: {
    DataTable,
    NewClient,
  },
  data() {
    return {
      showLabel: false,
      items: [],
      showClientModal: false,
      selectedRow: null,
    };
  },
  mounted() {
    if (localStorage.getItem("clients")) {
      this.items = JSON.parse(localStorage.getItem("clients"));
    }
  },
  methods: {
    openClientModal() {
      this.showClientModal = !this.showClientModal;
    },

    removeAllItems() {
      localStorage.removeItem("clients");
      window.location.reload();
    },

    openModalEdit(row) {
      this.openClientModal();
      this.selectedRow = row;
    },

    downloadAllClients() {
      let texto = "";

      let cabecalho = Object.keys(this.items[0]);
      texto += cabecalho.join("\t") + "\n";

      this.items.forEach(function (item) {
        let linha = cabecalho.map(function (chave) {
          return item[chave];
        });
        texto += linha.join("\t") + "\n";
      });

      let link = document.createElement("a");
      link.href = "data:text/plain;charset=utf-8," + encodeURIComponent(texto);
      link.download = "clientes.csv";
      link.click();
    },
  },
};
</script>

<style lang="scss">
.home {
  display: flex;
  flex-direction: column;
  width: 100%;
  justify-content: center;
  background-color: $blue-101;
}

.home-datatable {
  margin: 0;
  padding: 0;
  padding-left: $medium;
  padding-right: $medium;
}
</style>
