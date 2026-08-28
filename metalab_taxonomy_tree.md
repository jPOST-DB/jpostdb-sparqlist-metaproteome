# MetaLab - Taxonomy tree for SunBurst graph

## Parameters

* `sample`
  * default:
  * example: JPST000476_1

## `return`
```javascript
async ({sample}) => {
  sample = sample.replace("_", "/");
  let tsv = sample + "/Taxa.tsv";
  const data = await fetch("https://tools.jpostdb.org/subdb/metaproteome/data/" + tsv).then(r => r.text());
  //console.log(data);
  const phylum2color = await fetch("phylum2color").then(r => r.json());
  //console.log(phylum2color);
  const list = data.split(/\n/);
  let res = [
    {id: 1, name: "Organisms", color: "#bbbbbb"}];
  let name2id = {};
  let count = 2;
  let name_col = 0;
  let rank_col = 0;
  let phylum_col = 0;
  let intensity_col = 0;
  let chk = {};
  for (const d of list) {
    if (d.match(/^Name/)) {
        const items = d.split(/\t/);
        for (let i = 0; i < items.length; i++) {
          if (items[i] == "Name") {
            name_col = i;
          } else if (items[i] == "Rank") {
            rank_col = i;
          } else if (items[i].toLowerCase() == "phylum") {
            phylum_col = i;
          } else if (items[i].match(/^Intensity /)) {
            intensity_col = i;
            break;
          }
        } 
      continue;
    }
    const tax = d.split(/\t/);
    let intensity = 0;
    intensity += parseFloat(tax[intensity_col]);
    let color = "#dddddd";
    if (tax[phylum_col] && phylum2color[tax[phylum_col]]) color = phylum2color[tax[phylum_col]];
    const name = tax[name_col];
    const rank = tax[rank_col];
    //const size = Math.trunc(intensity);
    const size = intensity;
    if (!name2id[name]) name2id[name] = count++;
    const id = name2id[name];
    let p = false;
    let p_id = false;
    for (let i = (intensity_col - 1); i > rank_col; i--) {
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
        ...(rank.toLowerCase() == "species" && {size: size}),
        color: color,
        parent: p_id ? p_id : 1
      });
    }
  }
  return res;
}
```