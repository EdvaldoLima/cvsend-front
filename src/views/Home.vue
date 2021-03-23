<template>
  <v-container>
    <v-tabs v-model="tab">
      <v-tab>Formulário</v-tab>
      <v-tab @click="listarEmails()">E-mails enviados</v-tab>
    </v-tabs>
    <v-tabs-items v-model="tab">
      <v-tab-item>
        <v-col cols="12">
          <v-row>
            <v-alert border="left" color="red" v-model="errosShow" dark>
              <ul>
                <li v-for="erro in erros" :key="erro">{{ erro }}</li>
              </ul>
            </v-alert>
          </v-row>
        </v-col>
        <v-col>
          <v-row>
            <v-text-field v-model="cadastro.nome" label="Nome"></v-text-field>
          </v-row>
          <v-row>
            <v-text-field
              v-model="cadastro.email"
              type="email"
              label="E-mail"
            ></v-text-field>
          </v-row>
          <v-row>
            <v-text-field
              v-mask="'(##) #####-####'"
              v-model="cadastro.telefone"
              type="phone"
              label="Telefone"
              required
            ></v-text-field>
          </v-row>
          <v-row>
            <v-text-field
              v-model="cadastro.cargo"
              label="Cargo Dejesado"
            ></v-text-field>
          </v-row>
          <v-row>
            <v-select
              v-model="cadastro.escolaridade"
              :items="escolaridades"
              item-text="Escolaridade"
              item-value="Escolaridade"
              label="Escolaridade"
            ></v-select>
          </v-row>
          <v-row>
            <v-textarea
              v-model="cadastro.observacao"
              label="Observações"
            ></v-textarea>
          </v-row>
          <v-row>
            <v-file-input
              v-model="cadastro.arquivo"
              accept=".doc, .docx, .pdf"
              label="Currículo"
            ></v-file-input>
          </v-row>
          <v-row>
            <v-btn
              :loading="loadingSend"
              @click="enviarEmail()"
              color="blue"
              large
              block
              dark
            >
              <v-icon left>mdi-send</v-icon>
              enviar
            </v-btn>
          </v-row>

          <v-row>
            <v-dialog width="50%" v-model="dialog">
              <v-card>
                <v-card-title>Seu e-mail foi enviado</v-card-title>
                <v-card-text>
                  <p>
                    {{ mensagemSucesso }}
                  </p>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn
                    @click="[(dialog = !dialog), listarEmails()]"
                    color="blue"
                    dark
                    large
                  >
                    ok
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-row>
        </v-col>
      </v-tab-item>
      <v-tab-item>
        <v-text-field
          v-model="search"
          prepend-icon="mdi-magnify"
          label="Pesquisar"
        ></v-text-field>
        <v-data-table :search="search" :headers="headers" :items="emails">
          <template v-slot:[`item.data_envio`]="{ item }">
            {{ item.data_envio }}
          </template>
        </v-data-table>
      </v-tab-item>
    </v-tabs-items>
  </v-container>
</template>

<script>
// @ is an alias to /src

export default {
  name: "Home",
  data: () => ({
    dialog: false,
    tab: null,
    search: null,
    headers: [
      {
        text: "Nome",
        align: "center",
        value: "nome",
      },
      {
        text: "Email",
        align: "center",
        value: "email",
      },
      {
        text: "Telefone",
        align: "center",
        value: "telefone",
      },
      {
        text: "Escolaridade",
        align: "center",
        value: "escolaridade",
      },
      {
        text: "Observação",
        align: "center",
        value: "observacao",
      },
      {
        text: "Arquivo",
        align: "center",
        value: "arquivo",
      },
      {
        text: "IP",
        align: "center",
        value: "ip",
      },
      {
        text: "Data de Envio",
        align: "center",
        value: "data_envio",
      },
    ],
    escolaridades: [
      { Escolaridade: "ENSINO FUNDAMENTAL COMPLETO" },
      { Escolaridade: "ENSINO FUNDAMENTAL INCOMPLETO" },
      { Escolaridade: "ENSINO MÉDIO COMPLETO" },
      { Escolaridade: "ENSINO MÉDIO INCOMPLETO" },
      { Escolaridade: "ENSINO SUPERIOR COMPLETO" },
      { Escolaridade: "ENSINO SUPERIOR INCOMPLETO" },
      { Escolaridade: "MESTRADO" },
      { Escolaridade: "DOUTORADO" },
      { Escolaridade: "ESPECIALIZAÇÃO" },
    ],
    cadastro: {
      nome: null,
      email: null,
      telefone: null,
      cargo: null,
      escolaridade: null,
      observacao: null,
      arquivo: null,
    },
    emails: null,
    loadingSend: false,
    mensagemSucesso: null,
    erros: [],
    errosShow: false,
  }),
  methods: {
    enviarEmail() {
      this.loadingSend = true;
      let formData = new FormData();
      // O trecho de código a seguir não é nem um pouco usual, mas resolveu um problema com o validator do lumen
      if (this.cadastro.nome) {
        formData.append("nome", this.cadastro.nome);
      }
      if (this.cadastro.email) {
        formData.append("email", this.cadastro.email);
      }
      if (this.cadastro.telefone) {
        formData.append("telefone", this.cadastro.telefone);
      }
      if (this.cadastro.cargo) {
        formData.append("cargo", this.cadastro.cargo);
      }
      if (this.cadastro.escolaridade) {
        formData.append("escolaridade", this.cadastro.escolaridade);
      }
      if (this.cadastro.observacao) {
        formData.append("observacao", this.cadastro.observacao);
      }
      if (this.cadastro.arquivo) {
        formData.append("arquivo", this.cadastro.arquivo);
      }

      let headers = {
        "Content-Type": "multipart/form-data",
      };

      this.axios
        .post("emails", formData, headers)
        .then((response) => {
          this.mensagemSucesso = response.data;
          this.dialog = true;
          this.cadastro = {
            nome: null,
            email: null,
            telefone: null,
            cargo: null,
            escolaridade: null,
            observacao: null,
            arquivo: null,
          };
          (this.erros = []), (this.errosShow = false);
        })
        .catch((error) => {
          let errosResponse = Object.keys(error.response.data)
            .map((key) => {
              return error.response.data[key];
            })
            .flat();
          this.erros = errosResponse;
          this.errosShow = true;
        })
        .finally(() => {
          this.loadingSend = false;
        });
    },
    listarEmails() {
      this.axios.get("emails").then((response) => {
        this.emails = response.data;
      });
    },
  },
  mounted() {
    this.listarEmails();
  },
};
</script>
