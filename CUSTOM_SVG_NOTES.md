# Custom SVG Design Notes

這份文件記錄目前 `patterns/custom/` 這批自製 SVG 的方向，之後如果要繼續產生、重做或分類，可以先回來看這裡。

## 目標

- 產生可無縫拼接的 SVG pattern。
- 尺寸主要抓 `60` 到 `100`，目前多數落在 `80`、`88`、`90`、`96`、`100`。
- SVG 本身只負責「乾淨圖形結構」，不要在 SVG 裡做 riso 質感、錯位、紙張、噪點、印刷位移。
- riso 的錯版、質感、混色、縮放、theme 色彩都交給 HTML render 處理。
- 每張 SVG 要能當效果器背景，不只是裝飾圖案。

## 基本 SVG 規則

- 使用 `<rect id="bg" ... fill="#f3eccf"/>` 作為背景色鉤子。
- 主圖形使用 `currentColor`，讓 theme 可以獨立控制顏色。
- 不使用 filter、gradient、mask、image、raster。
- 不使用隨機小圓點撐畫面。除非未來明確做 halftone 類，否則避免 `<circle>` 和一堆小 `r`。
- 邊界要有拼接意識：上/下、左/右的線段、塊面或曲線需要能接續。
- 優先用幾何結構、邊界呼應、中心節點、負形、切角來增加記憶點。

## 目前分類

### `grid`

偏塊面、負形、階梯、portal、pinwheel。

代表：

- `counterform-tiles.svg`
- `hinged-block-maze.svg`
- `negative-step-portals.svg`
- `pinwheel-counterblocks.svg`

適合效果：

- overdrive
- utility
- sequencer
- filter
- bitcrusher

### `hatch`

偏線路、折返、交錯、斷裂 rail。

代表：

- `broken-rail-lattice.svg`
- `crossing-gate-weave.svg`
- `folded-circuit-rails.svg`
- `switchback-thread.svg`

適合效果：

- tremolo
- delay subdivision
- routing
- gate
- modulation

### `orbit`

偏中心節點、邊界 key、loop、電路感。

代表：

- `corner-switchbacks.svg`
- `edge-key-circuit.svg`
- `junction-ribbons.svg`
- `temple-node-loop.svg`

適合效果：

- phaser
- filter
- delay
- multi-effect
- expression / CV

### `organic`

偏流體、軟結、河道、鏡像曲線。

代表：

- `bent-paper-river.svg`
- `mirror-flow-gate.svg`
- `river-archipelago.svg`
- `soft-knot-cells.svg`

適合效果：

- chorus
- vibrato
- reverb
- granular
- tape / analog modulation

### `scallop`

偏扇形、鱗片、拱門、旋轉 quarter。

代表：

- `corner-fan-lock.svg`
- `rotary-fan-quarters.svg`
- `scale-gear-field.svg`

適合效果：

- rotary
- chorus
- phaser
- vintage modulation

### `spark`

偏星芒、刀片、compass、尖角中心圖形。

代表：

- `blade-compass-repeat.svg`
- `cut-star-medallion.svg`

適合效果：

- fuzz
- distortion
- octave
- exciter

### `waves`

偏波形、拓樸線、braid、上下左右互相纏繞。

代表：

- `signal-braid.svg`
- `topographic-switchback.svg`

適合效果：

- chorus
- flanger
- delay
- reverb
- tape echo

## 這批 SVG 的設計重點

這一輪有參考 `patterns/pm/pm-new-*` 的語法，但不是直接複製。主要吸收的是：

- 圖案不只是 repeat，而是 tile 內有角色分工。
- 角落、邊界、中心都要有對位功能。
- 塊面和線段可以互相穿插，不需要每張都平均鋪滿。
- 負形比單純加圖案更有用。
- 一張小尺寸 SVG 可以用很少元素產生很強的節奏。

## 之前失敗的方向

- 太多小圓點，看起來像 AI 亂補細節。
- 幾何太平均，沒有主視覺或記憶點。
- 只像普通 pattern，不像效果器背景。
- 沒有跟聲音功能產生關聯。
- 單張圖案看起來可以，但拼接後沒有更大的節奏。

## 後續主題方向

如果要做效果器背景，可以用 JAM Pedals 類似的方法：

`效果名稱 -> 雙關 / 諧音 -> 角色 -> 局部圖案 -> theme 配色`

例子：

- `Delay` -> echo / slapback / llama / valley / route -> 回圈、路徑、殘影線。
- `Distortion` -> rat / rattler / fang / burn -> 鱗片、咬痕、尖角、裂紋。
- `Compressor` -> squash / dinosaur / press -> 被壓扁的塊、骨骼節點、壓痕。
- `Chorus` -> choir / coral / koi / waterfall -> 群體波紋、雙線偏移、流體。
- `Tremolo` -> monk / pulse / lantern -> 明暗節奏、斷續 rail、radial pulse。
- `Phaser` -> phase / pheasant / radar / eye -> 羽毛相位、同心切片、掃描弧。
- `Filter` -> turtle shell / funnel / mouth -> 殼紋、孔洞、開合曲線。
- `Bitcrusher` -> crab / beetle / pixel jaw -> 像素鉗、甲殼切面、方格破碎。

## 建議的新分類

目前 `custom/` 是圖形語法分類。之後如果開始做效果器主題，可以另外新增：

- `animal`
- `myth`
- `crest`
- `place`
- `machine`
- `ritual`
- `signal`
- `nature`

這些分類可以和現有幾何分類並存。現有分類負責圖形語法，新分類負責主題世界觀。

## 產生新 SVG 時的 prompt 規則

可以用這段當作內部生成規格：

```text
Create a seamless SVG pattern tile for a riso-style audio effect background.
Tile size must be 60-100px.
Use a <rect id="bg"> background with fill "#f3eccf".
Use currentColor for all foreground shapes.
Do not use filters, gradients, masks, raster images, random dots, or decorative small circles.
The SVG itself should be clean vector geometry only.
Texture, misregistration, scaling, and color themes will be handled by HTML rendering.
The tile must have clear edge continuity and should use corners, center, and borders as structural anchors.
Prefer bold negative shapes, cutouts, routes, folded lines, animal/sonic metaphors, or symbolic fragments over generic decoration.
```

## 更新 `files.json`

新增或刪除 SVG 後，需要重建 `patterns/files.json`，因為瀏覽器不能直接列目錄。

PowerShell:

```powershell
$root = (Resolve-Path patterns).Path
$files = Get-ChildItem -Path patterns -Recurse -File -Filter '*.svg' | ForEach-Object { $_.FullName.Substring($root.Length + 1).Replace('\','/') } | Sort-Object
$data = [ordered]@{ version = 1; basePath = './patterns/'; files = @($files) }
$data | ConvertTo-Json -Depth 4 | Set-Content -Path patterns\files.json -Encoding UTF8
```

## 檢查

基本 XML 檢查：

```powershell
$bad = @()
Get-ChildItem -Path patterns\custom -Recurse -File -Filter '*.svg' | ForEach-Object {
  try { [xml](Get-Content $_.FullName -Raw) | Out-Null } catch { $bad += $_.FullName }
}
if ($bad.Count) { $bad } else { 'XML OK' }
```

檢查是否又出現小圓點：

```powershell
Select-String -Path patterns\custom\**\*.svg -Pattern '<circle',' r=' -SimpleMatch
```
