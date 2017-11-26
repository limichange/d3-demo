<template>
  <div class="hello">
    <h1>D3</h1>
    <svg width="960" height="600"></svg>
  </div>
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'IndexPage',
  data () {
    return {
    }
  },
  mounted () {
    const svg = d3.select('svg')
    const width = +svg.attr('width')
    const height = +svg.attr('height')

    console.log(d3)

    var color = d3.scaleOrdinal(d3.schemeCategory20)

    const simulation = d3.forceSimulation()
        .force('link', d3.forceLink().id(function(d) { return d.id }))
        .force('charge', d3.forceManyBody())
        .force('center', d3.forceCenter(width / 2, height / 2))

    d3.json('/static/miserables.json', function(error, graph) {
      if (error) throw error

      const link = svg
          .append('g')
          .attr('class', 'links')
          .selectAll('line')
          .data(graph.links)
          .enter()
            .append('line')
            .attr('stroke-width', d => d.value * .2)

      const node = svg.append('g')
          .attr('class', 'nodes')
          .selectAll('rect')
          .data(graph.nodes)
          .enter()
            .append('g')
            .attr('class', 'node')
            .call(
              d3.drag()
                .on('start', dragstarted)
                .on('drag', dragged)
                .on('end', dragended))

      d3
        .selectAll('g.node')
        .append('rect')
        .attr('width', 20)
        .attr('height', 20)
        .attr('fill', d => color(d.group))

      d3
        .selectAll('g.node')
        .append('text')
        .text(d => d.id)

      simulation
        .nodes(graph.nodes)
        .on('tick', ticked)

      simulation
        .force('link')
        .links(graph.links)

      function ticked() {
        link
          .attr('x1', d => d.source.x)
          .attr('y1', d => d.source.y)
          .attr('x2', d => d.target.x)
          .attr('y2', d => d.target.y)

        node
          .attr('cx', d => d.x)
          .attr('cy', d => d.y)

        node.selectAll('rect')
            .attr('x', d => d.x)
            .attr('y', d => d.y)

        node.selectAll('text')
            .attr('x', d => d.x + 22)
            .attr('y', d => d.y + 15)

      }
    })

    function dragstarted(d) {
      if (!d3.event.active) simulation.alphaTarget(0.3).restart()
      d.fx = d.x
      d.fy = d.y
    }

    function dragged(d) {
      d.fx = d3.event.x
      d.fy = d3.event.y
    }

    function dragended(d) {
      if (!d3.event.active) simulation.alphaTarget(0)
      d.fx = null
      d.fy = null
    }
  }
}
</script>

<style>
.links line {
  stroke: #999;
  stroke-opacity: .9;
}

.nodes rect {
  stroke: #fff;
  stroke-width: 1.3px;
}

</style>
