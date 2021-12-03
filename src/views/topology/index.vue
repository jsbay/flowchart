<!--
 * @FilePath src/views/topology/index.vue
 * @Created Bay丶<baizhanying@autobio.com.cn> 2021-11-19 15:07:30
 * @Modified Bay丶<baizhanying@autobio.com.cn> 2021-12-03 14:07:47
 * @Description https://codepen.io/Chieffo/pen/oNjPdwz?editors=1010
-->

<template>
  <div id="diagram-contanier">
    <el-button type="primary" class="absolute" @click="handleClick">点击</el-button>
    <el-button type="success" plain class="absolute" style="right: 86px" @click="save">保存</el-button>
    <div id="diagram" ref="diagram" />
    <ul id="diagramContextMenu" ref="diagramContextMenu" class="menu">
      <li id="cut" class="menu-item" @click="cxcommand">Cut</li>
      <li id="copy" class="menu-item" @click="cxcommand">Copy</li>
      <li id="paste" class="menu-item" @click="cxcommand">Paste</li>
      <li id="delete" class="menu-item" @click="cxcommand">Delete</li>
      <li id="color" class="menu-item">Color
        <ul class="menu">
          <li class="menu-item" style="background-color: #f38181;" @click="cxcommand($event, 'color')">Red</li>
          <li class="menu-item" style="background-color: #eaffd0;" @click="cxcommand($event, 'color')">Green</li>
          <li class="menu-item" style="background-color: #95e1d3;" @click="cxcommand($event, 'color')">Blue</li>
          <li class="menu-item" style="background-color: #fce38a;" @click="cxcommand($event, 'color')">Yellow</li>
        </ul>
      </li>
    </ul>
  </div>
</template>

<script>
import go from 'gojs'
const $ = go.GraphObject.make
export default {
  name: 'Index',
  data() {
    return {
      diagram: null,
      cxmEl: null
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.cxmEl = this.$refs['diagramContextMenu']

      const hideCX = () => {
        if (this.diagram.currentTool instanceof go.ContextMenuTool) {
          this.diagram.currentTool.doCancel()
        }
      }

      const showContextMenu = (obj, diagram, tool) => {
        const cmd = diagram.commandHandler
        let hasMenuItem = false
        const maybeShowItem = (elt, pred) => {
          if (pred) {
            elt.style.display = 'block'
            hasMenuItem = true
          } else {
            elt.style.display = 'none'
          }
        }

        maybeShowItem(document.getElementById('cut'), cmd.canCutSelection())
        maybeShowItem(document.getElementById('copy'), cmd.canCopySelection())
        maybeShowItem(document.getElementById('paste'), cmd.canPasteSelection(diagram.toolManager.contextMenuTool.mouseDownPoint))
        maybeShowItem(document.getElementById('delete'), cmd.canDeleteSelection())
        maybeShowItem(document.getElementById('color'), obj !== null)

        if (hasMenuItem) {
          this.cxmEl.classList.add('show-menu')
          // we don't bother overriding positionContextMenu, we just do it here:
          var mousePt = diagram.lastInput.viewPoint
          this.cxmEl.style.left = mousePt.x + 5 + 'px'
          this.cxmEl.style.top = mousePt.y + 'px'
        }

        // Optional: Use a `window` click listener with event capture to
        //           remove the context menu if the user clicks elsewhere on the page
        window.addEventListener('click', hideCX, true)
      }

      const hideContextMenu = () => {
        this.cxmEl.classList.remove('show-menu')
        window.removeEventListener('click', hideCX, true)
      }

      const myContextMenu = $(go.HTMLInfo, {
        show: showContextMenu,
        hide: hideContextMenu
      })

      this.diagram = $(go.Diagram, this.$refs['diagram'], {
        // layout: $(go.TreeLayout), // 主体布局为树形结构
        maxSelectionCount: 1, // 选中的节点最多为 1 个
        'toolManager.hoverDelay': 10, // 鼠标悬浮提示框显示延迟时间
        allowHorizontalScroll: false, // 禁止水平滚动
        allowVerticalScroll: false, // 禁止垂直滚动
        allowMove: false, // 不允许拖拽元素
        // allowClipboard: false, // 不允许复制粘贴
        // allowDelete: false, // 不允许删除
        padding: 70,
        'undoManager.isEnabled': true,
        contextMenu: myContextMenu
      })
      this.diagram.nodeTemplate = $(go.Node, 'Auto', { contextMenu: myContextMenu },
        $(go.Shape, 'Rectangle', { stroke: null }, new go.Binding('fill', 'color')),
        $(go.TextBlock, { margin: 6, font: '18px sans-serif', editable: true }, new go.Binding('text').makeTwoWay()))
      this.diagram.model = new go.GraphLinksModel([
        { key: 1, text: 'Alpha', color: 'lightblue' },
        { key: 2, text: 'Beta', color: 'orange' },
        { key: 3, text: 'Gamma', color: 'lightgreen' },
        { key: 4, text: 'Delta', color: 'pink' }
      ])
    })
  },
  methods: {
    handleClick(e, obj) {
      console.log(e)
      console.log(obj)
      const DEFAULT_NODE_DARA = { text: 'Node', color: '#badc58' }
      this.diagram.model.addNodeData(DEFAULT_NODE_DARA)
      var node = e.diagram.findPartForData(DEFAULT_NODE_DARA)
      console.log(node)
      node.location = this.diagram.lastInput.documentPoint

      // const node = this.diagram.findNodeForKey(1)
      // this.diagram.model.commit((d) => {
      //   d.set(node.data, 'color', 'red')
      // })
    },
    save() {
      console.log(this.diagram.model.toJson())
    },
    cxcommand(event, val) {
      if (val === undefined) val = event.currentTarget.id
      switch (val) {
        case 'cut': this.diagram.commandHandler.cutSelection(); break
        case 'copy': this.diagram.commandHandler.copySelection(); break
        case 'paste': this.diagram.commandHandler.pasteSelection(this.diagram.toolManager.contextMenuTool.mouseDownPoint); break
        case 'delete': this.diagram.commandHandler.deleteSelection(); break
        case 'color': {
          var color = window.getComputedStyle(event.target)['background-color']
          this.changeColor(this.diagram, color); break
        }
      }
      this.diagram.currentTool.stopTool()
    },
    changeColor(diagram, color) {
    // Always make changes in a transaction, except when initializing the diagram.
      diagram.startTransaction('change color')
      diagram.selection.each(function(node) {
        if (node instanceof go.Node) { // ignore any selected Links and simple Parts
        // Examine and modify the data, not the Node directly.
          var data = node.data
          // Call setDataProperty to support undo/redo as well as
          // automatically evaluating any relevant bindings.
          diagram.model.setDataProperty(data, 'color', color)
        }
      })
      diagram.commitTransaction('change color')
    }
  }
}
</script>

<style lang="scss" scoped>
#diagram-contanier {
  height: calc(100vh - 84px);
  position: relative;
  .absolute {
    position: absolute;
    top: 20px;
    right: 20px;
    z-index: 3;
  }
  #diagram {
    // background-color: #050f26;
    height: 100%;
  }
  ::v-deep canvas {
    outline: none;
  }

  .menu {
    display: none;
    position: absolute;
    opacity: 0;
    margin: 0;
    padding: 8px 0;
    z-index: 999;
    box-shadow: 0 5px 5px -3px rgba(0, 0, 0, 0.2),
      0 8px 10px 1px rgba(0, 0, 0, 0.14), 0 3px 14px 2px rgba(0, 0, 0, 0.12);
    list-style: none;
    background-color: #ffffff;
    border-radius: 4px;
  }

  .menu-item {
    display: block;
    position: relative;
    min-width: 60px;
    margin: 0;
    padding: 6px 16px;
    font: bold 12px sans-serif;
    color: rgba(0, 0, 0, 0.87);
    cursor: pointer;
  }

  .menu-item::before {
    position: absolute;
    top: 0;
    left: 0;
    opacity: 0;
    pointer-events: none;
    content: "";
    width: 100%;
    height: 100%;
    background-color: #000000;
  }

  .menu-item:hover::before {
    opacity: 0.04;
  }

  .menu .menu {
    top: -8px;
    left: 100%;
  }

  .show-menu,
  .menu-item:hover > .menu {
    display: block;
    opacity: 1;
  }
}
</style>
