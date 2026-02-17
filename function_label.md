# COG / NOG categoly info

## Parameters

* `category`
  * default:
  * example: COG, NOG, EC
  
## `return`
```javascript
async ({category})=>{
  let info = {};
  let category_list = [];
  if (category == "EC") {
    category_list = ["1", "2", "3", "4", "5", "6", "7", "-"];
    info = {
      1: {color: "#FFC1CC", label: "Oxidoreductases"},
      2: {color: "#FFDAB9", label: "Transferases"},
      3: {color: "#FFFACD", label: "ydrolases"},
      4: {color: "#B2FBA5", label: "Lyases"},
      5: {color: "#87CEEB", label: "Isomerases"},
      6: {color: "#E6E6FA", label: "Ligases"},
      7: {color: "#B0E0EA", label: "Translocases"},
      '-': {color: "#dddddd", label: "Unclassified"}
    };
  } else { // COG, NOG
    const cog = "https://ftp.ncbi.nih.gov/pub/COG/COG2024/data/cog-24.fun.tab";
    const tsv = await fetch(cog).then(r => r.text());
    const list = tsv.split(/\n/);
    list.forEach(line => {
      if (line.match(/^\d/)) return 0;
      let d = line.split(/\t/);
      if (d[0].match(/[A-Z]/)) {
        category_list.push(d[0]);
        info[d[0]] = {class: d[1], color: "#" + d[2], label: d[3]};
      }
    });
  }
  return {info: info, category: category_list};
}
```