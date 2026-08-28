# Peptide / PSM count

## Parameters

* `sample`
  * default:

## `return`
```javascript
async ({sample})=>{
  sample = sample.replace("_", "/");
  let tsv = sample + "/final_summary.tsv";
  const api = "https://tools.jpostdb.org/subdb/metaproteome/data/" + tsv;
  console.log(api);
  const text = await fetch(api).then(r => r.text());
  const list = text.split(/\n/);
  for (let line of list) {
    if (! line.match(/^Total/)) continue;
    const d = line.split(/\t/);
    return {psm: parseInt(d[3]), peptide: parseInt(d[5])};
  }
}
```