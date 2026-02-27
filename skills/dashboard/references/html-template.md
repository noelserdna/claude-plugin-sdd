# HTML Dashboard Template

Self-contained HTML template for the SDD traceability dashboard. The skill replaces `{{DATA_JSON}}` with the serialized `traceability-graph.json` content.

## Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SDD Dashboard — {{PROJECT_NAME}}</title>
<style>
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
:root{
  --bg:#0f1117;--surface:#1a1d27;--surface2:#242837;--border:#2e3348;
  --text:#e4e7f1;--text2:#9498b0;--accent:#6c8cff;--accent2:#4a6aef;
  --green:#34d399;--yellow:#fbbf24;--red:#f87171;--orange:#fb923c;--gray:#6b7280;
  --purple:#a78bfa;
  --font:-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,sans-serif;
  --mono:'SF Mono',Consolas,'Courier New',monospace;
}
body{font-family:var(--font);background:var(--bg);color:var(--text);line-height:1.5;overflow-x:hidden}
a{color:var(--accent);text-decoration:none}
a:hover{text-decoration:underline}

/* Header */
.header{background:var(--surface);border-bottom:1px solid var(--border);padding:16px 24px;display:flex;align-items:center;justify-content:space-between;position:sticky;top:0;z-index:100}
.header h1{font-size:18px;font-weight:600}
.header h1 span{color:var(--accent);font-weight:700}
.header-meta{font-size:12px;color:var(--text2)}

/* Pipeline Bar */
.pipeline{padding:20px 24px;display:flex;gap:4px;align-items:center;overflow-x:auto}
.pipeline-stage{flex:1;min-width:120px;padding:10px 12px;border-radius:8px;text-align:center;cursor:pointer;transition:transform .15s,box-shadow .15s;border:1px solid transparent;position:relative}
.pipeline-stage:hover{transform:translateY(-2px);box-shadow:0 4px 12px rgba(0,0,0,.3)}
.pipeline-stage.active{border-color:var(--accent)}
.pipeline-stage .stage-name{font-size:11px;font-weight:600;text-transform:uppercase;letter-spacing:.5px}
.pipeline-stage .stage-count{font-size:20px;font-weight:700;margin-top:2px}
.pipeline-stage .stage-status{font-size:10px;margin-top:2px;opacity:.8}
.pipeline-arrow{color:var(--text2);font-size:16px;flex-shrink:0}
.st-done{background:#162e23;color:var(--green)}
.st-running{background:#2e2a10;color:var(--yellow)}
.st-error{background:#2e1616;color:var(--red)}
.st-stale{background:#2e2216;color:var(--orange)}
.st-pending{background:var(--surface);color:var(--gray)}
.st-unknown{background:var(--surface);color:var(--text2)}

/* Stats Cards */
.stats{display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:12px;padding:0 24px 16px}
.stat-card{background:var(--surface);border:1px solid var(--border);border-radius:8px;padding:14px 16px}
.stat-card .stat-label{font-size:11px;color:var(--text2);text-transform:uppercase;letter-spacing:.5px}
.stat-card .stat-value{font-size:28px;font-weight:700;margin-top:2px}
.stat-card .stat-sub{font-size:12px;color:var(--text2);margin-top:2px}
.stat-card.warn .stat-value{color:var(--red)}
.stat-card.good .stat-value{color:var(--green)}

/* Filter Bar */
.filters{position:sticky;top:52px;z-index:90;background:var(--bg);padding:10px 24px;display:flex;gap:8px;flex-wrap:wrap;align-items:center;border-bottom:1px solid var(--border)}
.filter-input{background:var(--surface);border:1px solid var(--border);border-radius:6px;padding:6px 10px;color:var(--text);font-size:13px;min-width:200px;outline:none}
.filter-input:focus{border-color:var(--accent)}
.filter-select{background:var(--surface);border:1px solid var(--border);border-radius:6px;padding:6px 8px;color:var(--text);font-size:13px;outline:none;cursor:pointer}
.filter-select:focus{border-color:var(--accent)}
.filter-badge{background:var(--surface2);border:1px solid var(--border);border-radius:12px;padding:2px 10px;font-size:11px;color:var(--text2)}

/* Traceability Table */
.table-wrap{padding:0 24px 24px;overflow-x:auto}
table{width:100%;border-collapse:collapse;font-size:13px}
thead{position:sticky;top:96px;z-index:80}
th{background:var(--surface2);color:var(--text2);font-size:11px;text-transform:uppercase;letter-spacing:.5px;padding:8px 10px;text-align:left;border-bottom:2px solid var(--border);white-space:nowrap;cursor:pointer;user-select:none}
th:hover{color:var(--text)}
th .sort-icon{margin-left:4px;opacity:.4}
th.sorted .sort-icon{opacity:1;color:var(--accent)}
td{padding:7px 10px;border-bottom:1px solid var(--border);vertical-align:top}
tr:hover td{background:var(--surface)}
tr.row-full td:last-child{color:var(--green)}
tr.row-partial td:last-child{color:var(--yellow)}
tr.row-none td:last-child{color:var(--red)}

/* Cell badges */
.cell-count{display:inline-flex;align-items:center;justify-content:center;min-width:24px;height:22px;border-radius:4px;font-size:12px;font-weight:600;padding:0 6px}
.cell-count.has{background:#1a2e3a;color:var(--accent)}
.cell-count.zero{color:var(--gray)}
.cell-id{font-family:var(--mono);font-size:12px;font-weight:600;color:var(--accent);cursor:pointer}
.cell-id:hover{text-decoration:underline}
.cell-title{max-width:250px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.cell-priority{font-size:11px;font-weight:600;padding:2px 8px;border-radius:4px;white-space:nowrap}
.cell-priority.must{background:#2e1616;color:var(--red)}
.cell-priority.should{background:#2e2a10;color:var(--yellow)}
.cell-priority.could{background:#162e23;color:var(--green)}
.cell-priority.wont{background:var(--surface2);color:var(--gray)}

/* Status badge */
.status-badge{font-size:11px;font-weight:600;padding:2px 8px;border-radius:4px}
.status-badge.full{background:#162e23;color:var(--green)}
.status-badge.partial{background:#2e2a10;color:var(--yellow)}
.status-badge.none{background:#2e1616;color:var(--red)}
.status-badge.orphan{background:#2e162e;color:var(--purple)}

/* Detail Panel */
.detail-overlay{position:fixed;top:0;right:0;bottom:0;left:0;background:rgba(0,0,0,.5);z-index:200;display:none;opacity:0;transition:opacity .2s}
.detail-overlay.open{display:block;opacity:1}
.detail-panel{position:fixed;top:0;right:0;bottom:0;width:420px;max-width:90vw;background:var(--surface);border-left:1px solid var(--border);z-index:201;transform:translateX(100%);transition:transform .25s ease;overflow-y:auto;padding:20px}
.detail-panel.open{transform:translateX(0)}
.detail-close{position:absolute;top:12px;right:12px;background:none;border:none;color:var(--text2);font-size:20px;cursor:pointer;padding:4px 8px;border-radius:4px}
.detail-close:hover{background:var(--surface2);color:var(--text)}
.detail-id{font-family:var(--mono);font-size:16px;font-weight:700;color:var(--accent);margin-bottom:4px}
.detail-title{font-size:14px;color:var(--text);margin-bottom:12px}
.detail-file{font-family:var(--mono);font-size:12px;color:var(--text2);margin-bottom:16px}
.detail-section{margin-bottom:16px}
.detail-section h3{font-size:12px;color:var(--text2);text-transform:uppercase;letter-spacing:.5px;margin-bottom:8px;padding-bottom:4px;border-bottom:1px solid var(--border)}
.detail-chain{list-style:none}
.detail-chain li{padding:4px 0 4px 16px;position:relative;font-size:13px}
.detail-chain li::before{content:'';position:absolute;left:4px;top:0;bottom:0;width:2px;background:var(--border)}
.detail-chain li::after{content:'';position:absolute;left:1px;top:11px;width:8px;height:8px;border-radius:50%;background:var(--accent);border:2px solid var(--surface)}
.detail-chain li .chain-id{font-family:var(--mono);font-weight:600;color:var(--accent);cursor:pointer}
.detail-chain li .chain-type{font-size:11px;color:var(--text2);margin-left:6px}
.detail-chain li .chain-file{font-size:11px;color:var(--text2);display:block}

/* Empty state */
.empty{text-align:center;padding:60px 24px;color:var(--text2)}
.empty h2{font-size:18px;margin-bottom:8px;color:var(--text)}
.empty p{font-size:14px}

/* Responsive */
@media(max-width:768px){
  .pipeline{flex-wrap:wrap}
  .pipeline-stage{min-width:90px}
  .stats{grid-template-columns:repeat(2,1fr)}
  .detail-panel{width:100%}
}
</style>
</head>
<body>

<div class="header">
  <h1><span>SDD</span> Dashboard</h1>
  <div class="header-meta">
    <span id="hdr-project"></span> &middot; <span id="hdr-time"></span>
  </div>
</div>

<div class="pipeline" id="pipeline"></div>

<div class="stats" id="stats"></div>

<div class="filters">
  <input type="text" class="filter-input" id="fSearch" placeholder="Search by ID or title...">
  <select class="filter-select" id="fStatus"><option value="">All Status</option><option value="full">Full</option><option value="partial">Partial</option><option value="none">No UCs</option><option value="orphan">Orphan</option></select>
  <select class="filter-select" id="fPriority"><option value="">All Priority</option></select>
  <select class="filter-select" id="fType"><option value="">All Types</option></select>
  <select class="filter-select" id="fStage"><option value="">All Stages</option></select>
  <span class="filter-badge" id="fCount"></span>
</div>

<div class="table-wrap">
  <table id="matrix">
    <thead>
      <tr>
        <th data-col="id">ID <span class="sort-icon">&#9650;</span></th>
        <th data-col="title">Title <span class="sort-icon">&#9650;</span></th>
        <th data-col="priority">Priority <span class="sort-icon">&#9650;</span></th>
        <th data-col="uc">UC <span class="sort-icon">&#9650;</span></th>
        <th data-col="wf">WF <span class="sort-icon">&#9650;</span></th>
        <th data-col="api">API <span class="sort-icon">&#9650;</span></th>
        <th data-col="bdd">BDD <span class="sort-icon">&#9650;</span></th>
        <th data-col="inv">INV <span class="sort-icon">&#9650;</span></th>
        <th data-col="adr">ADR <span class="sort-icon">&#9650;</span></th>
        <th data-col="task">TASK <span class="sort-icon">&#9650;</span></th>
        <th data-col="status">Status <span class="sort-icon">&#9650;</span></th>
      </tr>
    </thead>
    <tbody id="tbody"></tbody>
  </table>
</div>

<div class="empty" id="empty" style="display:none">
  <h2>No artifacts found</h2>
  <p>Run the SDD pipeline to generate traceability artifacts, then re-run <code>/sdd:dashboard</code>.</p>
</div>

<div class="detail-overlay" id="detailOverlay"></div>
<div class="detail-panel" id="detailPanel">
  <button class="detail-close" id="detailClose">&times;</button>
  <div class="detail-id" id="dId"></div>
  <div class="detail-title" id="dTitle"></div>
  <div class="detail-file" id="dFile"></div>
  <div class="detail-section">
    <h3>Incoming (referenced by)</h3>
    <ul class="detail-chain" id="dIncoming"></ul>
  </div>
  <div class="detail-section">
    <h3>Outgoing (references)</h3>
    <ul class="detail-chain" id="dOutgoing"></ul>
  </div>
</div>

<script>
(function(){
  "use strict";
  var DATA = {{DATA_JSON}};

  // --- Helpers ---
  var $ = function(s){return document.getElementById(s)};
  var ce = function(t){return document.createElement(t)};
  var tc = function(el,t){el.textContent=t;return el};

  // --- Init Header ---
  $("hdr-project").textContent = DATA.projectName || "SDD Project";
  $("hdr-time").textContent = DATA.generatedAt ? new Date(DATA.generatedAt).toLocaleString() : "";

  // --- Pipeline Bar ---
  var pipeEl = $("pipeline");
  var stageFilter = "";
  DATA.pipeline.stages.forEach(function(s, i){
    if(i > 0){
      var arrow = ce("span");
      arrow.className = "pipeline-arrow";
      arrow.textContent = "\u2192";
      pipeEl.appendChild(arrow);
    }
    var cls = "st-" + (s.status || "unknown");
    var div = ce("div");
    div.className = "pipeline-stage " + cls;
    div.dataset.stage = s.name;
    div.innerHTML = '<div class="stage-name">' + esc(s.name.replace(/-/g," ")) + '</div>'
      + '<div class="stage-count">' + (s.artifactCount || 0) + '</div>'
      + '<div class="stage-status">' + esc(s.status || "unknown") + '</div>';
    div.addEventListener("click", function(){
      if(stageFilter === s.name){stageFilter="";div.classList.remove("active");}
      else{
        document.querySelectorAll(".pipeline-stage").forEach(function(e){e.classList.remove("active")});
        stageFilter = s.name; div.classList.add("active");
      }
      applyFilters();
    });
    pipeEl.appendChild(div);
  });

  // --- Stats Cards ---
  var st = DATA.statistics || {};
  var statsEl = $("stats");
  addStat("Total Artifacts", st.totalArtifacts || 0, typeSummary(st.byType), "");
  var cov = st.traceabilityCoverage || {};
  var covPct = cov.reqsWithUCs ? cov.reqsWithUCs.percentage : 0;
  addStat("Traceability", covPct.toFixed(1) + "%", "REQs with UCs", covPct >= 80 ? "good" : "");
  var orphanCount = (st.orphans || []).length;
  addStat("Orphans", orphanCount, "Defined but unreferenced", orphanCount > 0 ? "warn" : "good");
  var brokenCount = (st.brokenReferences || []).length;
  addStat("Broken Refs", brokenCount, "Referenced but undefined", brokenCount > 0 ? "warn" : "good");

  function addStat(label, value, sub, cls){
    var d = ce("div"); d.className = "stat-card " + cls;
    d.innerHTML = '<div class="stat-label">' + esc(label) + '</div>'
      + '<div class="stat-value">' + esc(String(value)) + '</div>'
      + '<div class="stat-sub">' + esc(sub) + '</div>';
    statsEl.appendChild(d);
  }
  function typeSummary(bt){
    if(!bt) return "";
    var parts = [];
    Object.keys(bt).forEach(function(k){parts.push(k + ":" + bt[k])});
    return parts.slice(0,5).join(" ");
  }

  // --- Build indexes ---
  var artById = {};
  (DATA.artifacts || []).forEach(function(a){artById[a.id] = a});

  var incoming = {}; // target -> [{source, type, file, line}]
  var outgoing = {}; // source -> [{target, type, file, line}]
  (DATA.relationships || []).forEach(function(r){
    if(!incoming[r.target]) incoming[r.target] = [];
    incoming[r.target].push(r);
    if(!outgoing[r.source]) outgoing[r.source] = [];
    outgoing[r.source].push(r);
  });

  // --- REQ rows for matrix ---
  var reqs = (DATA.artifacts || []).filter(function(a){return a.type === "REQ"});
  var traceTypes = ["UC","WF","API","BDD","INV","ADR","TASK"];

  function countRelated(reqId, targetType){
    var rels = incoming[reqId] || [];
    var ids = {};
    rels.forEach(function(r){
      var art = artById[r.source];
      if(art && art.type === targetType) ids[r.source] = 1;
    });
    // Also check outgoing
    var rels2 = outgoing[reqId] || [];
    rels2.forEach(function(r){
      var art = artById[r.target];
      if(art && art.type === targetType) ids[r.target] = 1;
    });
    return Object.keys(ids).length;
  }

  function reqStatus(reqId){
    var uc = countRelated(reqId, "UC");
    var bdd = countRelated(reqId, "BDD");
    var task = countRelated(reqId, "TASK");
    if(uc > 0 && bdd > 0 && task > 0) return "full";
    if(uc > 0) return "partial";
    return "none";
  }

  // --- Populate filter dropdowns ---
  var priorities = {};
  var types = {};
  (DATA.artifacts || []).forEach(function(a){
    if(a.priority) priorities[a.priority] = 1;
    types[a.type] = 1;
  });
  Object.keys(priorities).sort().forEach(function(p){
    var o = ce("option"); o.value = p; o.textContent = p;
    $("fPriority").appendChild(o);
  });
  Object.keys(types).sort().forEach(function(t){
    var o = ce("option"); o.value = t; o.textContent = t;
    $("fType").appendChild(o);
  });
  DATA.pipeline.stages.forEach(function(s){
    var o = ce("option"); o.value = s.name; o.textContent = s.name.replace(/-/g," ");
    $("fStage").appendChild(o);
  });

  // --- Render table ---
  var rows = [];
  reqs.forEach(function(req){
    var counts = {};
    traceTypes.forEach(function(t){ counts[t] = countRelated(req.id, t) });
    rows.push({art: req, counts: counts, status: reqStatus(req.id)});
  });

  // If no REQs, show all artifact types
  if(reqs.length === 0){
    (DATA.artifacts || []).forEach(function(a){
      rows.push({art: a, counts: {}, status: "none"});
    });
  }

  var sortCol = "id";
  var sortDir = 1;

  function renderTable(filtered){
    var tb = $("tbody");
    tb.innerHTML = "";
    if(filtered.length === 0){
      $("empty").style.display = "block";
      $("fCount").textContent = "0 results";
      return;
    }
    $("empty").style.display = "none";
    $("fCount").textContent = filtered.length + " result" + (filtered.length !== 1 ? "s":"");

    filtered.forEach(function(row){
      var tr = ce("tr");
      tr.className = "row-" + row.status;

      // ID
      var tdId = ce("td");
      var idSpan = ce("span");
      idSpan.className = "cell-id";
      idSpan.textContent = row.art.id;
      idSpan.addEventListener("click", function(){ openDetail(row.art.id) });
      tdId.appendChild(idSpan);
      tr.appendChild(tdId);

      // Title
      var tdTitle = ce("td");
      tdTitle.className = "cell-title";
      tdTitle.textContent = row.art.title || "";
      tdTitle.title = row.art.title || "";
      tr.appendChild(tdTitle);

      // Priority
      var tdPri = ce("td");
      if(row.art.priority){
        var priSpan = ce("span");
        priSpan.className = "cell-priority " + priClass(row.art.priority);
        priSpan.textContent = row.art.priority;
        tdPri.appendChild(priSpan);
      }
      tr.appendChild(tdPri);

      // Trace counts
      traceTypes.forEach(function(t){
        var td = ce("td");
        var c = row.counts[t] || 0;
        var badge = ce("span");
        badge.className = "cell-count " + (c > 0 ? "has" : "zero");
        badge.textContent = c > 0 ? c : "\u2014";
        td.appendChild(badge);
        tr.appendChild(td);
      });

      // Status
      var tdSt = ce("td");
      var stBadge = ce("span");
      stBadge.className = "status-badge " + row.status;
      stBadge.textContent = row.status.charAt(0).toUpperCase() + row.status.slice(1);
      tdSt.appendChild(stBadge);
      tr.appendChild(tdSt);

      tb.appendChild(tr);
    });
  }

  function priClass(p){
    if(!p) return "";
    var l = p.toLowerCase();
    if(l.indexOf("must") >= 0 || l === "critical" || l === "m") return "must";
    if(l.indexOf("should") >= 0 || l === "high" || l === "s") return "should";
    if(l.indexOf("could") >= 0 || l === "medium" || l === "c") return "could";
    return "wont";
  }

  // --- Filtering ---
  var debounceTimer;
  function applyFilters(){
    clearTimeout(debounceTimer);
    debounceTimer = setTimeout(doFilter, 150);
  }

  function doFilter(){
    var search = $("fSearch").value.toLowerCase();
    var fSt = $("fStatus").value;
    var fPri = $("fPriority").value;
    var fType = $("fType").value;
    var fStg = $("fStage").value || stageFilter;

    var filtered = rows.filter(function(r){
      if(search && r.art.id.toLowerCase().indexOf(search) < 0 && (r.art.title || "").toLowerCase().indexOf(search) < 0) return false;
      if(fSt && r.status !== fSt) return false;
      if(fPri && r.art.priority !== fPri) return false;
      if(fType && r.art.type !== fType) return false;
      if(fStg && r.art.stage !== fStg) return false;
      return true;
    });

    sortRows(filtered);
    renderTable(filtered);
  }

  function sortRows(arr){
    arr.sort(function(a,b){
      var va, vb;
      if(sortCol === "id"){ va = a.art.id; vb = b.art.id; }
      else if(sortCol === "title"){ va = a.art.title || ""; vb = b.art.title || ""; }
      else if(sortCol === "priority"){ va = a.art.priority || "zzz"; vb = b.art.priority || "zzz"; }
      else if(sortCol === "status"){ va = a.status; vb = b.status; }
      else{
        var t = sortCol.toUpperCase();
        va = a.counts[t] || 0; vb = b.counts[t] || 0;
        return (va - vb) * sortDir;
      }
      if(va < vb) return -1 * sortDir;
      if(va > vb) return 1 * sortDir;
      return 0;
    });
  }

  // Event listeners
  $("fSearch").addEventListener("input", applyFilters);
  $("fStatus").addEventListener("change", applyFilters);
  $("fPriority").addEventListener("change", applyFilters);
  $("fType").addEventListener("change", applyFilters);
  $("fStage").addEventListener("change", function(){stageFilter="";applyFilters()});

  // Sort headers
  document.querySelectorAll("th[data-col]").forEach(function(th){
    th.addEventListener("click", function(){
      var col = th.dataset.col;
      if(sortCol === col) sortDir *= -1;
      else{sortCol = col; sortDir = 1;}
      document.querySelectorAll("th").forEach(function(h){h.classList.remove("sorted")});
      th.classList.add("sorted");
      th.querySelector(".sort-icon").textContent = sortDir === 1 ? "\u25B2" : "\u25BC";
      applyFilters();
    });
  });

  // --- Detail Panel ---
  function openDetail(id){
    var art = artById[id];
    if(!art) return;
    $("dId").textContent = art.id;
    $("dTitle").textContent = art.title || "";
    $("dFile").textContent = art.file + (art.line ? ":" + art.line : "");

    var inc = incoming[id] || [];
    var out = outgoing[id] || [];

    renderChain($("dIncoming"), inc, "source");
    renderChain($("dOutgoing"), out, "target");

    $("detailOverlay").classList.add("open");
    $("detailPanel").classList.add("open");
  }

  function renderChain(ul, rels, field){
    ul.innerHTML = "";
    if(rels.length === 0){
      var li = ce("li");
      li.textContent = "None";
      li.style.color = "var(--text2)";
      ul.appendChild(li);
      return;
    }
    rels.forEach(function(r){
      var li = ce("li");
      var idSpan = ce("span");
      idSpan.className = "chain-id";
      idSpan.textContent = r[field];
      idSpan.addEventListener("click", function(){ openDetail(r[field]) });
      li.appendChild(idSpan);

      var typeSpan = ce("span");
      typeSpan.className = "chain-type";
      typeSpan.textContent = r.type;
      li.appendChild(typeSpan);

      var fileSpan = ce("span");
      fileSpan.className = "chain-file";
      fileSpan.textContent = r.sourceFile + (r.line ? ":" + r.line : "");
      li.appendChild(fileSpan);

      ul.appendChild(li);
    });
  }

  $("detailClose").addEventListener("click", closeDetail);
  $("detailOverlay").addEventListener("click", closeDetail);
  function closeDetail(){
    $("detailOverlay").classList.remove("open");
    $("detailPanel").classList.remove("open");
  }
  document.addEventListener("keydown", function(e){
    if(e.key === "Escape") closeDetail();
  });

  // --- Escape HTML ---
  function esc(s){
    var d = ce("span");
    d.textContent = s;
    return d.innerHTML;
  }

  // --- Initial render ---
  doFilter();
})();
</script>
</body>
</html>
```

## Template Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `{{DATA_JSON}}` | Serialized `traceability-graph.json` content | `{"$schema":"traceability-graph-v1",...}` |
| `{{PROJECT_NAME}}` | Project name for title tag | `HackInHire` |

## Substitution Instructions

1. Read `dashboard/traceability-graph.json` as a string
2. Replace `{{DATA_JSON}}` with the raw JSON string (already valid JS object literal)
3. Replace `{{PROJECT_NAME}}` with the project name from the JSON
4. Write result to `dashboard/index.html`

## Performance Notes

- Uses `textContent` instead of `innerHTML` for data rendering (XSS-safe)
- Event delegation via handler attachment during row creation
- Filter debounce at 150ms prevents layout thrashing
- CSS Grid for responsive stats layout
- Sticky header + filter bar for scroll context
- No external dependencies (zero network requests)
