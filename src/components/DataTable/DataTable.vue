<template>
  <div class="datatable">
    <div class="datatable-body">
      <div
        :class="{
          'datatable-header': !checkedRows.length,
          'datatable-header-resp': checkedRows.length,
        }"
      >
        <div
          v-if="data.length || (!data.length && hasPagination)"
          :aria-label="`${tableReference} data table search`"
          role="search"
          class="datatable-search"
        >
          <div class="datatable-search-icon">
            <i class="bi bi-search"></i>
          </div>
          <input
            :id="idSearch"
            :aria-labelledby="idSearch"
            v-model="search"
            :placeholder="searchPlaceholder"
            class="datatable-search-input"
            type="search"
            aria-label="input to search table"
            @input="searchTable()"
          />{{ search }}
        </div>
        <div class="datatable-remove-all" v-if="isAllSelected">
          <button type="button" class="btn btn-light btn-sm" @click="removeAll">
            <i class="bi bi-trash"> Excluir todos clientes </i>
          </button>
        </div>
        <div
          v-if="hasButtonDownload && data.length"
          @click="downloadContent"
          class="datatable-add-button"
        >
          <button type="button" class="btn btn-light btn-sm">
            <i class="bi bi-download"> Fazer download</i>
          </button>
        </div>
        <div
          v-if="showButtonAddNew"
          @click="createNew"
          class="datatable-add-button"
        >
          <button type="button" class="btn btn-light btn-sm">
            <i class="bi bi-plus-circle"> Novo cliente</i>
          </button>
        </div>
      </div>
      <div class="datatable-content" tabindex="0">
        <table class="datatable-table">
          <thead class="datatable-table-thead">
            <tr>
              <th class="datatable-checkbox-all">
                <input
                  v-if="data.length"
                  v-model="isAllSelected"
                  type="checkbox"
                  aria-label="checkbox"
                  @click="selectAll()"
                />
              </th>
              <th v-for="(item, index) in columns" :key="index">
                <span>
                  {{ item }}
                </span>
              </th>
            </tr>
          </thead>
          <tbody class="datatable-table-body">
            <tr
              v-for="(row, indexRow) in pagination[pageNumber - 1]"
              :key="indexRow"
            >
              <td></td>
              <td v-for="(col, indexCol) in row" :key="indexCol">
                <span>
                  {{ col }}
                </span>
              </td>
              <td class="datatable-action" @click="openModalEdit(row)">
                <i class="bi bi-pencil-square datatable-action-button"></i>
              </td>
            </tr>
          </tbody>
        </table>
        <div v-if="!data.length" class="datatable-no-data">
          Nenhum cliente encontrado.
        </div>
      </div>
    </div>
    <div class="datatable-nav">
      <div class="datatable-nav-left">
        <label
          :for="`${tableReference} page-items`"
          class="datatable-nav-left-item"
          >Mostar</label
        >
        <div class="datatable-nav-left-item">
          <select
            :id="`${tableReference} page-items`"
            v-model="itemsPerPage"
            :name="`${tableReference} page-items`"
            class="datatable-select-field"
            aria-label="Show"
          >
            <optgroup label="items por paginas">
              <option v-for="item in itemsPerPageList" :key="item">
                {{ item }}
              </option>
            </optgroup>
          </select>
        </div>
        <div class="datatable-nav-left-item">|</div>
        <div class="datatable-nav-left-item">
          {{ fromIndex }}-{{ toIndex }} de {{ data.length }} item
          <span v-show="getLenght > 1">s</span>
        </div>
      </div>
      <div class="datatable-nav-right">
        <label :for="`${tableReference} page`" class="datatable-nav-right-item">
          {{ pageNumber }} de {{ totalPages }}
          {{ totalPages > 1 ? " paginas" : " pagina" }}
        </label>
        <div class="datatable-nav-right-item">
          <div class="datatable-nav-right-select">
            <a
              v-if="pageNumber > 1"
              class="datatable-arrow-left"
              @click="changePage('<')"
              >&lt;</a
            >
            <select
              :id="`${tableReference} page`"
              v-model="pageNumber"
              :aria-label="
                pageNumber +
                ' of ' +
                totalPages +
                (totalPages > 1 ? ' pages' : ' page')
              "
              :name="`${tableReference} page`"
              class="datatable-select-field"
            >
              <optgroup label="Current page">
                <option v-for="index in totalPages" :key="index">
                  {{ index }}
                </option>
              </optgroup>
            </select>
            <a
              v-if="pageNumber < pagination.length || hasPagination"
              class="datatable-arrow-right"
              @click="changePage('>')"
              >&gt;</a
            >
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "DataTable",
  props: {
    columns: { type: Array, default: () => [] }, // array of columns
    data: { type: Array, default: () => [] },
    hasButtonAddNew: { type: String, default: null },
    hasEmitAddNew: { type: Boolean, default: false },
    hasButtonDownload: { type: Boolean, default: false },
    hasCheckbox: { type: Boolean, default: false },
    iconSearch: { type: String, default: "" },
    showIdLink: { type: Boolean, default: true },
    hasPagination: { type: Boolean, default: false },
    idSearch: { type: String, default: "search-input" },
    searchPlaceholder: { type: String, default: "Pesquisar" },
    tableReference: { type: String, default: "data-table" },
  },
  data() {
    return {
      itemsPerPageList: [10, 20, 30, 40, 50, 100],
      itemsPerPage: 10,
      pageNumber: 1,
      search: "",
      searchClients: [],
      pagination: [],
      checkedRows: [],
      hasActionColumn: false,
      searchOnFilters: [],
      isAllSelected: false,
    };
  },
  computed: {
    fromIndex() {
      const index =
        1 + (Number(this.pageNumber) - 1) * Number(this.itemsPerPage);
      if (this.search) {
        return 1;
      }
      return index;
    },
    toIndex() {
      const index = Number(this.fromIndex) + Number(this.itemsPerPage) - 1;
      let arraySize;
      if (this.search || this.searchOnFilters) {
        arraySize = this.getLenght;
      } else {
        arraySize = this.data.length;
      }
      if (index < arraySize) {
        return index;
      }
      return arraySize;
    },
    totalPages() {
      return this.pagination.length;
    },
    getLenght() {
      if (this.pagination.length) {
        return this.pagination[Number(this.pageNumber) - 1]
          ? this.pagination[Number(this.pageNumber) - 1].length
          : this.pagination[Number(this.pageNumber)];
      }
      return this.data.length;
    },
    showButtonAddNew() {
      return this.hasButtonAddNew || this.hasEmitAddNew;
    },
  },
  watch: {
    data() {
      this.setPagination(this.data);
    },
    itemsPerPage() {
      this.pagination = [];
      this.pageNumber = 1;
      this.search = "";
      this.setPagination(this.data);
    },
  },
  mounted() {
    this.setPagination(this.data);
  },
  methods: {
    setPagination(data) {
      this.pagination = [];
      const pages = Math.ceil(data.length / Number(this.itemsPerPage));
      for (let x = 0; x < pages; x++) {
        this.pagination.push([]);
        const initPos = x * Number(this.itemsPerPage);
        for (let y = initPos; y < initPos + Number(this.itemsPerPage); y++) {
          if (data[y]) {
            this.pagination[x].push(data[y]);
          } else {
            y = initPos + Number(this.itemsPerPage) + 1;
          }
        }
      }
    },
    changePage(signal) {
      if (signal === ">") {
        this.pageNumber++;
        if (this.hasPagination) {
          this.$emit("pagination", this.pageNumber, this.pagination);
        }
      } else {
        this.pageNumber--;
      }
    },
    searchTable() {
      this.checkedRows = [];
      this.searchClients = this.data.filter(
        (item) =>
          Object.values(item)
            .toString()
            .toUpperCase()
            .indexOf(this.search.toUpperCase()) !== -1
      );
      this.pagination = [];
      this.pageNumber = 1;
      this.setPagination(this.searchClients);

      if (this.hasPagination && !this.search && this.pageNumber > 1) {
        this.$emit("pagination", this.pageNumber, false);
      }
    },
    createNew() {
      this.$emit("add-new");
    },
    downloadContent() {
      this.$emit("downloadContent", true);
    },
    selectAll() {
      this.checkedRows = [];
      if (!this.isAllSelected && !this.search) {
        for (const i in this.data) {
          if (Object.prototype.hasOwnProperty.call(this.data, i)) {
            this.checkedRows.push(this.data[i].ID);
          }
        }
      }
    },
    removeAll() {
      this.$emit("remove-all");
    },
    removeClient(row) {
      this.$emit("remove-client", row);
    },
    openModalEdit(row) {
      this.$emit("open-edit", row);
    },
  },
};
</script>

<style lang="scss">
.datatable {
  display: flex;
  width: 100%;
  flex-direction: column;
}
.datatable-body {
  display: flex;
  background-color: $gray;
  flex-direction: column;
  border-radius: 0.6rem;
  width: 100%;
}
.datatable-header {
  display: flex;
  justify-content: flex-end;
  align-items: center;
}
.datatable-header > *:first-child {
  margin-right: auto;
}
.datatable-header-resp {
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-end;
  align-items: center;
  flex-direction: row;
}
.datatable-header-resp > *:first-child {
  margin-right: auto;
}
.datatable-search-icon {
  margin: 0.5rem;
}
.datatable-search {
  display: flex;
}
.datatable-search-input {
  border-style: none;
  margin: 0.5rem;
  height: $bigger;
  padding-left: 0.5rem;
}
.datatable-content {
  display: flex;
  background-color: $softblue-gray;
  border-radius: 0.6rem;
  padding-bottom: 1rem;
  overflow: auto;
  flex-direction: column;
}
.datatable-table {
  width: 100%;
  border-collapse: collapse;
}
.datatable-table-thead th {
  border-bottom: 0.2rem solid $gray-900 !important;
  font-size: 1rem;
  padding: 0.5rem;
  font-weight: bold;
  line-height: 1.25;
}
.datatable-table-body td {
  border-bottom: 0.1rem solid $softblue !important;
  font-size: 0.85rem;
  padding: 0.5rem;
  vertical-align: middle;
  line-height: 1.25;
}
.datatable-table-body tr:hover {
  background-color: $blue-101 !important;
}
.datatable-no-data {
  text-align: center;
  padding-top: 0.8rem;
}
.datatable-nav {
  font-size: 0.85rem;
  display: flex;
  background-color: $white;
  justify-content: space-between;
  padding-top: 0.5rem;
  flex-wrap: wrap;
}
.datatable-nav-left {
  display: flex;
  align-items: center;
  padding-bottom: 0.5rem;
}
.datatable-nav-left-item {
  display: flex;
  padding-left: 0.5rem;
}
.datatable-nav-right {
  display: flex;
  align-items: center;
  padding-bottom: 0.5rem;
}
.datatable-nav-right-item {
  display: flex;
  padding-right: 0.5rem;
}
.datatable-nav-right-select {
  display: flex;
  background-color: $white;
  border-radius: 0.3rem;
}
.datatable-arrow-left {
  background-color: $white;
  color: $softblue;
  padding-right: 0.2rem;
  padding-left: 0.2rem;
  border-style: solid;
  border-width: 0.1rem;
  border-color: $softblue;
  border-top-left-radius: 0.3rem;
  border-bottom-left-radius: 0.3rem;
  font-weight: bold;
}
.datatable-arrow-right {
  background-color: $white;
  color: $black;
  padding-right: 0.2rem;
  padding-left: 0.2rem;
  border-style: solid;
  border-width: 0.1rem;
  border-color: $black;
  border-top-right-radius: 0.3rem;
  border-bottom-right-radius: 0.3rem;
  font-weight: bold;
}
.datatable-remove-all {
  display: flex;
  margin: 0.3rem;
}
.datatable-remove-all-icon {
  margin: 0.1rem;
}
.datatable-add-button {
  display: flex;
  justify-content: end;
  border-radius: 1rem;
  margin: 0.5rem;
  color: $gray-900;
  align-items: center;
  cursor: pointer;
}
.datatable-action-button {
  margin-left: $medium;
  font-size: $medium;
  cursor: pointer;
}
.datatable-action {
  padding: 0 !important;
}
.datatable-select-field {
  border-radius: 0.2rem;
  color: $white;
  padding: 0.2rem;
  width: 3rem;
  background-size: 5px 8px, 5px 8px, 2em 2em;
  background-repeat: no-repeat;
  cursor: pointer;
}
.datatable-select-field option {
  background-color: $white;
  color: $white;
}
.datatable-select-field option:checked {
  background-color: $softblue;
  color: $white;
}
.datatable-text-limit {
  display: block;
  max-width: calc(100vw - 53vw);
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
.datatable-remove-limit {
  overflow: scroll;
  text-overflow: unset;
}
.datatable-button {
  background-color: $softblue;
  color: $gray-900;
  border-color: $softblue;
  font-size: 1rem;
  height: $medium;
  padding: 0 0.25rem 0 0.25rem;
}
.datatable-button:hover,
.datatable-button:focus {
  background-color: $gray;
  color: $white;
}
input {
  margin-right: 0.3rem;
}
.open-modal-edit,
.open-event {
  text-decoration: underline;
  text-decoration: solid;
  cursor: pointer;
}
@media screen and (max-width: 1280px) {
  .datatable-header-resp {
    justify-content: flex-start;
    align-items: center;
  }
}
@media screen and (max-width: 500px) {
  .datatable-search {
    display: flex;
    flex-wrap: wrap;
  }
  .datatable-filter {
    margin: 0 0.5rem 0.5rem;
  }
  .datatable-nav {
    font-size: 0.7rem;
  }
  .datatable-table-thead th {
    font-size: 0.8rem;
  }
}
</style>
