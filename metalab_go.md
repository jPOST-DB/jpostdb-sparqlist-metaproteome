# MetaLab - GO category intensity for graph

* single: req. 'sample'
* multi: req. 'project' + 'group'

## Parameters

* `project`
  * default: JPST000476
* `sample`
  * default:
* `group`
  * default: Control
* `go`
  * default: 0008150
  * example: 0003674, 0005575

## Endpoint

https://tools.jpostdb.org/proxy/sparql

## `go_list`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX go: <http://purl.obolibrary.org/obo/GO_>
SELECT DISTINCT ?go ?label
WHERE {
  ?go rdfs:subClassOf go:{{go}} ;
     rdfs:label ?label .
  FILTER(REGEX(STR(?go), "GO_"))
}
ORDER BY ?go
```

## `return`
```javascript
async ({project, sample, group, go_list}) => {
  let go = {};
  let color = {};
  let go_order = [];
  const alpha = ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O" ,"P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z"];
  for (let i = 0; i < go_list.results.bindings.length; i++) {
    const d = go_list.results.bindings[i];
    const id = d.go.value.replace(/http:\/\/purl.obolibrary.org\/obo\/GO_/, "GO:");
    go_order.push(id);
    go[id] = d.label.value;
    color[id] = await fetch("str2color?str=" + alpha[i]).then(r => r.text());
  }
  let info = [];
  if (sample) info = await fetch("dataset?sample=" + sample).then(r => r.json());
  else info = await fetch("dataset?project=" + project + "&group=" + group).then(r => r.json());
  const col = 17; // GO
  let res = [];
  for (let i = 0; i < info.length; i++) {
    const sample = info[i].sample_name;
    const data = await fetch("https://tools.jpostdb.org/subdb/metaproteome/data/" + project + "/" + info[i].sample_id + "/open_search/functional_annotation/functions.tsv").then(r => r.text());
    const list = data.split(/\n/);
    let intensity = {};
    intensity["unclassified"] = 0;
    let total = 0;
    for (const line of list) {
      if (line.match(/^Group_ID/)) continue;
      const d = line.split(/\t/);
      if (!d[6] || !d[6].match(/^[\d\.]+$/)) continue;
      let chk2 = false;
      let gos = d[col].split(",");
      for (let g of gos) {
        if (go[g]) {
          if (intensity[g] === undefined) intensity[g] = 0;
          // double count mode
          intensity[g] += parseFloat(d[6]);
          total += parseFloat(d[6]);
          // equal division mode
          // intensity[g] += parseFloat(d[6]) / gos.length;
          // total += parseFloat(d[6]) / gos.length;
          chk2 = true;
        }
      }
      if (! chk2) {
        intensity["unclassified"] += parseFloat(d[6]);
        total += parseFloat(d[6]);        
      }
    }
    let chk = false;
    go_order.forEach(d => {
      if (!intensity[d] || intensity[d] == 0) return 0;
      chk = true;
      const value = intensity[d] * 100 / total;
      res.push(
        {
          sample: sample,
          category: d,
          label: go[d],
          color: color[d],
          intensity: intensity[d],
          value: value,
          show_value: parseFloat(value.toFixed(2))
        }
      );
    });
    if (intensity["unclassified"] > 0 || !chk) {
      const value = intensity["unclassified"] * 100 / total;
      res.push(
        {
          sample: sample,
          category: "-",
          label: "Unclassified",
          color: "#dddddd",
          intensity: intensity["unclassified"],
          value: value,
          show_value: parseFloat(value.toFixed(2))
        }
      );
    }
  }
  return res;
}
```