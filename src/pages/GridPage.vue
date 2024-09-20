<template>
  <div>
    <q-layout view="lHh Lpr lFf">
      <q-header elevated class="bg-primary text-white">
        <q-toolbar>
          <q-btn flat dense round icon="menu" aria-label="Menu" />
          <q-toolbar-title>Quasar App</q-toolbar-title>
          <q-space />
          <q-select
            v-model="visibleColumns"
            multiple
            :options="columnOptions"
            option-value="name"
            option-label="label"
            emit-value
            map-options
            dense
            options-dense
            style="min-width: 150px"
            label="Columns"
          >
            <template v-slot:option="{ itemProps, itemEvents, opt, selected, toggleOption }">
              <q-item v-bind="itemProps" v-on="itemEvents">
                <q-item-section>
                  <q-checkbox :model-value="selected" @update:model-value="toggleOption(opt)">
                    <q-item-label>{{ opt.label }}</q-item-label>
                  </q-checkbox>
                </q-item-section>
              </q-item>
            </template>
          </q-select>
        </q-toolbar>
      </q-header>

      <q-page-container>
        <q-page padding>
          <CQTable 
            :rows="rows" 
            :columns="columns" 
            :visibleColumns="visibleColumns"
            @update:visibleColumns="updateVisibleColumns"
          />
        </q-page>
      </q-page-container>
    </q-layout>
  </div>
</template>

<script>
import { defineComponent } from 'vue'
import CQTable from '../components/CQTable.vue'

export default defineComponent({
  name: 'GridPage',
  components: { CQTable },
  data() {
    return {
      rows: [
        { id: 1, name: 'Jane Smith', age: 34, email: 'jane.smith@example.com' },
        { id: 2, name: 'John Doe', age: 28, email: 'john.doe@example.com' },
        { id: 3, name: 'Alice Johnson', age: 45, email: 'alice.johnson@example.com' },
        { id: 4, name: 'Bob Williams', age: 39, email: 'bob.williams@example.com' },
      ],
      columns: [
        { name: 'name', label: 'Name', field: 'name', sortable: true },
        { name: 'age', label: 'Age', field: 'age', sortable: true },
        { name: 'email', label: 'Email', field: 'email', sortable: true }
      ],
      visibleColumns: ['name', 'age', 'email']
    }
  },
  computed: {
    columnOptions() {
      return this.columns.map(col => ({
        label: col.label,
        name: col.name,
        value: col.name
      }))
    }
  },
  methods: {
    updateVisibleColumns(newVisibleColumns) {
      this.visibleColumns = newVisibleColumns
    }
  }
})
</script>

<style scoped>
.q-page-container {
  height: calc(100vh - 50px);
}

.q-page {
  padding: 16px;
}
</style>
