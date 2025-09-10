<template>
  <main>
    <h1>Catálogo de Empréstimo de Equipamentos</h1>

    <!-- Formulário -->
    <form @submit.prevent="onSubmit">
      <label>
        Nome:
        <input v-model="form.nome" required />
      </label>

      <label>
        Categoria:
        <input v-model="form.categoria" required />
      </label>

      <label>
        Patrimônio:
        <input v-model="form.patrimonio" required />
      </label>

      <label>
        Status:
        <select v-model="form.status" required>
          <option value="disponível">disponível</option>
          <option value="emprestado">emprestado</option>
        </select>
      </label>

      <button type="submit" :disabled="!isFormValid">
        <span v-if="!editing">Adicionar</span>
        <span v-else>Salvar</span>
      </button>
      <button v-if="editing" type="button" @click="cancelEdit">Cancelar</button>
    </form>

    <!-- Filtros -->
    <section>
      <input v-model="filters.categoria" placeholder="Filtrar categoria" />
      <select v-model="filters.status">
        <option value="">Todos</option>
        <option value="disponível">disponível</option>
        <option value="emprestado">emprestado</option>
      </select>
    </section>

    <!-- Contadores -->
    <p>Total: {{ totalEquipamentos }} | Disponíveis: {{ totalDisponiveis }} | Emprestados: {{ totalEmprestados }}</p>

    <!-- Lista -->
    <div v-if="filteredList.length === 0">Nenhum equipamento encontrado.</div>
    <table v-else>
      <thead>
        <tr>
          <th>Nome</th>
          <th>Categoria</th>
          <th>Patrimônio</th>
          <th>Status</th>
          <th>Ações</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in filteredList" :key="item.id">
          <td>{{ item.nome }}</td>
          <td>{{ item.categoria }}</td>
          <td>{{ item.patrimonio }}</td>
          <td :class="statusClass(item.status)">{{ item.status }}</td>
          <td>
            <button @click="startEdit(item)">Editar</button>
            <button @click="removeItem(item)">Remover</button>
          </td>
        </tr>
      </tbody>
    </table>
  </main>
</template>

<script>
const STORAGE_KEY = 'lab-equipments-v1'

export default {
  data() {
    return {
      equipments: [],
      editing: false,
      editingId: null,
      form: { nome: '', categoria: '', patrimonio: '', status: 'disponível' },
      filters: { categoria: '', status: '' }
    }
  },
  computed: {
    totalEquipamentos() {
      return this.equipments.length
    },
    totalDisponiveis() {
      return this.equipments.filter(e => e.status === 'disponível').length
    },
    totalEmprestados() {
      return this.equipments.filter(e => e.status === 'emprestado').length
    },
    filteredList() {
      return this.equipments.filter(item => {
        const byCat = !this.filters.categoria || item.categoria.toLowerCase().includes(this.filters.categoria.toLowerCase())
        const byStatus = !this.filters.status || item.status === this.filters.status
        return byCat && byStatus
      })
    },
    isFormValid() {
      return this.form.nome && this.form.categoria && this.form.patrimonio
    }
  },
  methods: {
    load() {
      const raw = localStorage.getItem(STORAGE_KEY)
      this.equipments = raw ? JSON.parse(raw) : []
    },
    saveStorage() {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(this.equipments))
    },
    generateId() {
      return Date.now() + '-' + Math.floor(Math.random() * 1000)
    },
    onSubmit() {
      if (this.editing) {
        this.saveEdit()
      } else {
        this.addItem()
      }
    },
    addItem() {
      this.equipments.push({ id: this.generateId(), ...this.form })
      this.saveStorage()
      this.resetForm()
    },
    startEdit(item) {
      this.editing = true
      this.editingId = item.id
      this.form = { ...item }
    },
    saveEdit() {
      const idx = this.equipments.findIndex(e => e.id === this.editingId)
      if (idx !== -1) {
        this.equipments[idx] = { id: this.editingId, ...this.form }
        this.saveStorage()
        this.cancelEdit()
      }
    },
    cancelEdit() {
      this.editing = false
      this.editingId = null
      this.resetForm()
    },
    removeItem(item) {
      if (window.confirm(`Remover "${item.nome}"?`)) {
        this.equipments = this.equipments.filter(e => e.id !== item.id)
        this.saveStorage()
      }
    },
    resetForm() {
      this.form = { nome: '', categoria: '', patrimonio: '', status: 'disponível' }
    },
    statusClass(status) {
      return status === 'disponível' ? 'verde' : 'vermelho'
    }
  },
  mounted() {
    this.load()
  }
}
</script>

<style>
/* Estilos globais */
body {
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  background: #f7f9fc;
  margin: 0;
  padding: 0;
  color: #333;
}

/* Container principal */
#app {
  max-width: 900px;
  margin: 40px auto;
  padding: 20px;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

/* Cabeçalho */
h1 {
  text-align: center;
  margin-bottom: 25px;
  font-size: 2rem;
  color: #2c3e50;
}

/* Formulário */
form {
  display: grid;
  gap: 15px;
  margin-bottom: 30px;
}

form input,
form select {
  padding: 10px 14px;
  border: 1px solid #dcdfe6;
  border-radius: 8px;
  font-size: 1rem;
  transition: border 0.2s ease;
}

form input:focus,
form select:focus {
  outline: none;
  border-color: #42b983;
  box-shadow: 0 0 5px rgba(66, 185, 131, 0.4);
}

button {
  cursor: pointer;
  padding: 10px 18px;
  font-size: 0.95rem;
  font-weight: 600;
  border-radius: 8px;
  border: none;
  transition: all 0.3s ease;
}

button[type="submit"] {
  background: #42b983;
  color: white;
}

button[type="submit"]:hover {
  background: #36996e;
}

button.delete {
  background: #e74c3c;
  color: white;
}

button.delete:hover {
  background: #c0392b;
}

button.edit {
  background: #3498db;
  color: white;
}

button.edit:hover {
  background: #2c80b4;
}

/* Lista */
ul {
  list-style: none;
  padding: 0;
}

li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #f2f6fc;
  margin-bottom: 12px;
  padding: 14px 18px;
  border-radius: 8px;
  transition: background 0.2s ease;
}

li:hover {
  background: #e9f5f1;
}

.item-info {
  flex: 1;
}

.actions {
  display: flex;
  gap: 10px;
}

</style>
