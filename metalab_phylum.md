# MetaLab - Phylum intensity for graph

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


## `return`
```javascript
async ({project, sample, group}) => {
  let info = [];
  if (sample) info = await fetch("dataset?sample=" + sample).then(r => r.json());
  else info = await fetch("dataset?project=" + project + "&group=" + group).then(r => r.json());
 // console.log(info);
  const p2color = await fetch("phylum2color").then(r => r.json());
  let res = [];
  for (let i = 0; i < info.length; i++) {
    const sample = info[i].sample_name;
    let tsv = project + "/" + info[i].sample_id + "/Taxa.tsv";
    const data = await fetch("https://tools.jpostdb.org/subdb/metaproteome/data/" + tsv).then(r => r.text());
    //console.log(data);
    const list = data.split(/\n/);
    let total = 0;
    let name_col = 0;
    let rank_col = 0;
    let phylum_col = 0;
    let intensity_col = 0;
    let r = [];
    for (const d of list) {
      if (d.match(/^Name/)) {
        const items = d.split(/\t/);
        for (let i = 0; i < items.length; i++) {
          if (items[i] == "Name") {
            name_col = i;
          } else if (items[i] == "Rank") {
            rank_col = i;
          } else if (items[i] == "phylum") {
            phylum_col = i;
          } else if (items[i].match(/^Intensity /)) {
            intensity_col = i;
            break;
          }
        }
        continue;
      }
      const tax = d.split(/\t/);
      if (!tax[rank_col] || (tax[rank_col] && tax[rank_col].toLowerCase() != "phylum")) continue;
      let intensity = parseFloat(tax[intensity_col]);
      total += intensity;
      let color = "#dddddd";
      if (tax[name_col]) {
        color = p2color[tax[name_col]] ? p2color[tax[phylum_col]] : "#dddddd";
      }
      r.push(
        {
          sample: sample,
          category: tax[name_col],
          color: color,
          intensity: intensity
        }
      );
    }
    r.sort((a,b) => a.category.localeCompare(b.category)).forEach(d => {
      d.value = d.intensity * 100 / total;
      d.show_value = parseFloat(d.value.toFixed(2));
      res.push(d);
    });
  }
  return res;
}
```
