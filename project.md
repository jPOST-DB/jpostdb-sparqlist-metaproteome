# Project

## Parameters

* `project` (Opt.)
  * default:
  
## `return`
```javascript
async ({project}) => {
  const metadata = "https://tools.jpostdb.org/subdb/metaproteome/data/projects.json";
  let json = await fetch(metadata).then(r => r.json());
  if (project) {
    for (let i = 0; i < json.length; i++) {
      if (json[i].project == project) return json[i];
    }
  } else {
    return json.sort((a, b) => {return a.sample_id - b.sample_id}).map(d => {
      d.url = "https://tools.jpostdb.org/subdb/metaproteome/?project=" + d.project;
      d.group_text = d.groups.join(", ");
      return d;
    });
  }
}
```