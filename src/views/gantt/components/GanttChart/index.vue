<!--
 * @FilePath src/views/gantt/components/GanttChart/index.vue
 * @Created Bay丶<baizhanying@autobio.com.cn> 2021-11-11 11:12:27
 * @Modified Bay丶<baizhanying@autobio.com.cn> 2021-11-11 14:12:06
 * @Description
-->

<template>
  <div id="diagram" ref="diagram" />
</template>

<script>
import go from 'gojs'
const $ = go.GraphObject.make
export default {
  name: 'GanttChart',
  data() {
    return {
      diagram: null
    }
  },
  mounted() {
    this.diagram =
        $(go.Diagram, this.$refs['diagram'], // Diagram refers to its DIV HTML element by id
          {
            _widthFactor: 1, // a scale for the nodes' positions and widths
            isReadOnly: true, // deny the user permission to alter the diagram or zoom in or out
            allowZoom: false, // 禁止缩放
            allowSelect: false, // 禁止选中
            allowHorizontalScroll: false, // 禁止画布横向滚动
            allowVerticalScroll: false, // 禁止画布纵向滚动
            'grid.visible': true, // 在背景显示一个网格
            'grid.gridCellSize': new go.Size(30, 150) // 在背景网格 格子大小
          })
    this.createNodeTemplates()
    this.crateLinkTemplate()

    this.diagram.model = new go.GraphLinksModel(
      [ // node data
        { key: 'a', color: 'coral', width: 120, loc: new go.Point(0, 40) },
        { key: 'b', color: 'turquoise', width: 160, loc: new go.Point(0, 60) },
        { key: 'c', color: 'coral', width: 150, loc: new go.Point(120, 80) },
        { key: 'd', color: 'turquoise', width: 190, loc: new go.Point(120, 100) },
        { key: 'e', color: 'coral', width: 150, loc: new go.Point(270, 120) },
        { key: 'f', color: 'turquoise', width: 130, loc: new go.Point(310, 140) },
        { key: 'g', color: 'coral', width: 155, loc: new go.Point(420, 160) },
        { key: 'begin', category: 'start', loc: new go.Point(-15, 20) },
        { key: 'end', category: 'end', loc: new go.Point(575, 180) }
      ],
      [ // link data
        { from: 'begin', to: 'a' },
        { from: 'begin', to: 'b' },
        { from: 'a', to: 'c' },
        { from: 'a', to: 'd' },
        { from: 'b', to: 'e' },
        { from: 'c', to: 'e' },
        { from: 'd', to: 'f' },
        { from: 'e', to: 'g' },
        { from: 'f', to: 'end' },
        { from: 'g', to: 'end' }
      ])

    const dateScale =
        $(go.Part, 'Graduated',
          {
            graduatedTickUnit: 1,
            graduatedMin: 0,
            graduatedMax: 3,
            pickable: false,
            location: new go.Point(0, 0)
          },
          $(go.Shape,
            { name: 'line', strokeWidth: 0, geometryString: 'M0 0 H' + 450 }
          ),
          $(go.TextBlock,
            {
              name: 'labels',
              font: '10pt sans-serif',
              alignmentFocus: new go.Spot(0, 0, -3, -3),
              graduatedFunction: function(v) {
                var d = new Date(2017, 6, 23)
                d.setDate(d.getDate() + v * 7)
                // format date output to string
                var options = { month: 'short', day: '2-digit' }
                return d.toLocaleDateString('en-US', options)
              }
            }
          )
        )
    this.diagram.add(dateScale)
  },
  methods: {
    createNodeTemplates() {
      // create the template for the standard nodes
      this.diagram.nodeTemplateMap.add('',
        $(go.Node, 'Auto',
          // links come from the right and go to the left side of the top of the node
          { fromSpot: go.Spot.Right, toSpot: new go.Spot(0.001, 0, 11, 0) },
          $(go.Shape, 'Rectangle',
            { height: 15 },
            new go.Binding('fill', 'color'),
            new go.Binding('width', 'width')),
          $(go.TextBlock,
            { margin: 2, alignment: go.Spot.Left },
            new go.Binding('text', 'key')),
          // using a function in the Binding allows the value to
          // change when Diagram.updateAllTargetBindings is called
          new go.Binding('location', 'loc',
            function(l) { return new go.Point(l.x, l.y) })
        ))

      // create the template for the start node
      this.diagram.nodeTemplateMap.add('start',
        $(go.Node,
          { fromSpot: go.Spot.Right, toSpot: go.Spot.Top, selectable: false },
          $(go.Shape, 'Diamond',
            { height: 15, width: 15 }),
          // make the location of the start node is not scalable
          new go.Binding('location', 'loc')
        ))

      // create the template for the end node
      this.diagram.nodeTemplateMap.add('end',
        $(go.Node,
          { fromSpot: go.Spot.Right, toSpot: go.Spot.Top, selectable: false },
          $(go.Shape, 'Diamond',
            { height: 15, width: 15 }),
          // make the location of the end node (with location.x < 0) scalable
          new go.Binding('location', 'loc',
            function(l) {
              if (l.x >= 0) return new go.Point(l.x, l.y)
              else return l
            })
        ))
    },
    crateLinkTemplate() {
      this.diagram.linkTemplate =
        $(go.Link,
          {
            routing: go.Link.Orthogonal,
            corner: 3,
            toShortLength: 2,
            selectable: false
          },
          $(go.Shape, { strokeWidth: 2 }),
          $(go.Shape, { toArrow: 'OpenTriangle' })
        )
    }
  }
}
</script>

<style lang="scss" scoped>
#diagram {
  width: 100%;
  height: 100%;
  ::v-deep canvas {
    outline: none;
  }
}
</style>
