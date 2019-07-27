<template>
  <div>
    <table
      id="table"
      @mousedown.left="downAction"
      @mousedown.right="downActionOnece"
      @mouseup.left="upAction"
      @mouseleave="upAction"
      @contextmenu="menuShow"
    >
      <tr v-for="(r,index) in tableData" :key="index">
        <td
          v-for="(item,tdIndex) in r"
          :key="tdIndex"
          :data-x="item.x"
          :data-y="item.y"
          :data-value="item.value"
          :class="[item.select ? 'select' : '']"
          @mousemove="moveAction"
        >
          <input
            type="text"
            :data-x="item.x"
            :data-y="item.y"
            :value="item.value"
            @input="inputAction"
          >
        </td>
      </tr>
    </table>
    <Menu
      id="menu"
      :top="menuTop"
      :left="menuLeft"
      @deleteRow="deleteRow"
      @InsertColumn="InsertColumn"
      @deleteColumn="deleteColumn"
      @InsertRow="InsertRow"
      @mergeTd="mergeTd"
    />
  </div>
</template>

<script>
import throttle from 'lodash.throttle'
import Menu from './Menu'

export default {
  name: 'Table',
  components: {
    Menu
  },
  data() {
    return {
      tableData: [], // colspan rowspan value x y // onmousedown onmousemove onmouseup
      row: '10',
      column: '10',
      record: false,
      /**
       * 选中的坐标
       */
      position: {
        x: {
          from: -1,
          to: -1
        },
        y: {
          from: -1,
          to: -1
        }
      },
      menuTop: -800,
      menuLeft: -800
    }
  },
  watch: {
    position: {
      handler(newName, oldName) {
        this.tableData = this.makeTableData()
      },
      immediate: true,
      deep: true
    }
  },
  created() {
    this.tableData = this.initTable()
  },
  mounted() {
    document.addEventListener('click', this.handlerClose)
  },
  destroyed() {
    document.removeEventListener('click', this.handlerClose)
  },
  methods: {
    deleteRow() {
      const { from, to } = this.position.x
      this.tableData.splice(Math.min(from, to), (Math.abs(to - from) + 1))
      this.row = this.row - (Math.abs(to - from) + 1)
    },
    deleteColumn() {
      const { from, to } = this.position.y
      this.column = this.column - (Math.abs(to - from) + 1)
      this.tableData.forEach((arr, index) => {
        arr.splice(Math.min(from, to), (Math.abs(to - from) + 1))
      })
    },
    InsertRow() {
      const { from, to } = this.position.x
      const max = Math.max(from, to)
      const tableData = []
      const column = parseInt(this.column)
      for (let r = 0; r <= max + 1; r++) {
        const arr = []
        for (let col = 0; col < parseInt(column); col++) {
          const select = this.inRange(r, col)
          let value = ''
          if (r === max + 1) {
            value = ''
          } else {
            value = this.tableData[r][col].value
          }
          const obj = {
            value,
            x: r,
            y: col,
            select
          }
          arr.push(obj)
        }
        tableData[r] = arr
      }
      const backArr = this.tableData.slice(max + 1)
      this.tableData = tableData.concat(backArr)
      this.row = parseInt(this.row) + 1
    },
    InsertColumn(event) {
      const { row, column } = this
      const { from, to } = this.position.y
      const max = Math.max(from, to)
      const tableData = []
      for (let r = 0; r < row; r++) {
        const arr = []
        for (let col = 0; col < parseInt(column) + 1; col++) {
          const select = this.inRange(r, col)
          let value = ''
          if (col === max + 1) { // 最新插入的列
            value = ''
          } else {
            if (col <= max) { // 插入列所在的前面的列
              value = this.tableData[r][col].value
            } else {
              value = this.tableData[r][col - 1].value
            }
          }
          const obj = {
            value,
            x: r,
            y: col,
            select
          }
          arr.push(obj)
        }
        tableData[r] = arr
      }
      this.tableData = tableData
      this.column = parseInt(column) + 1
    },
    mergeTd() {

    },
    menuShow(event) {
      this.menuTop = event.clientY
      this.menuLeft = event.clientX
      event.preventDefault()
      return false
    },
    /**
     * 取消选中区域的选中
     */
    handlerClose(event) {
      if (!document.getElementById('menu').contains(event.target)) {
        this.menuTop = -800
        this.menuLeft = -800
      }
      if (document.getElementById('table').contains(event.target) || document.getElementById('menu').contains(event.target)) {
        return ''
      }
      this.position.x.from = -1
      this.position.y.from = -1
      this.position.x.to = -1
      this.position.y.to = -1
    },
    moveAction: throttle(function(event) {
      if (this.record) {
        const { x, y } = event.target.dataset
        this.position.x.to = parseInt(x)
        this.position.y.to = parseInt(y)
      }
    }, 100),

    /**
     * 点击右键之后直接修改选中区域
     */
    downActionOnece(event) {
      // 如果选择的区域不是一块
      if ((this.position.x.from - this.position.x.to !== 0) ||
          (this.position.y.from - this.position.y.to !== 0)
      ) {
        return
      }
      this.position.x.from = parseInt(event.target.dataset.x)
      this.position.y.from = parseInt(event.target.dataset.y)
      this.position.x.to = parseInt(event.target.dataset.x)
      this.position.y.to = parseInt(event.target.dataset.y)
    },
    /**
     * 初始化选中数据
     */
    downAction(event) {
      this.record = true
      this.position.x.from = parseInt(event.target.dataset.x)
      this.position.y.from = parseInt(event.target.dataset.y)
      this.position.x.to = parseInt(event.target.dataset.x)
      this.position.y.to = parseInt(event.target.dataset.y)
    },
    upAction(event) {
      this.record = false
    },
    /**
     * 输入完成
     */
    inputAction(event) {
      const { dataset: { x, y }, value } = event.target
      this.tableData[x][y].value = value
    },
    /**
     * 初始化表格
     */
    initTable() {
      const { row, column } = this
      const tableData = []
      for (let r = 0; r < parseInt(row); r++) {
        const arr = []
        for (let col = 0; col < parseInt(column); col++) {
          const obj = {
            value: `(${r}:${col})`,
            x: r,
            y: col,
            select: false
          }
          arr.push(obj)
        }
        tableData.push(arr)
      }
      return tableData
    },
    /**
     * 选中区域的数组重新生成
     */
    makeTableData() {
      const tableData = [...this.tableData]
      const row = tableData.length
      if (row === 0) {
        return []
      }
      const column = tableData[0].length
      for (let r = 0; r < parseInt(row); r++) {
        const arr = []
        for (let col = 0; col < parseInt(column); col++) {
          const select = this.inRange(r, col)
          const obj = {
            value: tableData[r][col].value,
            x: r,
            y: col,
            select
          }
          arr.push(obj)
        }
        tableData[r] = arr
      }
      return tableData
    },
    inRange(x, y) {
      const { x: { from: xFrom, to: xTo }, y: { from: yFrom, to: yTo }} = this.position
      /**
       * 左上到右下
       */
      if ((x >= xFrom && x <= xTo) && (y >= yFrom && y <= yTo)) {
        return true
      }
      /**
       * 右下到左上
       */
      if ((x <= xFrom && x >= xTo) && (y <= yFrom && y >= yTo)) {
        return true
      }
      /**
       * 左下到右上
       */
      if ((x <= xFrom && x >= xTo) && (y >= yFrom && y <= yTo)) {
        return true
      }
      /**
       * 右上 到 左下
       */
      if ((x >= xFrom && x <= xTo) && (y <= yFrom && y >= yTo)) {
        return true
      }
      return false
    }
  }

}
</script>

<style scoped>
table {
  border-collapse: collapse;
  margin: 0 auto;
  text-align: center;
}
table:active,table:hover{
  border: 1px solid #cad9ea;
}
table td,
table th {
  border: 1px solid #cad9ea;
  color: #666;
  height: 30px;
  user-select: none;
}
table td {
  padding: 0 5px;
  width: 100px;
  box-sizing: border-box;
}
table td input{
  outline: none;
  border: 0;
  width: 100%;
  background: transparent;
  box-sizing: border-box;
  text-align: center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  height: 100%;
}
table td:hover{
  box-shadow: 0px 0px 10px 1px rgba(164, 232, 227, 0.2);
}
table tr:nth-child(odd) {
  background: #fff;
}
table tr:nth-child(even) {
  background: #f5fafa;
}
.select{
  background:#FFFFCC;
}
</style>
