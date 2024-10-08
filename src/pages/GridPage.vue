<template>
  <q-page padding>
    <q-card class="no-shadow card_style" bordered>
      <q-card-section class="row">
        <div class="col-12">
          <q-select
            v-model="visibleColumns"
            multiple
            outlined
            :options="columnOptions"
            option-value="name"
            option-label="label"
            emit-value
            map-options
            dense
            options-dense class="float-right"
            style="min-width: 250px"
            label="Visible columns"
          >
            <template v-slot:option="{ itemProps, itemEvents, opt, selected, toggleOption }">
              <q-item v-bind="itemProps" v-on="itemEvents">
                <q-item-section side>
                  <q-checkbox :model-value="selected" @update:model-value="toggleOption(opt)"></q-checkbox>
                </q-item-section>
                <q-item-section>
                  {{ opt.label }}
                </q-item-section>
              </q-item>
            </template>
          </q-select>
        </div>
      </q-card-section>
      <q-separator/>
      <q-card-section class="q-pa-none overflow-hidden">
        <CQTable
          :rows="rows"
          :columns="columns"
          :visibleColumns="visibleColumns"
          @update:visibleColumns="updateVisibleColumns"
        />
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script>
import {defineComponent} from 'vue'
import CQTable from '../components/CQTable.vue'

export default defineComponent({
  name: 'GridPage',
  components: {CQTable},
  data() {
    return {
      rows: [
        {id: 1, name: 'Jane Smith', age: 34, email: 'jane.smith@example.com'},
        {id: 2, name: 'John Doe', age: 28, email: 'john.doe@example.com'},
        {id: 3, name: 'Alice Johnson', age: 45, email: 'alice.johnson@example.com'},
        {id: 4, name: 'Bob Williams', age: 39, email: 'bob.williams@example.com'},
      ],
      columns: [
        {name: 'name', label: 'Name', field: 'name', sortable: true, align: 'left'},
        {name: 'age', label: 'Age', field: 'age', sortable: true, align: 'center'},
        {name: 'email', label: 'Email', field: 'email', sortable: true, align: 'left'}
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

.card_style {
  border-radius: 12px;
}
</style>
