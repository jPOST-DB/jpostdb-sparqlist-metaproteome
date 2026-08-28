# MetaLab - Function category (EC, COG, NOG) intensity for graph

* single: req. 'sample'
* multi: req. 'project' + 'group'

## Parameters

* `project`
  * default: JPST999999
* `sample`
  * default: JPST999999_1
* `group`
  * default:
* `category`
  * default: EC

## `return`
```javascript
async ({project, sample, group, category}) => {
  let info = [];
  info = await fetch("dataset?sample=" + sample).then(r => r.json());
 // if (sample) info = await fetch("dataset?sample=" + sample).then(r => r.json());
//  else info = await fetch("dataset?project=" + project + "&group=" + group).then(r => r.json());
//  const function_label = await fetch("function_label?category=" + category).then(r => r.json());
  console.log(info);
/*  console.log(function_label );
  let col = 0;
  let res = [];
  for (let i = 0; i < info.length; i++) {
    const sample = info[i].sample_name;
    const data = await fetch("https://tools.jpostdb.org/subdb/metaproteome/data/" + project + "/" + info[i].sample_id + "/open_search/functional_annotation/functions.tsv").then(r => r.text());
  console.log(data);
  } */
  return 0;
}

```
