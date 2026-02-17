# MetaLab - Taxonomy tree for SunBurst graph

## Parameters

* `sample`
  * default: JPST000476_1

## `return`
```javascript
async ({sample}) => {
  sample = sample.replace("_", "/");
  const data = await fetch("https://tools.jpostdb.org/subdb/metaproteome/data/" + sample + "/open_search/taxonomy_analysis/Taxa.tsv").then(r => r.text());
  const phylum2color = await fetch("phylum2color").then(r => r.json());
  console.log(phylum2color);
  const list = data.split(/\n/);
  let res = [
    {id: 1, name: "Organisms", color: "#bbbbbb"}];
  let name2id = {};
  let count = 2;
  let chk = {};
  for (const d of list) {
    if (d.match(/^Name/)) continue;
    const tax = d.split(/\t/);
    let intensity = 0;
    intensity += parseFloat(tax[10]);
    let color = "#dddddd";
    if (tax[4] && phylum2color[tax[4]]) color = phylum2color[tax[4]];
    const name = tax[0];
    const rank = tax[1];
    //const size = Math.trunc(intensity);
    const size = intensity;
    if (!name2id[name]) name2id[name] = count++;
    const id = name2id[name];
    let p = false;
    let p_id = false;
    for (let i = 9; i > 1; i--) {
      if (tax[i] == name || !tax[i]) {
       // console.log([0, name, tax[i]]);
        continue;
      } else {
      //  console.log([1, name, tax[i]]);
        p = tax[i];
        if (!name2id[p]) name2id[p] = count++;
        p_id = name2id[p];
        break;
      }
    }
    if (size > 0 && !chk[name]) {
   // if (!chk[name]) {
      // "CAG-xxx" のような同じ Family, Genus 名を回避
      chk[name] = true;
      res.push({
        id: id,
        name: name,
        ...(rank == "Species" && {size: size}),
        color: color,
        parent: p_id ? p_id : 1
      });
    }
  }
  return res;
}
```