<script setup lang="ts">
import { ref, computed } from 'vue';
import axios from 'axios';

// URL base do backend
const baseURL = 'https://8080-nicolaspereira-provavue-p0j09uuxq5y.ws-us116.gitpod.io/vocabulo';

// Lista principal de vocábulos
const vocabulos = ref<any[]>([]);

// Campos para consulta
const termoConsulta = ref('');
const versaoConsulta = ref<number | null>(null);

// Mensagem de erro/resultado vazio
const mensagemErro = ref('');

// Novo vocábulo para cadastro
const novoVocabulo = ref({
  id: null,
  termo: '',
  significado: '',
  versao: null,
  dataHoraDesativacao: null,
});

// Erros de validação para o formulário
const erros = ref({
  termo: '',
  significado: '',
});

// Função para buscar todos os vocábulos ao carregar
const carregarVocabulos = async () => {
  try {
    const response = await axios.get(`${baseURL}`);
    vocabulos.value = response.data;
    mensagemErro.value = ''; // Limpa mensagem de erro ao obter dados
  } catch (error) {
    mensagemErro.value = 'Erro ao carregar os vocábulos.';
    console.error(error);
  }
};

// Função para adicionar situação "ativo/desativado" à lista de vocábulos
const vocabulosComSituacao = computed(() =>
  vocabulos.value.map((vocabulo) => ({
    ...vocabulo,
    situacao: vocabulo.dataHoraDesativacao ? 'Desativado' : 'Ativo',
  }))
);

// Função para cadastrar um novo vocábulo
const cadastrarVocabulo = async () => {
  erros.value = { termo: '', significado: '' };

  if (!novoVocabulo.value.termo) erros.value.termo = 'O campo termo é obrigatório.';
  if (!novoVocabulo.value.significado)
    erros.value.significado = 'O campo significado é obrigatório.';

  if (erros.value.termo || erros.value.significado) return;

  try {
    await axios.post(`${baseURL}`, novoVocabulo.value);
    carregarVocabulos(); // Atualiza a lista
    // Limpa os inputs
    novoVocabulo.value = {
      id: null,
      termo: '',
      significado: '',
      versao: null,
      dataHoraDesativacao: null,
    };
  } catch (error) {
    console.error('Erro ao cadastrar vocábulo:', error);
  }
};

// Função para buscar vocábulos por termo e versão
const buscarVocabulos = async () => {
  if (!termoConsulta.value || versaoConsulta.value === null) {
    mensagemErro.value = 'Por favor, preencha o termo e a versão.';
    return;
  }

  try {
    const response = await axios.get(
      `${baseURL}/${encodeURIComponent(termoConsulta.value)}/${versaoConsulta.value}`
    );
    if (response.data.length > 0) {
      vocabulos.value = response.data;
      mensagemErro.value = ''; // Limpa mensagem de erro se resultados forem encontrados
    } else {
      vocabulos.value = [];
      mensagemErro.value = 'Nenhum vocábulo foi encontrado para os critérios fornecidos.';
    }
  } catch (error) {
    mensagemErro.value = 'Erro ao buscar vocábulo.';
    console.error(error);
  }
};

// Carrega os vocábulos ao abrir a página
carregarVocabulos();
</script>

<template>
  <div>
    <!-- Consulta por termo e versão -->
    <h2>Buscar Vocábulo</h2>
    <form @submit.prevent="buscarVocabulos">
      <div>
        <label for="termoConsulta">Termo:</label>
        <input id="termoConsulta" v-model="termoConsulta" type="text" />
      </div>
      <div>
        <label for="versaoConsulta">Versão:</label>
        <input id="versaoConsulta" v-model.number="versaoConsulta" type="number" />
      </div>
      <button type="submit">Buscar</button>
    </form>

    <!-- Mensagem de erro ou sucesso -->
    <p v-if="mensagemErro" style="color: red;">{{ mensagemErro }}</p>

    <!-- Lista de vocábulos -->
    <h2>Lista de Vocábulos</h2>
    <table v-if="vocabulos.length > 0" border="1" cellspacing="0" cellpadding="8" style="width: 100%; text-align: left;">
      <thead>
        <tr>
          <th>ID</th>
          <th>Termo</th>
          <th>Significado</th>
          <th>Versão</th>
          <th>Situação</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="vocabulo in vocabulosComSituacao" :key="vocabulo.id || vocabulo.termo">
          <td>{{ vocabulo.id }}</td>
          <td>{{ vocabulo.termo }}</td>
          <td>{{ vocabulo.significado }}</td>
          <td>{{ vocabulo.versao }}</td>
          <td>{{ vocabulo.situacao }}</td>
        </tr>
      </tbody>
    </table>
    <p v-else>Nenhum vocábulo encontrado.</p>

    <!-- Formulário de cadastro -->
    <h2>Cadastrar Novo Vocábulo</h2>
    <form @submit.prevent="cadastrarVocabulo">
      <div>
        <label for="termo">Termo:</label>
        <input id="termo" v-model="novoVocabulo.termo" type="text" />
        <span style="color: red;">{{ erros.termo }}</span>
      </div>
      <div>
        <label for="significado">Significado:</label>
        <input id="significado" v-model="novoVocabulo.significado" type="text" />
        <span style="color: red;">{{ erros.significado }}</span>
      </div>
      <div>
        <label for="versao">Versão:</label>
        <input id="versao" v-model="novoVocabulo.versao" type="number" />
      </div>
      <div>
        <label for="dataHoraDesativacao">Data de Desativação:</label>
        <input id="dataHoraDesativacao" v-model="novoVocabulo.dataHoraDesativacao" type="datetime-local" />
      </div>
      <button type="submit">Cadastrar</button>
    </form>
  </div>
</template>

<style scoped>
form {
  margin-bottom: 20px;
}

form div {
  margin-bottom: 10px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  padding: 10px;
  text-align: left;
}
</style>
