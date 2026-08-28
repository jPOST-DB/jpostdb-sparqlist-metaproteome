# MetaLab - KO list

## Parameters

* `sample`
  * default:
  * example: JPST000476_1

## `return`
```javascript
async ({sample}) => {
  sample = sample.replace("_", "/");
  let tsv = sample + "/Annotated_proteins.tsv";
  const data = await fetch("https://tools.jpostdb.org/subdb/metaproteome/data/" + tsv).then(r => r.text());
  const list = data.split(/\n/);
  let intensity = {};
  let col = 24; // KO
  for (const line of list) {
    if (line.match(/^Group_ID/)) continue;
    const d = line.split(/\t/);
    if (!d[6] || parseFloat(d[6]) == 0 || !d[6].match(/^[\d\.]+$/)) continue;
    if (d[col] == "-") continue;
    const ko = d[col].split(",");
    ko.forEach(k => {
      k = k.replace("ko:", "");
      if (intensity[k] === undefined) intensity[k] = 0;
      // double count mode
      intensity[k] += parseFloat(d[6]);
      // equal division mode
      // intensity[k] += parseFloat(d[6]) / ko.length;
    });
  }
  let max = 0;
  for (let d of Object.keys(intensity)) {
    if (intensity[d] > max) max = intensity[d];
  }
  
  // linear interpolation
  function lerp(a, b, t) {
    return a + (b - a) * t;
  }

  function gradientBlueToRed(t) {
    // clamp 0..1
    t = Math.max(0, Math.min(1, t));

    // start: #57a2ff
    const start = { r: 0x57, g: 0xa2, b: 0xff };
    // end:   #ff5757
    const end   = { r: 0xff, g: 0x57, b: 0x57 };

    const r = Math.round(lerp(start.r, end.r, t));
    const g = Math.round(lerp(start.g, end.g, t));
    const b = Math.round(lerp(start.b, end.b, t));

    const toHex = (v) => v.toString(16).padStart(2, "0");
    return `#${toHex(r)}${toHex(g)}${toHex(b)}`;
  }
  
  let text = "";
  for (let d of Object.keys(intensity)) {
    text += d + " " + gradientBlueToRed( intensity[d] / max ) + "\n";
  }
  return text;
}
```