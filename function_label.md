# COG / NOG / EC category info

`palette` で表示色を切り替えられる。

* `standard` (既定) ... 色相を完全に保持したまま明度を下げ彩度を上げる。NCBI / EC の指定色は `color_raw` に残る。
* `light` ... `standard` より控えめ。
* `raw` ... 指定色をそのまま返す。

指定色は淡すぎて白背景では判別しづらい (COG の D / Y / V は対白コントラスト比 1.05-1.07)。
変換は OKLCH 上でのアフィン写像なので、クラス内のランプ順序と相対間隔は厳密に保たれ、
「色相 = 機能クラス」という NCBI の設計はそのまま残る。

## Parameters

* `category`
  * default:
  * example: COG, NOG, EC
* `palette`
  * default: standard
  * example: standard, light, raw

## `return`
```javascript
async ({category, palette})=>{
  // ---- OKLCH ベースの配色調整 -------------------------------------------------
  // NCBI / EC の指定色は淡すぎて白背景で判別しづらい (COG の D/Y/V は対白コントラスト比 1.05-1.07)。
  // 色相は完全に保持したまま、明度を目標帯にアフィン写像し、彩度を kC 倍する。
  // アフィン写像なのでクラス内のランプ順序と相対間隔は厳密に保たれ、
  // 「色相 = 機能クラス」という NCBI の設計はそのまま残る。
  // palette=raw で元の指定色をそのまま返す。
  const PALETTE_PRESET = {
    raw:      null,
    light:    {L0: 0.68, L1: 0.86, kC: 1.3},
    standard: {L0: 0.60, L1: 0.76, kC: 1.4}
  };

  function _srgbToLinear(c) { return c <= 0.04045 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4); }
  function _linearToSrgb(c) {
    c = Math.min(1, Math.max(0, c));
    return c <= 0.0031308 ? 12.92 * c : 1.055 * Math.pow(c, 1 / 2.4) - 0.055;
  }
  function _oklabToLinear(L, a, b) {
    const l = Math.pow(L + 0.3963377774 * a + 0.2158037573 * b, 3);
    const m = Math.pow(L - 0.1055613458 * a - 0.0638541728 * b, 3);
    const s = Math.pow(L - 0.0894841775 * a - 1.2914855480 * b, 3);
    return [ 4.0767416621 * l - 3.3077115913 * m + 0.2309699292 * s,
            -1.2684380046 * l + 2.6097574011 * m - 0.3413193965 * s,
            -0.0041960863 * l - 0.7034186147 * m + 1.7076147010 * s];
  }
  function hexToOklch(hex) {
    const h = hex.replace("#", "");
    const [r, g, b] = [0, 2, 4].map(i => _srgbToLinear(parseInt(h.substr(i, 2), 16) / 255));
    const l = Math.cbrt(0.4122214708 * r + 0.5363325363 * g + 0.0514459929 * b);
    const m = Math.cbrt(0.2119034982 * r + 0.6806995451 * g + 0.1073969566 * b);
    const s = Math.cbrt(0.0883024619 * r + 0.2817188376 * g + 0.6299787005 * b);
    const L = 0.2104542553 * l + 0.7936177850 * m - 0.0040720468 * s;
    const A = 1.9779984951 * l - 2.4285922050 * m + 0.4505937099 * s;
    const B = 0.0259040371 * l + 0.7827717662 * m - 0.8086757660 * s;
    return {L: L, C: Math.hypot(A, B), H: (Math.atan2(B, A) * 180 / Math.PI + 360) % 360};
  }
  function oklchToHex(L, C, H) {
    const rad = H * Math.PI / 180;
    const rgb = _oklabToLinear(L, C * Math.cos(rad), C * Math.sin(rad));
    return "#" + rgb.map(v => Math.round(_linearToSrgb(v) * 255).toString(16).padStart(2, "0")).join("").toUpperCase();
  }
  function _inGamut(L, C, H) {
    const rad = H * Math.PI / 180;
    return _oklabToLinear(L, C * Math.cos(rad), C * Math.sin(rad))
           .every(v => v >= -0.0008 && v <= 1.0008);
  }
  // sRGB に収め、さらに 8bit 丸め後の色相ずれが 1 度未満になるまで彩度を下げる
  function _safe(L, C, H) {
    if (!_inGamut(L, C, H)) {
      let lo = 0, hi = C;
      for (let i = 0; i < 28; i++) { const mid = (lo + hi) / 2; if (_inGamut(L, mid, H)) lo = mid; else hi = mid; }
      C = lo;
    }
    for (let i = 0; i < 40; i++) {
      const hex = oklchToHex(L, C, H);
      const got = hexToOklch(hex);
      if (got.C < 0.004 || Math.abs(((got.H - H + 180) % 360 + 360) % 360 - 180) < 1.0) return hex;
      C *= 0.94;
    }
    return oklchToHex(L, C, H);
  }
  // info: {key: {color: "#rrggbb", ...}} を破壊的に書き換える
  function applyPalette(info, preset_name) {
    const p = PALETTE_PRESET[preset_name];
    if (!p) return info;                       // raw
    const keys = Object.keys(info);
    const lch = {};
    keys.forEach(k => { lch[k] = hexToOklch(info[k].color); });
    const Ls = keys.map(k => lch[k].L);
    const lo = Math.min(...Ls), hi = Math.max(...Ls);
    keys.forEach(k => {
      const t = (hi > lo) ? (lch[k].L - lo) / (hi - lo) : 0.5;
      info[k].color_raw = info[k].color;
      info[k].color = _safe(p.L0 + t * (p.L1 - p.L0), lch[k].C * p.kC, lch[k].H);
    });
    return info;
  }

  let info = {};
  let category_list = [];
  let class_label = {};   // COG/NOG の機能クラス名 (凡例をクラス単位でまとめるために返す)
  if (category == "EC") {
    category_list = ["1", "2", "3", "4", "5", "6", "7", "-"];
    info = {
      1: {color: "#FFC1CC", label: "Oxidoreductases"},
      2: {color: "#FFDAB9", label: "Transferases"},
      3: {color: "#FFFACD", label: "Hydrolases"},
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
    // NCBI の元ファイルで色列が欠けているカテゴリの補完。
    // 2024 版では X (Mobilome) のみが 3 列 (letter, class, label) で色を持たない。
    // class 2 のランプの続きになる色を充てている (既存 COG 色との最小 ΔE00 = 6.1)。
    const missing_color = {X: "8CE08C"};
    list.forEach(line => {
      if (! line.trim()) return 0;
      const d = line.split(/\t/);
      if (line.match(/^\d/)) { // クラス見出し行: "1<TAB>INFORMATION STORAGE AND PROCESSING"
        class_label[d[0]] = d[1];
        return 0;
      }
      if (! d[0].match(/^[A-Z]$/)) return 0;
      const has_color = (d.length >= 4);
      const color = has_color ? d[2] : (missing_color[d[0]] || "cccccc");
      const label = has_color ? d[3] : d[2];
      category_list.push(d[0]);
      info[d[0]] = {
        class: d[1],
        class_label: class_label[d[1]],
        color: "#" + color,
        label: label
      };
    });
  }
  applyPalette(info, palette || "standard");
  return {info: info, category: category_list, class: class_label};
}
```