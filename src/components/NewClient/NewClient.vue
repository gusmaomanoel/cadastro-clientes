<template>
  <div class="modal">
    <div class="modal-dialog modal-dialog-centered modal-lg">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="exampleModalLabel" v-if="!itemData">
            Novo Cliente
          </h5>
          <h5 class="modal-title" id="exampleModalLabel" v-if="itemData">
            Editar Cliente
          </h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
            @click="close"
          ></button>
        </div>
        <div class="modal-body">
          <form class="row g-3 needs-validation">
            <div class="col-md-6">
              <InputField
                label="Primeiro nome"
                :required="!data.NOME && validated"
                v-model="data.NOME"
              />
            </div>
            <div class="col-md-6">
              <InputField
                label="Sobrenome"
                :required="!data.SOBRENOME && validated"
                v-model="data.SOBRENOME"
              />
            </div>
            <div class="col-md-6">
              <InputField
                type="date"
                label="Data de nascimento"
                v-model="data.NASCIMENTO"
              />
            </div>
            <div class="col-md-6">
              <InputField
                label="E-mail"
                type="email"
                :required="!data.EMAIL && validated"
                :invalidEmail="invalidEmail"
                v-model="data.EMAIL"
              />
            </div>
            <div class="col-md-6">
              <InputField label="Cidade" v-model="data.CIDADE" />
            </div>
            <div class="col-md-6">
              <InputField label="Estado" v-model="data.ESTADO" />
            </div>
            <div class="col-md-6">
              <InputField label="Complemento" v-model="data.COMPLEMENTO" />
            </div>
            <div class="col-md-6">
              <InputField
                label="Telefone"
                v-model="data.TELEFONE"
                :maxLength="15"
                @mask-cel-phone="maskCelphone($event)"
              />
            </div>
          </form>
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-primary"
            @click="save"
            v-if="!this.itemData"
          >
            Salvar cliente
          </button>
          <button
            type="button"
            class="btn btn-primary"
            @click="saveUpdates"
            v-if="this.itemData"
          >
            Salvar alterações
          </button>
          <button
            type="button"
            class="btn btn-secondary"
            @click="removeClient"
            v-if="this.itemData"
          >
            Excluir cliente
          </button>
          <button
            type="button"
            class="btn btn-secondary"
            data-bs-dismiss="modal"
            @click="close"
          >
            Fechar
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import InputField from "../InputField/InputField.vue";
export default {
  name: "NewClient",
  components: {
    InputField,
  },
  props: {
    itemData: { type: Object, default: null },
  },
  data() {
    return {
      showLabel: false,
      items: [],
      showClientModal: false,
      validated: false,
      data: {},
      invalidEmail: false,
    };
  },
  mounted() {
    if (this.itemData) {
      this.data = this.itemData;
    }
  },
  methods: {
    save() {
      const emailRegex =
        /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
      if (this.data.EMAIL && !emailRegex.test(this.data.EMAIL.toLowerCase())) {
        return (this.invalidEmail = true);
      } else {
        this.invalidEmail = false;
      }
      if (this.data.NOME && this.data.SOBRENOME && this.data.EMAIL) {
        const currentItems = localStorage.getItem("clients");
        if (currentItems && currentItems.length) {
          this.items = JSON.parse(localStorage.getItem("clients"));
        }
        let id = this.items.length;
        id++;
        this.items.push({
          ID: id,
          NOME: this.data.NOME,
          SOBRENOME: this.data.SOBRENOME,
          NASCIMENTO: this.data.NASCIMENTO ? this.data.NASCIMENTO : "",
          EMAIL: this.data.EMAIL,
          CIDADE: this.data.CIDADE ? this.data.CIDADE : "",
          ESTADO: this.data.ESTADO ? this.data.ESTADO : "",
          COMPLEMENTO: this.data.COMPLEMENTO ? this.data.COMPLEMENTO : "",
          TELEFONE: this.data.TELEFONE ? this.data.TELEFONE : "",
        });
        this.saveDataLocalStorage();
        window.location.reload();
        this.close();
      } else {
        this.validated = true;
      }
    },
    saveDataLocalStorage() {
      const parsed = JSON.stringify(this.items);
      localStorage.setItem("clients", parsed);
    },
    saveUpdates() {
      const emailRegex =
        /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
      if (
        this.data.EMAIL &&
        this.data.EMAIL.length &&
        !emailRegex.test(this.data.EMAIL.toLowerCase())
      ) {
        return (this.invalidEmail = true);
      } else {
        this.invalidEmail = false;
      }
      if (
        this.data.NOME &&
        this.data.NOME.length &&
        this.data.EMAIL &&
        this.data.EMAIL.length
      ) {
        const clientSave = JSON.parse(localStorage.getItem("clients"));

        const clientExist = clientSave.find((c) => c.ID === this.itemData.ID);
        Object.assign(clientExist, this.itemData);

        const clientExistIndex = clientSave.findIndex(
          (c) => c.ID === this.itemData.ID
        );

        if (clientExistIndex !== -1) {
          clientSave[clientExistIndex] = this.itemData;
        }
        localStorage.setItem("clients", JSON.stringify(clientSave));
        window.location.reload();
        this.close();
      } else {
        this.validated = true;
      }
    },
    close() {
      this.$emit("close");
      window.location.reload();
    },

    maskCelphone(event) {
      let tecla = event.key;
      let telefone = event.target.value.replace(/\D+/g, "");

      if (/^[0-9]$/i.test(tecla)) {
        let tamanho = telefone.length;

        if (tamanho >= 12) {
          return false;
        }

        if (tamanho > 10) {
          telefone = telefone.replace(/^(\d\d)(\d{5})(\d{4}).*/, "($1) $2-$3");
        } else if (tamanho > 5) {
          telefone = telefone.replace(
            /^(\d\d)(\d{4})(\d{0,4}).*/,
            "($1) $2-$3"
          );
        } else if (tamanho > 2) {
          telefone = telefone.replace(/^(\d\d)(\d{0,5})/, "($1) $2");
        } else {
          telefone = telefone.replace(/^(\d*)/, "($1");
        }

        event.target.value = telefone;
      }
    },

    removeClient() {
      let allClients = JSON.parse(localStorage.getItem("clients"));
      const clientToRemove = allClients.find((c) => c.ID === this.itemData.ID);
      if (clientToRemove !== -1) {
        allClients = allClients.splice(clientToRemove, 1);
        localStorage.setItem("clients", JSON.stringify(allClients));
        window.location.reload();
        this.close();
      }
    },
  },
};
</script>
<style>
.modal {
  display: flex;
  position: fixed;
  top: 0;
  left: 0;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  width: 100%;
  flex-flow: column;
  min-height: calc(100vh - 9.9rem);
  transition: opacity 0.3s ease;
}

.modal-lg {
  width: 100%;
}
</style>
