# MetaLab - Peptide count for graph

* for multi

## Parameters

* `project`
  * default:
* `group`
  * defaulr:
  
## `return`
```javascript
async ({project, group})=>{
  const data = await fetch("dataset?project=" + project + "&group=" + group).then(r => r.json());
  return data.map(d => {
    return {
      sample: d.sample_name,
      category: "Peptides",
      label: "",
      color: "#9ddae0",
      value: d.peptide,
      show_value: d.peptide
    };
  });  
}
```