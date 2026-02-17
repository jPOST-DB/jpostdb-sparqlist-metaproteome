# string to color

## Parameters

* `str`
  * default:

## `return`
```javascript
({str})=>{
  
// Label (A..Z, AA..ZZ, etc) -> pastel-ish color
// Adjacent labels (by index) are well-separated using golden angle.

const GOLDEN_ANGLE = 137.50776405003785; // degrees

function lettersToIndex(label) {
  // Excel風: A=0, B=1, ... Z=25, AA=26, AB=27, ...
  // 非英字が混ざる例外にも対応: A-Z だけ拾う
  const s = String(label).toUpperCase().replace(/[^A-Z]/g, "");
  if (!s) return 0;

  let n = 0;
  for (let i = 0; i < s.length; i++) {
    n = n * 26 + (s.charCodeAt(i) - 64); // A=1..Z=26
  }
  return n - 1; // A->0
}

function hslToHex(h, s, l) {
  s /= 100; l /= 100;
  const c = (1 - Math.abs(2 * l - 1)) * s;
  const hh = ((h % 360) + 360) % 360 / 60;
  const x = c * (1 - Math.abs((hh % 2) - 1));
  let r = 0, g = 0, b = 0;

  if (hh < 1) [r, g, b] = [c, x, 0];
  else if (hh < 2) [r, g, b] = [x, c, 0];
  else if (hh < 3) [r, g, b] = [0, c, x];
  else if (hh < 4) [r, g, b] = [0, x, c];
  else if (hh < 5) [r, g, b] = [x, 0, c];
  else [r, g, b] = [c, 0, x];

  const m = l - c / 2;
  const toHex = (v) => Math.round((v + m) * 255).toString(16).padStart(2, "0");
  return `#${toHex(r)}${toHex(g)}${toHex(b)}`;
}

function pastelColorFromLabel(label, opts = {}) {
  const {
    saturation = 58, // 少し濃いめパステル
    lightness = 72,
    hueOffset = 0,
    // 文字列が例外（長い/非英字）でも “ある程度” 分散させたいなら下のhashMixをON
    hashMix = true,
  } = opts;

  const idx = lettersToIndex(label);

  // 連番の隣接を離す：idxごとにゴールデンアングルで進める
  let hue = (idx * GOLDEN_ANGLE + hueOffset) % 360;

  // 例外ラベル（非英字/長い）でも同じidxになりがちなので、必要なら少しだけ混ぜる
  // ※隣接分離の主成分はゴールデンアングルなので、微小に留める
  if (hashMix) {
    let h = 2166136261 >>> 0; // FNV-1a
    const str = String(label);
    for (let i = 0; i < str.length; i++) {
      h ^= str.charCodeAt(i);
      h = Math.imul(h, 16777619);
    }
    hue = (hue + ((h >>> 0) % 21) - 10) % 360; // ±10°だけ微調整
  }

  // 見分けやすさのため、S/Lも少しだけ揺らす（派手にしない範囲）
  const s = Math.max(0, Math.min(100, saturation + ((idx % 7) - 3))); // ±3
  const l = Math.max(0, Math.min(100, lightness + ((Math.floor(idx / 7) % 5) - 2))); // ±2

  return hslToHex(hue, s, l);
}
  
  return pastelColorFromLabel(str);
}
```