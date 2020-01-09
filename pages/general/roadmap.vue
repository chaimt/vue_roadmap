<template>
  <div id="div_template">
  </div>

</template>

<script>
  import * as d3 from "d3"
  import {hierarchy, pack} from 'd3-hierarchy'
  import {loadDataNodes, mergeUserNodesData, mergeUserNodeData, getUserStateColor} from '@/api/datamodel'
  // import {contextmenu} from '@/api/d3-v4-contextmenu'

  export default {
    data() {
      return {
        items: [
          {
            label: "info",
            items: [
              {
                label: "links",
                onClick: d => {
                  let nodeLinks = d.data.links;
                  if (nodeLinks != null) {
                    links.transition()
                      .duration(200)
                      .style("opacity", .6);
                    links.transition()
                      .duration(800)
                      .style("opacity", .9);
                    let linkList = nodeLinks.map(link => `<li><a href="${link.ref}" target="_blank">${link.name}</a><br /></li>`);

                    links.html('<p>\n' +
                      'Additional Links:<br />\n' +
                      '<ul>' +
                      linkList +
                      '</ul>' +
                      '</p>'
                    )
                      .style("left", (d3.event.pageX) + "px")
                      .style("top", (d3.event.pageY - 18) + "px");
                  }
                }
                // label: "order",
                // onClick: d => {
                // }
              }
            ]
          }
        ],
        loadData: [],
        flowers: [
          {
            name: 'Roses',
            amount: 25,
            color: '#cc2936'
          },
          {
            name: 'Tulips',
            amount: 40,
            color: '#00a03e'
          },
          {
            name: 'Daisies',
            amount: 15,
            color: '#2a93d4'
          },
          {
            name: 'Narcissuses',
            amount: 9,
            color: '#F7AD0A'
          }
        ]
      }
    },

    mounted() {
      // `this` points to the vm instance
      this.renderTree(this.items);
    },

    methods: {
      // async fetchData() {
      //   console.log('fetchData');
      //   let data = await d3.json("/db/roadmap_data.json");
      //   this.loadData = data;
      // },

      renderTree(items) {
        // console.log('renderTree');
        // // Set the dimensions and margins of the diagram
        let margin = {top: 30, right: 90, bottom: 30, left: 90},
          width = 1960 - margin.left - margin.right,
          height = 1300 - margin.top - margin.bottom;

        var roadmapNodes = null;
        var graphDirection = 'vertical';

        function loadData(treeFile, userFile, direction) {
          console.log('loadData');
          graphDirection = direction;
          d3.json("/db/roadmap_data.json").then(roadmap => {
            let arr = loadDataNodes(roadmap.children);
            roadmapNodes = new Map(arr.map(i => [i.id, i]));

            d3.json("/db/" + userFile).then(user => {
              arr = loadDataNodes(user.children);
              let userNodes = new Map(arr.map(i => [i.id, i]));

              mergeUserNodesData(roadmapNodes, userNodes);
              d3.json("/db/" + treeFile).then(treeData => {
                root = d3.hierarchy(treeData);
                root.x0 = 0;
                root.y0 = 0;
                update(root);
              });
            })
          });
        }

        loadData("tree_profile.json", "user_chaim.json","horizontal");

        const svg = d3.select(this.$el)
          .append('svg')
          .attr("width", width + margin.right + margin.left)
          .attr("height", height + margin.top + margin.bottom)
          .append('g')
          .attr("transform", "translate("
            + margin.left + "," + margin.top + ")");

        let tooltip = d3.select("#div_template")
          .append("div")
          .style("opacity", 0)
          .attr("class", "tooltip")
          .style("background-color", "white")
          .style("border", "solid")
          .style("border-width", "2px")
          .style("border-radius", "5px")
          .style("padding", "5px");

        let links = d3.select("#div_template")
          .append("div")
          .style("opacity", 0)
          .attr("class", "links")
          .style("background-color", "white")
          .style("border", "solid")
          .style("border-width", "2px")
          .style("border-radius", "5px")
          .style("padding", "5px");

        let mouseover = d => {
          links.transition()
            .duration(200)
            .style("opacity", 0);
          tooltip
            .style("opacity", 1)
          d3.select(this)
            .style("stroke", "black")
            .style("opacity", 1);
        }
        let mousemove = d => {
          links.transition()
            .duration(200)
            .style("opacity", 0);
          tooltip
            .html(d.data.tooltip)
            .style("top", (event.pageY) + "px")
            .style("left", (event.pageX) + "px");
        }
        let mouseleave = d => {
          tooltip
            .style("opacity", 0)
          d3.select(this)
            .style("stroke", getUserStateColor(d, "steelblue"))
            .style("opacity", 0.8)
        }

        let i = 0,
          duration = 750,
          root;

        // declares a tree layout and assigns the size
        let treemap = d3.tree().size([height, width]);

        // Collapse the node and all it's children
        function collapse(d) {
          if (d.children) {
            d._children = d.children
            d._children.forEach(collapse)
            d.children = null
          }
        }


        function update(source) {
          // Assigns the x and y position for the nodes
          let treeData = treemap(root);

          // Compute the new tree layout.
          let nodes = treeData.descendants(),
            links = treeData.descendants().slice(1);

          // Normalize for fixed-depth.
          nodes.forEach(d => {
            let roadmap = roadmapNodes.get(d.data.id);
            mergeUserNodeData(d.data, roadmap);
            d.y = d.depth * 180
          });

          // ****************** Nodes section ***************************
          // Update the nodes...
          let node = svg.selectAll('g.node')
            .data(nodes, d => {
              return d.id || (d.id = ++i);
            });

          // Enter any new modes at the parent's previous position.
          let nodeEnter = node.enter().append('g')
            .attr('class', 'node')
            .attr("transform", d => {
              return "translate(" + source.y0 + "," + source.x0 + ")";
            })
            // .on('contextmenu', d3.contextmenu(items))
            .on('click', click);

          // Add Circle for the nodes
          nodeEnter.append('circle')
            .attr('class', 'node')
            .attr('r', 1e-6)
            .style("fill", d => {
              return d._children ? "lightsteelblue" : "#fff";
            })
            .style("stroke", d => {
              return getUserStateColor(d, "steelblue");
            });

          // Add labels for the nodes
          let textNode = nodeEnter.append('text')
            .attr("dy", ".35em");
          if (graphDirection == 'vertical') {
            textNode
              .attr("y", d => {
                return d.children || d._children ? -13 : 13;
              })
              .attr("text-anchor", "middle");
          } else {
            textNode
              .attr("x", d => {
                return d.children || d._children ? -13 : 13;
              })
              .attr("text-anchor", d => {
                return d.children || d._children ? "end" : "start";
              });
          }
          textNode.text(d => {
            return d.data.name != null ? d.data.name : d.data.id + "[missing]";
          });

          //set up exercise node
          // let exerciseNode = nodeEnter
          //   .filter(d => {
          //     if (d.data.type == "exercise") {
          //       return true; //This item will be included in the selection
          //     } else {
          //       return false;
          //     }
          //   });
          // exerciseNode
          //   .append('rect')
          //   .attr('class', 'exercise')
          //   .attr('x', 50)
          //   .attr('width', 100)
          //   .attr('height', 40)
          //   .attr('stroke', 'black')
          //   .attr('fill', 'gold');
          // exerciseNode
          //   .append('text')
          //   .attr('class', 'exercise')
          //   .attr('x', 70)
          //   .attr("text-anchor", "middle")
          //   .attr("dy", ".35em")
          //   .attr('stroke', 'black')
          //   .text("chaim tes me");
          // // .text(d => {
          // //     return d.data.description != null ? d.data.description : d.data.id + "[missing]";
          // // });

          //set up order node
          let orderNode = nodeEnter
            .filter(d => {
              return d.data.order != null;
            });
          orderNode
            .append('circle')
            .attr('class', 'node')
            .attr('r', 10)
            .attr("transform", d => {
              return "translate(" + source.y0 + 16 + "," + source.x0 + 16 + ")";
            })
            .style("fill", "gold")
            .attr('stroke-width', '1px')
            .style("stroke", "black");
          orderNode
            .append('text')
            .attr('class', 'order')
            .attr("text-anchor", "middle")
            .attr("dy", ".35em")
            .attr("transform", d => {
              return "translate(" + source.y0 + 15 + "," + source.x0 + 15 + ")";
            })
            .attr('stroke', 'black')
            .text(d => {
              return d.data.order != null ? d.data.order : d.data.id + "[missing]";
            });

          // UPDATE
          let nodeUpdate = nodeEnter.merge(node);

          // Transition to the proper position for the node
          nodeUpdate.transition()
            .duration(duration)
            .attr("transform", d => {
              if (graphDirection == 'vertical')
                return "translate(" + d.x + "," + d.y + ")";
              else
                return "translate(" + d.y + "," + d.x + ")";
            });

          // Update the node attributes and style
          nodeUpdate.select('circle.node')
            .attr("r", d => {
              let relevance = d.data.relevance == null ? 0 : d.data.relevance;
              return 10 + relevance;
            })
            // .attr('r', 10)
            .style("fill", d => {
              return getUserStateColor(d, "#fff");
            })
            .style("stroke", d => {
              return d._children ? "PLUM" : getUserStateColor(d, "steelblue");
            })
            // .on('contextmenu', d3.contextmenu(items))
            .attr('cursor', 'pointer')
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave);

          // Remove any exiting nodes
          let nodeExit = node.exit().transition()
            .duration(duration)
            .attr("transform", d => {
              if (graphDirection == 'vertical')
                return "translate(" + source.x + "," + source.y + ")";
              else
                return "translate(" + source.y + "," + source.x + ")";
            })
            .remove();

          // On exit reduce the node circles size to 0
          nodeExit.select('circle')
            .attr('r', 1e-6);

          // On exit reduce the opacity of text labels
          nodeExit.select('text')
            .style('fill-opacity', 1e-6);

          // nodeExit.parents();
          // ****************** links section ***************************
          // Update the links...
          let link = svg.selectAll('path.link')
            .data(links, d => {
              return d.id;
            });

          // Enter any new links at the parent's previous position.
          let linkEnter = link.enter().insert('path', "g")
            .attr("class", "link")
            .style("stroke", d => {
              let color = d.data.color == null ? "lightgray" : d.data.color;
              return color;
            })
            .attr('d', d => {
              let o = {x: source.x0, y: source.y0}
              return diagonal(o, o)
            });

          // UPDATE
          let linkUpdate = linkEnter.merge(link);

          // Transition back to the parent element position
          linkUpdate.transition()
            .duration(duration)
            .attr('d', d => {
              return diagonal(d, d.parent)
            });

          // Remove any exiting links
          link.exit().transition()
            .duration(duration)
            .attr('d', d => {
              let o = {x: source.x, y: source.y}
              return diagonal(o, o)
            })
            .remove();

          // Store the old positions for transition.
          nodes.forEach(d => {
            d.x0 = d.x;
            d.y0 = d.y;
          });

          // Creates a curved (diagonal) path from parent to the child nodes
          function diagonal(s, d) {
            var path;
            if (graphDirection == 'vertical') {
              path = `M ${s.x} ${s.y}
            C ${(s.x + d.x) / 2} ${s.y},
              ${(s.x + d.x) / 2} ${d.y},
              ${d.x} ${d.y}`
            } else {
              path = `M ${s.y} ${s.x}
            C ${(s.y + d.y) / 2} ${s.x},
              ${(s.y + d.y) / 2} ${d.x},
              ${d.y} ${d.x}`
            }

            return path
          }

          // Toggle children on click.
          function click(d) {
            if (d.children) {
              d._children = d.children;
              d.children = null;
            } else {
              d.children = d._children;
              d._children = null;
            }
            update(d);
          }
        }
      }
    }


  }
</script>

<style>
  body {
    font: 16px -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica,
    Arial, sans-serif;
  }

  svg {
    display: block;
    margin: 0 auto;
  }

  .flower {
    transition: transform 0.2s ease-in-out;
    text-anchor: middle;
  }

  .flower__circle {
    transition: r 0.2s ease-in-out;
  }

  .flower__label {
    fill: #fff;
    font-weight: bold;
    text-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  }

  .controls {
    display: flex;
    justify-content: center;
    margin-top: 20px;
  }

  .control {
    display: inline-flex;
    flex-direction: column;
    margin: 0 4px;
  }

  .control label {
    font-size: 14px;
    font-weight: bold;
    margin-bottom: 4px;
  }

  .control input {
    display: block;
    font: inherit;
    width: 100px;
  }

  .node circle {
    fill: #fff;
    stroke: steelblue;
    stroke-width: 3px;
  }

  .node text {
    font: 12px sans-serif;
  }

  .link {
    fill: none;
    stroke: #ccc;
    stroke-width: 2px;
  }

  div.tooltip {
    position: absolute;
    text-align: center;
    padding: 2px;
    font: 12px sans-serif;
    background: lightsteelblue;
    border: 0px;
    border-radius: 8px;
    pointer-events: none;
  }

  .exercise {
    position: absolute;
    text-align: center;
    padding: 2px;
    font: 12px sans-serif;
    background: lightsteelblue;
    border: 0px;
    border-radius: 8px;
    pointer-events: none;
  }

  .order {
    position: absolute;
    text-align: center;
    padding: 2px;
    font: 12px sans-serif;
    background: lightsteelblue;
    border: 0px;
    border-radius: 8px;
    pointer-events: none;
  }

  div.links {
    position: absolute;
    text-align: center;
    padding: 2px;
    font: 12px sans-serif;
    background: red;
    border: 0px;
    border-radius: 8px;
  }
</style>
