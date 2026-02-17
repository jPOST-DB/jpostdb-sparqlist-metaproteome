# MetaLab - Phylum intensity for graph

* single: req. 'sample'
* multi: req. 'project' + 'group'

## Parameters

* `project`
  * default: JPST000476
* `sample`
  * default:
* `group`
  * default:


## `return`
```javascript
async ({project, sample, group}) => {
  let info = [];
  if (sample) info = await fetch("dataset?sample=" + sample).then(r => r.json());
  else info = await fetch("dataset?project=" + project + "&group=" + group).then(r => r.json());
  const p2color = await fetch("phylum2color").then(r => r.json());
  let res = [];
  for (let i = 0; i < info.length; i++) {
    const sample = info[i].sample_name;
    const data = await fetch("https://tools.jpostdb.org/subdb/metaproteome/data/" + project + "/" + info[i].sample_id + "/open_search/taxonomy_analysis/Taxa.tsv").then(r => r.text());
    const list = data.split(/\n/);
    let total = 0;
    let r = [];
    for (const d of list) {
      if (d.match(/^Name/)) continue;
      const tax = d.split(/\t/);
      if (tax[1] != "Phylum") continue;
      let intensity = parseFloat(tax[10]);
      total += intensity;
      let color = "#dddddd";
      if (tax[0]) {
        color = p2color[tax[0]] ? p2color[tax[4]] : "#dddddd";
      }
      r.push(
        {
          sample: sample,
          category: tax[0],
          color: color,
          intensity: intensity
        }
      );
    }
    r.forEach(d => {
      d.value = d.intensity * 100 / total;
      d.show_value = parseFloat(d.value.toFixed(2));
      res.push(d);
    });
  }
  return res;
}
```
