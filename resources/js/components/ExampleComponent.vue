<template>
  <div class="container">
    <div id="paper"></div>
    <button class="danger" @click="addShape">Add Shape</button>
    <button class="warning" @click="addShape">Remove Shape</button>
  </div>
</template>

<script>
// import * as _ from 'lodash';
// import * as $ from 'backbone';
const joint = require("jointjs/dist/joint.js");

import "jointjs/dist/joint.css";

export default {
  data: {
    graph: null,
    cellMap: {}, 
    rootCell: null,
    selectedCell: null
  },

  created() {
    const vm = this;
    var graphInstance = new joint.dia.Graph();
    vm.graph = graphInstance;
  },
  mounted() {
    var paper = new joint.dia.Paper({
      el: $("#paper"),
      width: 1200,
      height: 800,
      gridSize: 1,
      model: this.graph,
      perpendicularLinks: true,
      restrictTranslate: true
    });
    paper.on('cell:pointerdown', (cellView, evt, x, y) => {
      if (this.cellMap[cellView.model.id]) {
        this.activeCell = this.cellMap[cellView.model.id];
      }
      //this.activeCell.cell.remove()
    }
);
  },
  methods: {
    link(source, target, breakpoints) {
      var cell = new joint.shapes.org.Arrow({
        source: { id: source.id },
        target: { id: target.id },
        vertices: breakpoints,
        attrs: {
          ".connection": {
            fill: "none",
            "stroke-linejoin": "round",
            "stroke-width": "2",
            stroke: "#4b4a67"
          }
        }
      });
      this.graph.addCell(cell);
      return cell;
    },
    addShape() {

      const X_OFFSET = 200;
      const Y_OFFSET = 0;
      let x = 50;
      let y = 200;
      if (this.activeCell) {
        const { x: previousX, y: previousY } = this.activeCell.cell.attributes.position;
        x = previousX + X_OFFSET;
        y = previousY + Y_OFFSET;
      }

      var cell = new joint.shapes.basic.Rect();
      cell.resize(100, 200);
      cell.position(x, y);
      cell.attr("root/title", "joint.shapes.standard.Rectangle");
      cell.attr("label/text", "Rectangle");
      cell.attr("body/fill", "lightblue");
      cell.addTo(this.graph);
    

      this.graph.addCell(cell);
      let newCell = { cell, linkForwards: null, linkUpwards: [], linkDownwards: []};
      this.cellMap = {
        [cell.id]: newCell,
        ...this.cellMap
      }
      const previousCell = this.activeCell;
      if (previousCell) {
        // determine which direction to link
        previousCell.linkForwards = newCell;
        this.link(previousCell.cell, cell);
      }

      if (!this.activeCell) {
        this.rootCell = newCell;
      }
      console.log(this.rootCell);
      this.activeCell = newCell;
      return cell;
    }
  }
};
</script>
