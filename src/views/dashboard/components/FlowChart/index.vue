<!--
 * @FilePath src/views/dashboard/components/FlowChart/index.vue
 * @Created Bay丶<baizhanying@autobio.com.cn> 2021-11-10 14:23:50
 * @Modified Bay丶<baizhanying@autobio.com.cn> 2021-11-11 13:58:47
 * @Description https://github.com/NorthwoodsSoftware/GoJS/blob/master/samples/flowchart.html
-->

<template>
  <div id="diagram-contanier">
    <div id="diagram-tool">
      <slot>
        <el-button type="primary" @click="save">保存</el-button>
        <el-button @click="load">加载</el-button>
      </slot>
    </div>
    <div id="diagram-wrap">

      <div id="diagram-palette" ref="palette" />
      <div id="diagram" ref="diagram" />
    </div>
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
      palette: null,

      textStyle: {
        font: 'bold 9pt Lato, Helvetica, Arial, sans-serif',
        stroke: '#F8F8F8'
      },
      chart: { 'class': 'GraphLinksModel',
        'linkFromPortIdProperty': 'fromPort',
        'linkToPortIdProperty': 'toPort',
        'nodeDataArray': [
          { 'category': 'Start', 'text': '开始', 'key': -1, 'loc': '-107.94070280440474 -309.08209495195183' },
          { 'text': '流程', 'key': -2, 'loc': '-107.68839591351332 -189.421875' },
          { 'category': 'Conditional', 'text': '逻辑判断', 'key': -3, 'loc': '-107.84415803147851 -72.68596298078074' },
          { 'category': 'End', 'text': '结束', 'key': -4, 'loc': '-109.21840230579596 76.632035837822' }
        ],
        'linkDataArray': [
          { 'from': -2, 'to': -3, 'fromPort': 'B', 'toPort': 'T', 'points': [-107.68839591351332, -169.421875, -107.68839591351332, -159.421875, -107.68839591351332, -131.05391899039037, -107.84415803147851, -131.05391899039037, -107.84415803147851, -102.68596298078074, -107.84415803147851, -92.68596298078074] },
          { 'from': -1, 'to': -2, 'fromPort': 'B', 'toPort': 'T', 'points': [-107.94070280440474, -289.08209495195183, -107.94070280440474, -279.08209495195183, -107.94070280440474, -249.25198497597592, -107.68839591351332, -249.25198497597592, -107.68839591351332, -219.421875, -107.68839591351332, -209.421875] },
          { 'from': -3, 'to': -4, 'fromPort': 'B', 'toPort': 'T', 'visible': true, 'points': [-107.84415803147851, -52.685962980780744, -107.84415803147851, -42.685962980780744, -107.84415803147851, 1.9730364285206292, -109.21840230579596, 1.9730364285206292, -109.21840230579596, 46.632035837822, -109.21840230579596, 56.632035837822], 'text': '是' },
          { 'from': -3, 'to': -2, 'fromPort': 'R', 'toPort': 'T', 'visible': true, 'points': [-47.844158031478514, -72.68596298078074, -37.844158031478514, -72.68596298078074, -37.844158031478514, -240.921875, -107.68839591351332, -240.921875, -107.68839591351332, -219.421875, -107.68839591351332, -209.421875], 'text': '否' }
        ] }

    }
  },
  mounted() {
    const showLinkLabel = (e) => {
      var label = e.subject.findObject('LABEL')
      if (label !== null) label.visible = (e.subject.fromNode.data.category === 'Conditional')
    }

    this.diagram = $(go.Diagram, this.$refs['diagram'], {
      LinkDrawn: showLinkLabel, // this DiagramEvent listener is defined below
      LinkRelinked: showLinkLabel,
      allowHorizontalScroll: false,
      allowVerticalScroll: false,
      'undoManager.isEnabled': true // enable undo & redo
    })
    this.setLinkTemplate()
    this.createPattle()

    // 设置 linkFromPortIdProperty 属性, 避免 link 自己重绘
    this.diagram.model.linkFromPortIdProperty = 'fromPort' // 必须记住portIds
    this.diagram.model.linkToPortIdProperty = 'toPort'

    this.diagram.toolManager.linkingTool.temporaryLink.routing = go.Link.Orthogonal
    this.diagram.toolManager.relinkingTool.temporaryLink.routing = go.Link.Orthogonal
  },
  methods: {
    save() {
      console.log(this.diagram.model.toJson())
    },
    load() {
      this.diagram.model = go.Model.fromJson(this.chart)
    },
    createPattle() {
      this.diagram.nodeTemplateMap.add('', // the default category
        $(go.Node, 'Table', this.nodeStyle(),
        // the main object is a Panel that surrounds a TextBlock with a rectangular Shape
          $(go.Panel, 'Auto',
            $(go.Shape, 'RoundedRectangle',
              { desiredSize: new go.Size(120, 40), fill: '#006266', strokeWidth: 0 },
              new go.Binding('figure', 'figure'), new go.Binding('fill', 'color')),
            $(go.TextBlock, this.textStyle,
              {
                margin: 8,
                maxSize: new go.Size(160, NaN),
                wrap: go.TextBlock.WrapFit,
                editable: true
              },
              new go.Binding('text').makeTwoWay()),
            {
              contextMenu: // define a context menu for each node
                $('ContextMenu', // that has one button
                  $('ContextMenuButton',
                    {
                      'ButtonBorder.fill': 'white',
                      '_buttonFillOver': 'skyblue'
                    },
                    $(go.TextBlock, 'change color'),
                    { click: this.changeColor })
                  // more ContextMenuButtons would go here
                ) // end Adornment
            }
          ),
          // four named ports, one on each side:
          this.makePort('T', go.Spot.Top, go.Spot.Top, false, true),
          this.makePort('L', go.Spot.Left, go.Spot.Left, true, true),
          this.makePort('R', go.Spot.Right, go.Spot.Right, true, true),
          this.makePort('B', go.Spot.Bottom, go.Spot.Bottom, true, false)
        ))
      this.diagram.nodeTemplateMap.add('Conditional',
        $(go.Node, 'Table', this.nodeStyle(),
        // the main object is a Panel that surrounds a TextBlock with a rectangular Shape
          $(go.Panel, 'Auto',
            $(go.Shape, 'Diamond',
              { desiredSize: new go.Size(120, 40), fill: '#F79F1F', strokeWidth: 0 },
              new go.Binding('figure', 'figure')),
            $(go.TextBlock, this.textStyle,
              {
                margin: 8,
                maxSize: new go.Size(160, NaN),
                wrap: go.TextBlock.WrapFit,
                editable: true
              },
              new go.Binding('text').makeTwoWay())
          ),
          // four named ports, one on each side:
          this.makePort('T', go.Spot.Top, go.Spot.Top, false, true),
          this.makePort('L', go.Spot.Left, go.Spot.Left, true, true),
          this.makePort('R', go.Spot.Right, go.Spot.Right, true, true),
          this.makePort('B', go.Spot.Bottom, go.Spot.Bottom, true, false)
        ))
      this.diagram.nodeTemplateMap.add('Start',
        $(go.Node, 'Table', this.nodeStyle(),
          $(go.Panel, 'Spot',
            $(go.Shape, 'Ellipse',
              { desiredSize: new go.Size(120, 40), fill: '#009432', strokeWidth: 0 }),
            $(go.TextBlock, 'Start', this.textStyle,
              new go.Binding('text'))
          ),
          // 创建端口, 以供连线(开始节点仅创建底部一个端口)
          // this.makePort('L', go.Spot.Left, go.Spot.Left, true, false),
          // this.makePort('R', go.Spot.Right, go.Spot.Right, true, false),
          this.makePort('B', go.Spot.Bottom, go.Spot.Bottom, true, false)
        ))
      this.diagram.nodeTemplateMap.add('End',
        $(go.Node, 'Table', this.nodeStyle(),
          $(go.Panel, 'Spot',
            $(go.Shape, 'Ellipse',
              { desiredSize: new go.Size(120, 40), fill: '#EA2027', strokeWidth: 0 }),
            $(go.TextBlock, 'End', this.textStyle,
              new go.Binding('text'))
          ),
          // 创建端口, 以供连线(结束节点仅创建顶部一个端口)
          this.makePort('T', go.Spot.Top, go.Spot.Top, false, true)
          // this.makePort('L', go.Spot.Left, go.Spot.Left, false, true),
          // this.makePort('R', go.Spot.Right, go.Spot.Right, false, true)
        ))

      this.palette = $(go.Palette, this.$refs['palette'], // must name or refer to the DIV HTML element
        {
          // Instead of the default animation, use a custom fade-down
          allowZoom: false,
          padding: 20,
          'animationManager.initialAnimationStyle': go.AnimationManager.None,
          'InitialAnimationStarting': this.animateFadeDown, // Instead, animate with this function
          nodeTemplateMap: this.diagram.nodeTemplateMap, // share the templates used by myDiagram
          model: new go.GraphLinksModel([ // specify the contents of the Palette
            { category: 'Start', text: '开始' },
            { text: '流程' },
            { category: 'Conditional', text: '逻辑判断' },
            { category: 'End', text: '结束' }
          ])
        })
    },
    changeColor(e, obj) {
      this.diagram.commit((d) => {
        // get the context menu that holds the button that was clicked
        var contextmenu = obj.part
        // get the node data to which the Node is data bound
        var nodedata = contextmenu.data
        // compute the next color for the node
        var newcolor = 'lightblue'
        switch (nodedata.color) {
          case 'lightblue': newcolor = 'lightgreen'; break
          case 'lightgreen': newcolor = 'lightyellow'; break
          case 'lightyellow': newcolor = 'orange'; break
          case 'orange': newcolor = 'lightblue'; break
        }
        // modify the node data
        // this evaluates data Bindings and records changes in the UndoManager
        d.model.set(nodedata, 'color', newcolor)
      }, 'changed color')
    },
    // 设置 全局 link 样式
    setLinkTemplate() {
      this.diagram.linkTemplate = $(go.Link, // the whole link panel
        {
          routing: go.Link.AvoidsNodes, // 避免 link 穿过节点
          curve: go.Link.JumpOver, //  交叉线 设置
          corner: 5, // 连接线转角弧度
          toShortLength: 4,
          relinkableFrom: true,
          relinkableTo: true,
          reshapable: true,
          // resegmentable: true,
          // mouse-overs subtly highlight links:
          mouseEnter: function(e, link) { link.findObject('HIGHLIGHT').stroke = 'rgba(30,144,255,0.2)' },
          mouseLeave: function(e, link) { link.findObject('HIGHLIGHT').stroke = 'transparent' },
          selectionAdorned: false
        },
        new go.Binding('points').makeTwoWay(),
        $(go.Shape, // the highlight shape, normally transparent
          { isPanelMain: true, strokeWidth: 8, stroke: 'transparent', name: 'HIGHLIGHT' }),
        $(go.Shape, // the link path shape
          { isPanelMain: true, stroke: 'gray', strokeWidth: 2 },
          new go.Binding('stroke', 'isSelected', function(sel) { return sel ? 'dodgerblue' : 'gray' }).ofObject()),
        $(go.Shape, // the arrowhead
          { toArrow: 'standard', strokeWidth: 0, fill: 'gray' }),
        $(go.Panel, 'Auto', // the link label, normally not visible
          { visible: false, name: 'LABEL', segmentIndex: 2, segmentFraction: 0.5 },
          new go.Binding('visible', 'visible').makeTwoWay(),
          $(go.Shape, 'RoundedRectangle', // the label shape
            { fill: '#F8F8F8', strokeWidth: 0 }),
          $(go.TextBlock, '是', // the label
            {
              textAlign: 'center',
              font: '8pt helvetica, arial, sans-serif',
              stroke: '#333333',
              editable: true
            },
            new go.Binding('text').makeTwoWay())
        )
      )
    },
    nodeStyle() {
      return [
        new go.Binding('location', 'loc', go.Point.parse).makeTwoWay(go.Point.stringify),
        { locationSpot: go.Spot.Center }
      ]
    },
    // 创建端口, 以供连线
    makePort(name, align, spot, output, input) {
      var horizontal = align.equals(go.Spot.Top) || align.equals(go.Spot.Bottom)
      // the port is basically just a transparent rectangle that stretches along the side of the node,
      // and becomes colored when the mouse passes over it
      return $(go.Shape,
        {
          fill: 'transparent', // changed to a color in the mouseEnter event handler
          strokeWidth: 0, // no stroke
          width: horizontal ? NaN : 8, // if not stretching horizontally, just 8 wide
          height: !horizontal ? NaN : 8, // if not stretching vertically, just 8 tall
          alignment: align, // align the port on the main Shape
          stretch: (horizontal ? go.GraphObject.Horizontal : go.GraphObject.Vertical),
          portId: name, // declare this object to be a "port"
          fromSpot: spot, // declare where links may connect at this port
          fromLinkable: output, // declare whether the user may draw links from here
          toSpot: spot, // declare where links may connect at this port
          toLinkable: input, // declare whether the user may draw links to here
          cursor: 'pointer', // show a different cursor to indicate potential link point
          mouseEnter: function(e, port) { // the PORT argument will be this Shape
            if (!e.diagram.isReadOnly) port.fill = 'rgba(255,0,255,0.5)'
          },
          mouseLeave: function(e, port) {
            port.fill = 'transparent'
          }
        })
    },
    animateFadeDown(e) {
      var diagram = e.diagram
      var animation = new go.Animation()
      animation.isViewportUnconstrained = true // So Diagram positioning rules let the animation start off-screen
      animation.easing = go.Animation.EaseOutExpo
      animation.duration = 900
      // Fade "down", in other words, fade in from above
      animation.add(diagram, 'position', diagram.position.copy().offset(0, 200), diagram.position)
      animation.add(diagram, 'opacity', 0, 1)
      animation.start()
    }
  }
}
</script>

<style lang="scss" scoped>
#diagram-contanier {
  height: 100%;
  display: flex;
  flex-direction: column;
  border: 1px solid #eee;
  #diagram-tool {
    width: 100%;
    height: 40px;
    line-height: 40px;
    background-color: #dfdfdf;
    display: flex;
    align-items: center;
    justify-content: flex-end;
    padding-right: 20px;
  }

  #diagram-wrap {
    flex-grow: 1;
    height: 100%;
    display: flex;
    #diagram-palette {
      flex: 0 0 180px;
      height: 100%;
      // border-right: 1px solid #dfdfdf;
      background-color: #eee;
    }
    #diagram {
      width: 100%;
      flex-grow: 1;
    }

    ::v-deep canvas {
      outline: none;
    }
  }
}
</style>
