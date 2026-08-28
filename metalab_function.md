# MetaLab - Function category (EC, COG, NOG) intensity for graph

* single: req. 'sample'
* multi: req. 'project' + 'group'

## Parameters

* `project`
  * default:
  * example: JPST000476
* `sample`
  * default:
* `group`
  * default:
  * example: Control
* `category`
  * default:
  * example: COG, NOG, EC

## `return`
```javascript
async ({project, sample, group, category}) => {
  if (! project) project = sample.match(/^(JPST\d+)/)[1];
  let info = [];
  if (sample) info = await fetch("dataset?sample=" + sample).then(r => r.json());
  else info = await fetch("dataset?project=" + project + "&group=" + group).then(r => r.json());
  const function_label = await fetch("function_label?category=" + category).then(r => r.json());
  let col = 0;
  let res = [];
  for (let i = 0; i < info.length; i++) {
    const sample = info[i].sample_name;
    let tsv = project + "/" + info[i].sample_id + "/Annotated_proteins.tsv";
    const data = await fetch("https://tools.jpostdb.org/subdb/metaproteome/data/" + tsv).then(r => r.text());
    const list = data.split(/\n/);
    let intensity = {};
    intensity["unclassified"] = 0;
    let total = 0;
    for (const line of list) {
      if (line.match(/^Group_ID/)) {
        const items = line.split(/\t/);
        for (let i = 0; i < items.length; i++) {
          if (items[i] == "NOG category" && category == "NOG") {
            col = i;
            break;
          } else if (items[i] == "COG category" && category == "COG") {
            col = i;
            break;
          } else if (items[i] == "EC_id" && category == "EC") {
            col = i;
            break;
          } 
        } 
        continue;
      }
      const d = line.split(/\t/);
      if (!d[6] || !d[6].match(/^[\d\.]+$/)) continue;
      if (! d[col]) {
        intensity["unclassified"] += parseFloat(d[6]);
        total += parseFloat(d[6]);
        continue;
      }
      let letters = [];
      if (category == "EC") {
        letters = d[col].split(",").map(e => e.split("")[0]); // EC class (1st digit) array
        letters = [...new Set(letters)]; // unique class
      } else {
        letters = d[col].split(""); // COG, NOG
      }
      for (let l of letters) {
        if (intensity[l] === undefined) intensity[l] = 0;
        // double count mode
        intensity[l] += parseFloat(d[6]);
        total += parseFloat(d[6]);
        // equal division mode
        // intensity[l] += parseFloat(d[6]) / letters.length;
        // total += parseFloat(d[6]) / letters.length;
      }
    }
    function_label.category.forEach(d => {
      if (!intensity[d] || intensity[d] == 0) return 0;
      const value = intensity[d] * 100 / total;
      res.push(
        {
          sample: sample,
          category: d,
          label: function_label.info[d].label,
          color: function_label.info[d].color,
          intensity: intensity[d],
          value: value,
          show_value: parseFloat(value.toFixed(2))
        }
      );
    });
    if (intensity["unclassified"] > 0) {
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
