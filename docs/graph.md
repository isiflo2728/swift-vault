# Knowledge Graph

> An interactive map of how every project and concept connects. Drag nodes, scroll to zoom, hover to highlight, click to open.

<style>
#graph-wrap {
  position: relative;
  background: #111;
  border-radius: 12px;
  overflow: hidden;
  margin: 1.5rem 0;
  border: 1px solid #2a2a2a;
}
#graph-legend {
  position: absolute;
  top: 14px;
  left: 16px;
  display: flex;
  align-items: center;
  gap: 16px;
  font-size: 12px;
  font-family: Inter, sans-serif;
  z-index: 10;
  background: rgba(17,17,17,0.85);
  padding: 6px 12px;
  border-radius: 8px;
  border: 1px solid #2a2a2a;
}
.leg-dot { display: inline-block; width: 10px; height: 10px; border-radius: 50%; margin-right: 4px; vertical-align: middle; }
.leg-project { background: #FF5722; }
.leg-concept { background: #4fc3f7; }
#graph-legend span { color: #aaa; }
#graph-hint {
  position: absolute;
  bottom: 12px;
  right: 14px;
  font-size: 11px;
  color: #555;
  font-family: Inter, sans-serif;
  z-index: 10;
}
#kg { width: 100%; display: block; }
#graph-tooltip {
  position: absolute;
  background: #1e1e1e;
  border: 1px solid #333;
  border-radius: 6px;
  padding: 6px 10px;
  font-size: 12px;
  font-family: Inter, sans-serif;
  color: #eee;
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.15s;
  z-index: 20;
  white-space: nowrap;
}
</style>

<div id="graph-wrap">
  <div id="graph-legend">
    <span><span class="leg-dot leg-project"></span>Project</span>
    <span><span class="leg-dot leg-concept"></span>Concept</span>
  </div>
  <div id="graph-hint">drag · scroll to zoom · click to open</div>
  <div id="graph-tooltip"></div>
  <svg id="kg" height="600"></svg>
</div>

<script src="https://d3js.org/d3.v7.min.js"></script>
<script>
(function () {
  const nodes = [
    { id: "WordScramble",      type: "project", url: "../projects/wordscramble/" },
    { id: "AnimationTechnique",type: "project", url: "../projects/animation-technique/" },
    { id: "iExpense",          type: "project", url: "../projects/iexpense/" },
    { id: "Moonshot",          type: "project", url: "../projects/moonshot/" },
    { id: "Cupcake Corner",    type: "project", url: "../projects/cupcake-corner/" },
    { id: "BookWorm",          type: "project", url: "../projects/bookworm/" },
    { id: "SwiftDataProject",  type: "project", url: "../projects/swiftdataproject/" },
    { id: "InstaFilter",       type: "project", url: "../projects/instafilter/" },

    { id: "@State",             type: "concept", url: "../concepts/state-data-flow/" },
    { id: "@Binding",           type: "concept", url: "../concepts/state-data-flow/" },
    { id: "@Observable",        type: "concept", url: "../concepts/state-data-flow/" },
    { id: "@Environment",       type: "concept", url: "../concepts/state-data-flow/" },
    { id: "@Query",             type: "concept", url: "../concepts/state-data-flow/" },
    { id: "@Bindable",          type: "concept", url: "../concepts/state-data-flow/" },
    { id: "@AppStorage",        type: "concept", url: "../concepts/persistence/" },
    { id: "NavigationStack",    type: "concept", url: "../concepts/navigation/" },
    { id: "Sheet",              type: "concept", url: "../concepts/navigation/" },
    { id: "navigationDestination", type: "concept", url: "../concepts/navigation/" },
    { id: "confirmationDialog", type: "concept", url: "../concepts/navigation/" },
    { id: "List",               type: "concept", url: "../concepts/layouts-lists/" },
    { id: "LazyVGrid",          type: "concept", url: "../concepts/layouts-lists/" },
    { id: "ScrollView",         type: "concept", url: "../concepts/layouts-lists/" },
    { id: "Implicit Animation", type: "concept", url: "../concepts/animations/" },
    { id: "Explicit Animation", type: "concept", url: "../concepts/animations/" },
    { id: "Transitions",        type: "concept", url: "../concepts/animations/" },
    { id: "DragGesture",        type: "concept", url: "../concepts/animations/" },
    { id: "UserDefaults",       type: "concept", url: "../concepts/persistence/" },
    { id: "SwiftData",          type: "concept", url: "../concepts/persistence/" },
    { id: "#Predicate",         type: "concept", url: "../concepts/persistence/" },
    { id: "SortDescriptor",     type: "concept", url: "../concepts/persistence/" },
    { id: "@Relationship",      type: "concept", url: "../concepts/persistence/" },
    { id: "CloudKit",           type: "concept", url: "../concepts/persistence/" },
    { id: "URLSession",         type: "concept", url: "../concepts/networking/" },
    { id: "Codable",            type: "concept", url: "../concepts/networking/" },
    { id: "async/await",        type: "concept", url: "../concepts/networking/" },
    { id: "AsyncImage",         type: "concept", url: "../concepts/networking/" },
    { id: "CIFilter",           type: "concept", url: "../concepts/core-image/" },
    { id: "CIContext",          type: "concept", url: "../concepts/core-image/" },
    { id: "PhotosPicker",       type: "concept", url: "../concepts/core-image/#photospicker" },
    { id: "ShareLink",          type: "concept", url: "../concepts/core-image/#sharing-the-result" },
    { id: "StoreKit",           type: "concept", url: "../projects/instafilter/#swapping-filters-storekitt-review-prompt" },
  ];

  const links = [
    { source: "WordScramble", target: "@State" },
    { source: "WordScramble", target: "List" },
    { source: "WordScramble", target: "NavigationStack" },

    { source: "AnimationTechnique", target: "Implicit Animation" },
    { source: "AnimationTechnique", target: "Explicit Animation" },
    { source: "AnimationTechnique", target: "Transitions" },
    { source: "AnimationTechnique", target: "DragGesture" },

    { source: "iExpense", target: "@Observable" },
    { source: "iExpense", target: "@State" },
    { source: "iExpense", target: "Sheet" },
    { source: "iExpense", target: "UserDefaults" },
    { source: "iExpense", target: "Codable" },

    { source: "Moonshot", target: "NavigationStack" },
    { source: "Moonshot", target: "navigationDestination" },
    { source: "Moonshot", target: "LazyVGrid" },
    { source: "Moonshot", target: "ScrollView" },
    { source: "Moonshot", target: "Codable" },

    { source: "Cupcake Corner", target: "@Observable" },
    { source: "Cupcake Corner", target: "URLSession" },
    { source: "Cupcake Corner", target: "Codable" },
    { source: "Cupcake Corner", target: "async/await" },
    { source: "Cupcake Corner", target: "AsyncImage" },
    { source: "Cupcake Corner", target: "navigationDestination" },

    { source: "BookWorm", target: "SwiftData" },
    { source: "BookWorm", target: "@Query" },
    { source: "BookWorm", target: "@Environment" },
    { source: "BookWorm", target: "@Binding" },
    { source: "BookWorm", target: "List" },

    { source: "SwiftDataProject", target: "SwiftData" },
    { source: "SwiftDataProject", target: "@Query" },
    { source: "SwiftDataProject", target: "@Environment" },
    { source: "SwiftDataProject", target: "@Bindable" },
    { source: "SwiftDataProject", target: "#Predicate" },
    { source: "SwiftDataProject", target: "SortDescriptor" },
    { source: "SwiftDataProject", target: "@Relationship" },
    { source: "SwiftDataProject", target: "NavigationStack" },
    { source: "SwiftDataProject", target: "navigationDestination" },
    { source: "SwiftDataProject", target: "CloudKit" },
    { source: "SwiftDataProject", target: "List" },

    { source: "InstaFilter", target: "@State" },
    { source: "InstaFilter", target: "@Environment" },
    { source: "InstaFilter", target: "@AppStorage" },
    { source: "InstaFilter", target: "async/await" },
    { source: "InstaFilter", target: "CIFilter" },
    { source: "InstaFilter", target: "CIContext" },
    { source: "InstaFilter", target: "PhotosPicker" },
    { source: "InstaFilter", target: "ShareLink" },
    { source: "InstaFilter", target: "StoreKit" },
    { source: "InstaFilter", target: "confirmationDialog" },
  ];

  const degree = {};
  nodes.forEach(n => degree[n.id] = 0);
  links.forEach(l => { degree[l.source]++; degree[l.target]++; });

  function radius(d) {
    const base = d.type === "project" ? 14 : 8;
    return base + Math.sqrt(degree[d.id]) * 3;
  }

  const svg = d3.select("#kg");
  const wrap = document.getElementById("graph-wrap");
  const W = wrap.clientWidth;
  const H = 600;
  svg.attr("width", W).attr("height", H);

  const g = svg.append("g");

  svg.call(
    d3.zoom()
      .scaleExtent([0.25, 4])
      .on("zoom", e => g.attr("transform", e.transform))
  );

  const sim = d3.forceSimulation(nodes)
    .force("link", d3.forceLink(links).id(d => d.id).distance(d => {
      return d.source.type === "project" ? 110 : 90;
    }))
    .force("charge", d3.forceManyBody().strength(-350))
    .force("center", d3.forceCenter(W / 2, H / 2))
    .force("collision", d3.forceCollide().radius(d => radius(d) + 12));

  const defs = svg.append("defs");
  const filter = defs.append("filter").attr("id", "glow");
  filter.append("feGaussianBlur").attr("stdDeviation", "3").attr("result", "blur");
  const feMerge = filter.append("feMerge");
  feMerge.append("feMergeNode").attr("in", "blur");
  feMerge.append("feMergeNode").attr("in", "SourceGraphic");

  const link = g.append("g")
    .selectAll("line")
    .data(links)
    .join("line")
    .attr("stroke", "#2e2e2e")
    .attr("stroke-width", 1.5);

  let dragMoved = false;

  const node = g.append("g")
    .selectAll("g")
    .data(nodes)
    .join("g")
    .style("cursor", "pointer")
    .call(
      d3.drag()
        .on("start", (e, d) => {
          dragMoved = false;
          if (!e.active) sim.alphaTarget(0.3).restart();
          d.fx = d.x;
          d.fy = d.y;
        })
        .on("drag", (e, d) => {
          dragMoved = true;
          d.fx = e.x;
          d.fy = e.y;
        })
        .on("end", (e, d) => {
          if (!e.active) sim.alphaTarget(0);
          d.fx = null;
          d.fy = null;
          if (!dragMoved) {
            window.location.href = d.url;
          }
          dragMoved = false;
        })
    );

  node.filter(d => d.type === "project")
    .append("circle")
    .attr("r", d => radius(d) + 5)
    .attr("fill", "none")
    .attr("stroke", "#FF5722")
    .attr("stroke-width", 1)
    .attr("opacity", 0.25)
    .attr("filter", "url(#glow)");

  node.append("circle")
    .attr("r", radius)
    .attr("fill", d => d.type === "project" ? "#FF5722" : "#4fc3f7")
    .attr("stroke", d => d.type === "project" ? "#ff8a65" : "#b3e5fc")
    .attr("stroke-width", d => d.type === "project" ? 2 : 1.5);

  node.append("text")
    .text(d => d.id)
    .attr("x", d => radius(d) + 6)
    .attr("y", 4)
    .attr("fill", d => d.type === "project" ? "#ff8a65" : "#e0f7fa")
    .attr("font-size", d => d.type === "project" ? "12px" : "10.5px")
    .attr("font-family", "Inter, JetBrains Mono, monospace")
    .attr("font-weight", d => d.type === "project" ? "600" : "400");

  const tooltip = d3.select("#graph-tooltip");

  node
    .on("mouseover", function (event, d) {
      const connected = new Set([d.id]);
      links.forEach(l => {
        if (l.source.id === d.id) connected.add(l.target.id);
        if (l.target.id === d.id) connected.add(l.source.id);
      });

      node.selectAll("circle:last-of-type")
        .attr("opacity", n => connected.has(n.id) ? 1 : 0.1);
      node.selectAll("text")
        .attr("opacity", n => connected.has(n.id) ? 1 : 0.1);
      node.selectAll("circle:first-of-type")
        .attr("opacity", n => connected.has(n.id) ? 0.25 : 0.03);

      link
        .attr("stroke", l => (l.source.id === d.id || l.target.id === d.id) ? "#FF5722" : "#1a1a1a")
        .attr("stroke-width", l => (l.source.id === d.id || l.target.id === d.id) ? 2.5 : 1)
        .attr("stroke-opacity", l => (l.source.id === d.id || l.target.id === d.id) ? 0.9 : 0.05);

      const tag = d.type === "project" ? "Project" : "Concept";
      const conns = links.filter(l => l.source.id === d.id || l.target.id === d.id).length;
      tooltip
        .style("opacity", 1)
        .html(`<strong>${d.id}</strong><br><span style="color:#888">${tag} · ${conns} connection${conns !== 1 ? "s" : ""}</span>`);
    })
    .on("mousemove", function (event) {
      const [mx, my] = d3.pointer(event, wrap);
      tooltip.style("left", (mx + 14) + "px").style("top", (my - 10) + "px");
    })
    .on("mouseout", function () {
      node.selectAll("circle:last-of-type").attr("opacity", 1);
      node.selectAll("text").attr("opacity", 1);
      node.selectAll("circle:first-of-type").attr("opacity", 0.25);
      link.attr("stroke", "#2e2e2e").attr("stroke-width", 1.5).attr("stroke-opacity", 1);
      tooltip.style("opacity", 0);
    });

  sim.on("tick", () => {
    link
      .attr("x1", d => d.source.x).attr("y1", d => d.source.y)
      .attr("x2", d => d.target.x).attr("y2", d => d.target.y);
    node.attr("transform", d => `translate(${d.x},${d.y})`);
  });
})();
</script>

---

## Concept Coverage by Project

|  | WordScramble | AnimationTechnique | iExpense | Moonshot | Cupcake Corner | BookWorm | SwiftDataProject | InstaFilter |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| @State | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| @Binding | | | | | | ✓ | | |
| @Bindable | | | | | | | ✓ | |
| @Observable | | | ✓ | | ✓ | | | |
| @Environment | | | | | | ✓ | ✓ | ✓ |
| @Query | | | | | | ✓ | ✓ | |
| @Relationship | | | | | | | ✓ | |
| @AppStorage | | | | | | | | ✓ |
| NavigationStack | ✓ | | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Sheet | | | ✓ | | | | | |
| navigationDestination | | | | ✓ | ✓ | | ✓ | |
| confirmationDialog | | | | | | | | ✓ |
| List | ✓ | | ✓ | | | ✓ | ✓ | |
| LazyVGrid | | | | ✓ | | | | |
| ScrollView | | | | ✓ | | | | |
| Implicit Animation | | ✓ | | | | | | |
| Explicit Animation | | ✓ | | | | | | |
| Transitions | | ✓ | | | | | | |
| DragGesture | | ✓ | | | | | | |
| UserDefaults | | | ✓ | | | | | |
| SwiftData | | | | | | ✓ | ✓ | |
| #Predicate | | | | | | | ✓ | |
| SortDescriptor | | | | | | | ✓ | |
| CloudKit | | | | | | | ✓ | |
| Codable | | | ✓ | ✓ | ✓ | | | |
| URLSession | | | | | ✓ | | | |
| async/await | | | | | ✓ | | | ✓ |
| AsyncImage | | | | | ✓ | | | |
| CIFilter | | | | | | | | ✓ |
| CIContext | | | | | | | | ✓ |
| PhotosPicker | | | | | | | | ✓ |
| ShareLink | | | | | | | | ✓ |
| StoreKit | | | | | | | | ✓ |