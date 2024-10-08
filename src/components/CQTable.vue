<template>
  <div class="c-q-table-wrapper" ref="tableWrapper">
    <q-table
      :rows="rows"
      :columns="visibleColumnsData"
      row-key="id"
      :sort-method="customSort"
      @request="onRequest"
      class="c-q-table"
    >
      <template v-slot:header="props">
        <q-tr :props="props">
          <q-th
            v-for="(col, index) in visibleColumnsData"
            :key="col.name"
            :style="getColumnStyle(col.name)"
            class="relative-position"
          >
            <div
              class="row items-center full-width column-header draggable-area"
              @mousedown="startColumnDrag(col.name, $event)"
            >
              <div class="col">{{ col.label }}</div>
              <q-icon
                v-if="col.sortable"
                :name="props.sort.field === col.name ? (props.sort.descending ? 'arrow_downward' : 'arrow_upward') : 'unfold_more'"
                class="q-ml-sm cursor-pointer"
                @click.stop="props.sort(col.name)"
              />
            </div>
            <div
              v-if="index < visibleColumnsData.length - 1"
              class="column-resizer"
              @mousedown.stop="startColumnResize(col.name, $event)"
            ></div>
          </q-th>
        </q-tr>
      </template>
      <template v-slot:body="props">
        <q-tr :props="props">
          <q-td
            v-for="col in visibleColumnsData"
            :key="col.name"
            :props="props"
            :style="getColumnStyle(col.name)"
          >
            {{ props.row[col.field] }}
          </q-td>
        </q-tr>
      </template>
    </q-table>
    <div v-if="draggedColumn" class="drag-indicator" :style="dragIndicatorStyle"></div>
  </div>
</template>

<script>
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'CQTable',
  props: {
    columns: {
      type: Array,
      required: true
    },
    rows: {
      type: Array,
      required: true
    },
    visibleColumns: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      columnOrder: [],
      columnWidths: {},
      draggedColumn: null,
      resizedColumn: null,
      startX: 0,
      dragX: 0,
      sortBy: null,
      descending: false,
      totalWidth: 0
    }
  },
  computed: {
    visibleColumnsData() {
      return this.columnOrder
        .filter(name => this.visibleColumns.includes(name))
        .map(name => this.columns.find(col => col.name === name))
        .filter(Boolean)
    },
    dragIndicatorStyle() {
      if (!this.draggedColumn) return {}
      const tableRect = this.$refs.tableWrapper.getBoundingClientRect()
      return {
        width: `${this.columnWidths[this.draggedColumn]}px`,
        height: `${tableRect.height}px`,
        top: '0px',
        left: `${this.dragX - tableRect.left}px`,
        opacity: 0.7
      }
    }
  },
  watch: {
    visibleColumns: {
      handler(newVal) {
        this.columnOrder = newVal
        this.$nextTick(() => {
          this.initializeColumnWidths()
        })
      },
      immediate: true
    }
  },
  mounted() {
    this.initializeColumnWidths()
    window.addEventListener('resize', this.handleResize)
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.handleResize)
  },
  methods: {
    handleResize() {
      this.initializeColumnWidths()
    },
    initializeColumnWidths() {
      const tableWidth = this.$refs.tableWrapper.clientWidth
      this.totalWidth = tableWidth
      const defaultWidth = Math.max(100, tableWidth / this.visibleColumns.length)
      this.visibleColumns.forEach(colName => {
        if (!this.columnWidths[colName]) {
          this.columnWidths[colName] = defaultWidth
        }
      })
      this.adjustColumnWidths()
      this.$forceUpdate()
    },
    adjustColumnWidths() {
      const currentTotal = Object.values(this.columnWidths).reduce((sum, width) => sum + width, 0)
      const ratio = this.totalWidth / currentTotal
      Object.keys(this.columnWidths).forEach(colName => {
        this.columnWidths[colName] *= ratio
      })
    },
    getColumnStyle(colName) {
      return {
        width: `${this.columnWidths[colName]}px`,
        transition: this.draggedColumn ? 'width 0.2s ease-out' : 'none',
        'text-align': this.columns.find(col => col.name === colName)?.align || 'left'
      }
    },
    startColumnDrag(columnName, event) {
      if (event.target.classList.contains('column-resizer') || event.target.classList.contains('q-icon')) {
        return
      }
      this.draggedColumn = columnName
      this.dragX = event.clientX
      document.addEventListener('mousemove', this.handleColumnDrag)
      document.addEventListener('mouseup', this.endColumnDrag)
    },
    handleColumnDrag(event) {
      if (!this.draggedColumn) return

      this.dragX = event.clientX
      const draggedIndex = this.columnOrder.indexOf(this.draggedColumn)
      const draggedWidth = this.columnWidths[this.draggedColumn]
      const dragCenter = this.dragX - (draggedWidth / 2)

      let newIndex = draggedIndex
      let accumulatedWidth = 0
      for (let i = 0; i < this.columnOrder.length; i++) {
        const colName = this.columnOrder[i]
        const colWidth = this.columnWidths[colName]
        const colCenter = accumulatedWidth + (colWidth / 2)

        if (dragCenter < colCenter) {
          newIndex = i
          break
        }

        accumulatedWidth += colWidth
      }

      if (newIndex !== draggedIndex) {
        const newOrder = [...this.columnOrder]
        newOrder.splice(draggedIndex, 1)
        newOrder.splice(newIndex, 0, this.draggedColumn)
        this.columnOrder = newOrder
      }
    },
    endColumnDrag() {
      this.$emit('update:visibleColumns', this.columnOrder)
      this.draggedColumn = null
      document.removeEventListener('mousemove', this.handleColumnDrag)
      document.removeEventListener('mouseup', this.endColumnDrag)
    },
    startColumnResize(columnName, event) {
      event.preventDefault()
      this.resizedColumn = columnName
      this.startX = event.clientX
      document.addEventListener('mousemove', this.handleColumnResize)
      document.addEventListener('mouseup', this.endColumnResize)
    },
    handleColumnResize(event) {
      if (!this.resizedColumn) return

      const index = this.columnOrder.indexOf(this.resizedColumn)
      const nextColumnName = this.columnOrder[index + 1]

      const diffX = event.clientX - this.startX
      const newWidth = Math.max(50, this.columnWidths[this.resizedColumn] + diffX)
      const oldWidth = this.columnWidths[this.resizedColumn]

      this.columnWidths[this.resizedColumn] = newWidth
      this.columnWidths[nextColumnName] -= (newWidth - oldWidth)

      this.startX = event.clientX
      this.$forceUpdate()
    },
    endColumnResize() {
      this.resizedColumn = null
      document.removeEventListener('mousemove', this.handleColumnResize)
      document.removeEventListener('mouseup', this.endColumnResize)
    },
    customSort(rows, sortBy, descending) {
      const data = [...rows]
      if (sortBy) {
        data.sort((a, b) => {
          const x = descending ? b[sortBy] : a[sortBy]
          const y = descending ? a[sortBy] : b[sortBy]
          if (x < y) return -1
          if (x > y) return 1
          return 0
        })
      }
      return data
    },
    onRequest(props) {
      this.sortBy = props.pagination.sortBy
      this.descending = props.pagination.descending
    }
  }
})
</script>

<style scoped>
.c-q-table-wrapper {
  position: relative;
  width: 100%;
  overflow-x: auto;
}

.c-q-table {
  width: 100%;
  table-layout: fixed;
}

.q-table thead tr th, .q-table tbody td {
  transition: width 0.2s ease-out;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.draggable-area {
  cursor: move; /* fallback if grab cursor is unsupported */
  cursor: grab;
  cursor: -moz-grab;
  cursor: -webkit-grab;
}

.draggable-area:active {
  cursor: grabbing;
  cursor: -moz-grabbing;
  cursor: -webkit-grabbing;
}

.column-resizer {
  width: 8px;
  height: 100%;
  position: absolute;
  right: -4px;
  top: 0;
  cursor: col-resize;
  user-select: none;
  background-color: #ddd;
  z-index: 2;
}

.column-resizer:hover {
  background-color: #bbb;
}

.column-header {
  padding-right: 8px;
}

.drag-indicator {
  position: absolute;
  background-color: rgba(0, 0, 0, 0.1);
  border: 1px dashed #666;
  pointer-events: none;
  z-index: 9999;
}
</style>
