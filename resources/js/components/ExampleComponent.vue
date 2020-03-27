<template>
  <div class="container">
    <div id="paper"></div>
    <button class="danger" @click="addShape('forward')">Forward</button>
    <button class="danger" @click="addShape('upward')">Upward</button>
    <button class="danger" @click="addShape('downward')">Downward</button>
    <button class="warning" @click="remove">Remove Shape</button>
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
      gridSize: 20,
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
    paper.$el.on('mousewheel DOMMouseScroll', function onMouseWheel(e) {
      //function onMouseWheel(e){
      e.preventDefault();
      e = e.originalEvent;

      var delta = Math.max(-1, Math.min(1, (e.wheelDelta || -e.detail))) / 50;
      var offsetX = (e.offsetX || e.clientX - $(this).offset().left);

      var offsetY = (e.offsetY || e.clientY - $(this).offset().top);
      var p = offsetToLocalPoint(offsetX, offsetY);
      var newScale = joint.V(paper.viewport).scale().sx + delta;
      console.log(' delta' + delta + ' ' + 'offsetX' + offsetX + 'offsety--' + offsetY + 'p' + p.x + 'newScale' + newScale)
      if (newScale > 0.1 && newScale < 2) {
        paper.setOrigin(0, 0);
        paper.scale(newScale, newScale, p.x, p.y);
      }
    });

     const offsetToLocalPoint = (x, y) => {
      var svgPoint = paper.svg.createSVGPoint();
      svgPoint.x = x;
      svgPoint.y = y;

      var pointTransformed = svgPoint.matrixTransform(paper.viewport.getCTM().inverse());
      return pointTransformed;
    }
  },
  methods: {
    link(source, target, direction, position) {
      /*
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
      */
     const link = new joint.shapes.standard.Link();
     let sourceLinkOptions = {}
     if (direction === 'downward') {
      sourceLinkOptions= {
            anchor: {
              name: 'bottomRight',
              args: {
                dx: position === 1 ? -5 : -10 * position, // multiply this with offset for each instance
                dy: -10
              }
            }
      }
     }
    if (direction === 'upward') {
      sourceLinkOptions= {
            anchor: {
              name: 'topRight',
              args: {
                dx: position === 1 ? -5 : -10 * position, // multiply this with offset for each instance
                dy: 0
              }
            }
      }
     }
     link.source(source, sourceLinkOptions);
     link.target(target);
     link.router('metro');
     link.connector('jumpover');
     link.addTo(this.graph);
     return link;
    },
    remove(){
      this.activeCell.cell.remove();
    },
    addShape(direction) {
      const X_OFFSET = 200;
      const Y_OFFSET = 180;
      let x = 50;
      let y = 200;
      if (this.activeCell) {
        const { x: previousX, y: previousY } = this.activeCell.cell.attributes.position;


        if (direction === 'forward') {
          x = previousX + X_OFFSET;
          y = previousY;
        } 

        if (direction === 'upward') {
          const position = this.activeCell.linkUpwards.length + 1; // 0, 1, 2
          if(position === 1) {
            x = previousX + (50 * position);
          } else {
            x = previousX;
          }
          y = previousY - Y_OFFSET * position;
        }

        if (direction === 'downward') {
          const position = this.activeCell.linkDownwards.length + 1; // 0, 1, 2
          if(position === 1) {
            x = previousX + (50 * position);
          } else {
            x = previousX;
          }
          
          y = previousY + Y_OFFSET * position;
        }
      }

      var cell = new joint.shapes.basic.Rect();
      cell.resize(50, 100);
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
        let position = 1;
        // determine which direction to link
        if (direction === 'forward') {
          previousCell.linkForwards = newCell;
        }
        
        if (direction === 'downward') {
          position = previousCell.linkDownwards.length + 1;
          previousCell.linkDownwards.push(newCell);
        }

        if (direction === 'upward') {
          position = previousCell.linkUpwards.length + 1;
          previousCell.linkUpwards.push(newCell);
        }
        this.link(previousCell.cell, cell, direction, position);
      }

      if (!this.activeCell) {
        this.rootCell = newCell;
      }
      this.activeCell = newCell;
      return cell;
    }
  }
};
</script>
