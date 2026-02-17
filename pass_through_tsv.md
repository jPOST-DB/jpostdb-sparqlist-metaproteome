# Pass through tsv

## Parameters

* `sample`
  * default: JPST000476_1
  
## `return`
```javascript
async ({sample})=>{
  const id = sample.match(/_(\d+)$/)[1];
  return await fetch("https://tools.jpostdb.org/subdb/metaproteome/data/JPST000476/" + id + "/open_search/functional_annotation/functions.tsv").then(r => r.text());
}
```
