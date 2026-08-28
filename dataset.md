# Dataset

## Parameters

* `project` (Opt.) (Req. project or sample)
  * default:
  * example: JPST000476
* `sample` (Opt.)
  * default: 
  * example: JPST000476_1
* `group`
  * default:
  * example: Psychiatry, Endocrinology

## `return`
```javascript
async ({project, sample, group, stanza}) => {
  if (! project) project = sample.match(/^(JPST\d+)/)[1];
  const project_meta = await fetch("project?project=" + project).then(r => r.json());
  const metadata = "https://tools.jpostdb.org/subdb/metaproteome/data/" + project + "/metadata.json";
  //console.log(metadata);
  let json = await fetch(metadata).then(r => r.json());
  if (sample) {
    for (let i = 0; i < json.length; i++) {
      //console.log(json[i].sample_name);
      if (json[i].sample_name == sample) {
        const count = await fetch("pep_count?sample=" + sample).then(r => r.json());
        json[i].sample_type = project_meta.sample_type;
        json[i].taxonomy = project_meta.taxonomy;
        json[i].taxonomy_label = project_meta.taxonomy_label;
        json[i].psm = count.psm;
        json[i].peptide = count.peptide;
        return [ json[i] ];
      }
    }
  } else {
    for (let i = 0; i < json.length; i++) {
      const count = await fetch("pep_count?sample=" + json[i].sample_name).then(r => r.json());
      json[i].sample_type = project_meta.sample_type;
      json[i].taxonomy = project_meta.taxonomy;
      json[i].taxonomy_label = project_meta.taxonomy_label;
      json[i].psm = count.psm;
      json[i].peptide = count.peptide;
    }
    if (group) {
      let res = [];
      json.sort((a, b) => {return a.sample_id - b.sample_id}).forEach(d => {
        if (d.group == group) res.push(d);
      });
      return res;
    }
    return json.sort((a, b) => {return a.sample_id - b.sample_id}).map(d => {
      d.url = "https://tools.jpostdb.org/subdb/metaproteome/?sample=" + d.sample_name;
      return d;
    });
  }
}
```