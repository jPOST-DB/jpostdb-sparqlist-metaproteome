# Dataset

## Parameters

* `slice`
  * default: {"group": "Slice1", "list": ["JPST000476_1", "JPST000476_2", "JPST000476_3"]}

## `return`
```javascript
async ({slice}) => {
  const slice_json = JSON.parse(slice);
  let project_meta = {};
  let sample_meta = {};
  let res = [];
  for (let sample of slice_json.list) {
    const project = sample.match(/^(JPST\d+)/)[1];
    if (! project_meta[project]) {
      project_meta[project] = await fetch("project?project=" + project).then(r => r.json());
      const metadata = "https://tools.jpostdb.org/subdb/metaproteome/data/" + project + "/metadata.json";
      sample_meta[project] = await fetch(metadata).then(r => r.json());
    }
    const json = sample_meta[project];
    for (let d of json) {
      if (d.sample_name != sample) continue;
      const count = await fetch("pep_count?sample=" + sample).then(r => r.json());
      d.url = "https://tools.jpostdb.org/subdb/metaproteome/?sample=" + d.sample_name;
      d.slice_group = slice_json.group;
      d.sample_type = project_meta[project].sample_type;
      d.taxonomy = project_meta[project].taxonomy;
      d.taxonomy_label = project_meta[project].taxonomy_label;
      d.psm = count.psm;
      d.peptide = count.peptide;
      res.push(d);
    }
  }
  return res;
}
```