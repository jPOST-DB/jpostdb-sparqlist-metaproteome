# Pass through tsv

## Parameters

* `sample`
  * default:
  * example: JPST000476_1
  
## `return`
```javascript
async ({sample})=>{
  const tmp = sample.match(/(.+)_(\d+)$/);
  return await fetch("https://tools.jpostdb.org/subdb/metaproteome/data/" + tmp[1] + "/" + tmp[2] + "/Annotated_proteins.tsv").then(r => r.text());
}
```
