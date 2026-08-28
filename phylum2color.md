# Phylum to color

門ごとの表示色。設計方針:

* **同義名は完全に同じ色**にする (Firmicutes/Bacillota, Actinobacteriota/Actinomycetota,
  Proteobacteria/Pseudomonadota, Desulfobacterota/Thermodesulfobacteriota)。
* **分類的に近い群は同じ色相帯の連続した色**にする。Firmicutes 群 (GTDB の分割名 _A/_B/_C/_G) は
  1 本のローズ系ランプに並べ、古細菌 (Thermoplasmatota / Methanobacteriota) は紫系でまとめる。
* 群をまたぐ組み合わせは CIEDE2000 で ΔE >= 10 を確保する (積み上げ棒で隣り合っても判別できる下限)。
* `p__` は GTDB の門未割当プレースホルダ。灰色を明示的に割り当て、未登録門のフォールバックと区別する。

## `return`
```javascript
()=>{
   return {
     // --- Firmicutes / Bacillota 群: 同一色相のランプ (濃 -> 淡) ---
     Firmicutes: "#D9736A",
     Bacillota: "#D9736A",              // = Firmicutes (同義)
     Firmicutes_A: "#E59089",
     Firmicutes_B: "#EDA9A9",
     Firmicutes_C: "#F0BFC6",
     Firmicutes_G: "#F3D5DC",

     // --- 主要門 ---
     Bacteroidota: "#9FC4E8",

     Actinobacteriota: "#EED27A",
     Actinomycetota: "#EED27A",         // = Actinobacteriota (同義)

     Proteobacteria: "#7DCC74",
     Pseudomonadota: "#7DCC74",         // = Proteobacteria (同義)

     Desulfobacterota: "#8FD3C8",
     Thermodesulfobacteriota: "#8FD3C8", // = Desulfobacterota (同義)

     // --- その他の細菌 ---
     Fusobacteriota: "#5FA9D6",
     Verrucomicrobiota: "#A89BD8",
     Campylobacterota: "#C46FA8",
     Synergistota: "#4FB39C",
     Cyanobacteria: "#D9B07C",
     Fibrobacterota: "#C9A83F",
     Spirochaetota: "#7C7FC4",
     Elusimicrobiota: "#E2842C",
     Eremiobacterota: "#7FA05C",
     Patescibacteria: "#C7869B",

     // --- 古細菌: 紫系でまとめる ---
     Thermoplasmatota: "#B47ED8",
     Methanobacteriota: "#9C6BB5",

     // --- 門が未割当 (GTDB のプレースホルダ) ---
     p__: "#C4C4C4"
   };
}
```
