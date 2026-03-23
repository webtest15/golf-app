<script>
  let step = 1;
  let count = 6;
  let players = [];
  let groups = [];
  let startInfo = [];
  let selected = null;

  function goToStep2() {
    players = Array.from({ length: count }, (_, i) => ({
      no: i + 1, name: "", team: "", hdcp: 0
    }));
    step = 2;
  }

  function addPlayer() {
    players = [...players, { no: players.length + 1, name: "", team: "", hdcp: 0 }];
  }

  function goToStep3() {
    groups = makeBalancedGroups(players);
    startInfo = groups.map(() => ({ course: "OUT", time: "" }));
    step = 3;
  }

  function makeBalancedGroups(list) {
    const sorted = [...list].sort((a, b) => a.hdcp - b.hdcp);
    const perGroup = 4;
    const groupCount = Math.ceil(sorted.length / perGroup);
    const result = Array.from({ length: groupCount }, () => []);
    let index = 0;
    for (let i = 0; i < sorted.length; i++) {
      result[index].push(sorted[i]);
      index = (index + 1) % groupCount;
    }
    return result;
  }

  function onPersonClick(gIndex, pIndex) {
    if (!selected) { selected = { gIndex, pIndex }; return; }
    const a = selected;
    const b = { gIndex, pIndex };
    if (a.gIndex === b.gIndex && a.pIndex === b.pIndex) { selected = null; return; }
    swap(a, b);
    selected = null;
  }

  function onGroupClick(gIndex, e) {
    if (e.target.closest("li")) return;
    if (!selected) return;
    const { gIndex: fromG, pIndex } = selected;
    if (fromG === gIndex) { selected = null; return; }
    const person = groups[fromG][pIndex];
    const newFrom = [...groups[fromG]];
    newFrom.splice(pIndex, 1);
    const newTo = [...groups[gIndex], person];
    groups = groups.map((g, i) => i === fromG ? newFrom : i === gIndex ? newTo : g);
    selected = null;
  }

  function onEmptySlotClick(gIndex) {
    if (!selected) return;
    const { gIndex: fromG, pIndex } = selected;
    if (fromG === gIndex) { selected = null; return; }
    const person = groups[fromG][pIndex];
    const newFrom = [...groups[fromG]];
    newFrom.splice(pIndex, 1);
    const newTo = [...groups[gIndex], person];
    groups = groups.map((g, i) => i === fromG ? newFrom : i === gIndex ? newTo : g);
    selected = null;
  }

  function swap(a, b) {
    const newGroups = groups.map(g => [...g]);
    const temp = newGroups[a.gIndex][a.pIndex];
    newGroups[a.gIndex][a.pIndex] = newGroups[b.gIndex][b.pIndex];
    newGroups[b.gIndex][b.pIndex] = temp;
    groups = newGroups;
  }

  // ★ 画像保存
  let captureTarget;
  let saving = false;

  async function saveImage() {
    if (saving) return;
    saving = true;

    if (!window.html2canvas) {
      await new Promise((resolve, reject) => {
        const s = document.createElement("script");
        s.src = "https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js";
        s.onload = resolve;
        s.onerror = reject;
        document.head.appendChild(s);
      });
    }

    try {
      const canvas = await window.html2canvas(captureTarget, {
        backgroundColor: "#f0f4f8",
        scale: 2,
        useCORS: true,
        logging: false
      });

      const link = document.createElement("a");
      link.download = "組み合わせ表.png";
      link.href = canvas.toDataURL("image/png");
      link.click();
    } catch (e) {
      alert("画像の保存に失敗しました。");
      console.error(e);
    } finally {
      saving = false;
    }
  }
</script>

<main>
  <h1>test</h1>

  <!-- STEP 1：人数選択 -->
  {#if step === 1}
    <div class="card">
      <h2>人数選択</h2>
      <p>人数を指定してください。</p>
      <select bind:value={count}>
        {#each [3,4,5,6,7,8,9,10,12,16] as n}
          <option value={n}>{n} 人</option>
        {/each}
      </select>
      <button on:click={goToStep2}>次へ</button>
    </div>
  {/if}

  <!-- STEP 2：参加者入力 -->
  {#if step === 2}
    <div class="card">
      <h2>参加者入力</h2>
      <p>参加者の名前・所属・HDCP を入力してください。</p>
      <div class="table">
        <div class="header">
          <div>No</div><div>名前</div><!--<div>所属</div>--><div>HDCP</div>
        </div>
        {#each players as p, i}
          <div class="row">
            <div>{p.no}</div>
            <input placeholder="名前" bind:value={p.name} />
            <!--<input placeholder="所属" bind:value={p.team} />-->
            <input type="number" bind:value={p.hdcp} />
          </div>
        {/each}
      </div>
      <button class="add" on:click={addPlayer}>＋ 追加</button>
      <div class="buttons">
        <button on:click={() => step = 1}>戻る</button>
        <button on:click={goToStep3}>次へ</button>
      </div>
    </div>
  {/if}

  <!-- STEP 3：組み合わせ結果 -->
  {#if step === 3}
    <div class="card">
      <h2>組み合わせ表の完成！</h2>

      <!-- ★ キャプチャ対象（h1 を除いてここだけ保存） -->
      <div class="capture-area" bind:this={captureTarget}>
        <div class="capture-title">⛳ 組み合わせ表</div>

        <div class="groups">
          {#each groups as group, gIndex}
            <div class="group" on:click={(e) => onGroupClick(gIndex, e)}>
              <h3>{gIndex + 1} 組</h3>

              <div class="start-settings">
                <select bind:value={startInfo[gIndex].course}>
                  <option value="OUT">OUT</option>
                  <option value="IN">IN</option>
                </select>
                <input type="time" bind:value={startInfo[gIndex].time} />
              </div>

              <ul>
                {#each group as p, pIndex}
                  <li
                    class:selected={selected && selected.gIndex === gIndex && selected.pIndex === pIndex}
                    on:click={(e) => { e.stopPropagation(); onPersonClick(gIndex, pIndex); }}
                  >
                    {p.name || `No.${p.no}`}（{p.hdcp}）
                  </li>
                {/each}

                {#if group.length < 4}
                  <li class="empty-slot"
                      on:click={(e) => { e.stopPropagation(); onEmptySlotClick(gIndex); }}>
                    ＋ 追加枠
                  </li>
                {/if}
              </ul>
            </div>
          {/each}
        </div>
      </div>
      <!-- /capture-area -->

      <!-- ★ 画像保存ボタン -->
      <button class="save-btn" on:click={saveImage} disabled={saving}>
        {saving ? "保存中..." : "📷 画像として保存"}
      </button>

      <button class="back-btn" on:click={() => step = 2}>戻る</button>
    </div>
  {/if}
</main>

<style>
  main {
    font-family: "Meiryo", "メイリオ", sans-serif;
    color: #333;
    max-width: 100%;
  }

  h1 {
    margin-bottom: 28px;
    font-size: 26px;
    font-weight: 600;
    text-align: center;
  }

  .card {
    background: #fff;
    border-radius: 14px;
    padding: 28px;
    width: 60%;
    box-shadow: 0 4px 14px rgba(0,0,0,0.08);
    border: 1px solid #e3e6ea;
    margin: 0 auto;
  }

  @media (max-width: 600px) {
    .card { width: 80%; margin: 0 auto; }
  }

  h2 {
    margin-top: 0;
    margin-bottom: 12px;
    font-size: 20px;
    font-weight: 600;
  }

  p {
    margin-top: 0;
    margin-bottom: 16px;
    color: #555;
    font-size: 14px;
  }

  select, input {
    padding: 10px 12px;
    margin: 4px 0;
    width: 100%;
    border-radius: 6px;
    border: 1px solid #c8ccd2;
    font-size: 14px;
    background: #fafafa;
    font-family: "Meiryo", "メイリオ", sans-serif;
  }

  button {
    margin-top: 18px;
    padding: 12px 16px;
    font-size: 15px;
    border-radius: 6px;
    border: none;
    background: #1976d2;
    color: white;
    cursor: pointer;
    width: 100%;
    font-weight: 600;
    letter-spacing: 0.5px;
  }

  button:hover { background: #125ca8; }

  .table { margin-top: 20px; }

  .header, .row {
    display: grid;
    grid-template-columns: 40px 1fr 100px;
    gap: 10px;
    align-items: center;
    font-size: 14px;
  }

  @media (max-width: 600px) {
    .header, .row {
      grid-template-columns: 30px 1fr 70px;
      gap: 6px;
    }
  }

  .header {
    font-weight: 600;
    margin-bottom: 10px;
    padding-bottom: 6px;
    border-bottom: 1px solid #ddd;
  }

  .row { padding: 6px 0; }

  .row input {
    width: 100%;
    box-sizing: border-box;
  }

  /* ★ キャプチャエリア */
  .capture-area {
    background: #f0f4f8;
    border-radius: 10px;
    padding: 16px;
    margin-bottom: 4px;
  }

  .capture-title {
    font-size: 16px;
    font-weight: 700;
    color: #1976d2;
    margin-bottom: 12px;
    text-align: center;
    letter-spacing: 1px;
  }

  .groups {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
    gap: 18px;
  }

  .group {
    background: white;
    border-radius: 10px;
    padding: 16px;
    border: 1px solid #ddd;
    box-shadow: 0 2px 8px rgba(0,0,0,0.05);
  }

  .group h3 {
    margin: 0 0 10px 0;
    font-size: 18px;
    font-weight: 600;
    color: #1976d2;
  }

  .group ul {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .group li {
    padding: 8px 0;
    border-bottom: 1px solid #eee;
    font-size: 17px;
    cursor: pointer; 
  }

  .group li:last-child { border-bottom: none; }

  .buttons {
    display: flex;
    justify-content: space-between;
    margin-top: 20px;
  }

  .buttons button { width: 48%; }

  .add {
    margin-top: 12px;
    padding: 8px 12px;
    background: #4caf50;
    color: white;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    width: 100%;
    font-weight: 600;
  }

  .add:hover { background: #3e8e41; }

  .start-settings {
    display: flex;
    gap: 12px;
    margin: 8px 0 14px 0;
  }

  li.selected {
    background: #d0e8ff;
    border-radius: 6px;
  }

  .empty-slot {
    padding: 8px 0;
    border-radius: 6px;
    text-align: center;
    cursor: pointer;
    margin-top: 6px;
    background-color: #bbb;
    color: #fff;
  }

  .empty-slot:hover { background: #cccbcb; }

  /* ★ 画像保存ボタン */
  .save-btn {
    background: #f57c00;
    margin-top: 16px;
  }

  .save-btn:hover:not(:disabled) { background: #e65100; }

  .save-btn:disabled {
    background: #ccc;
    cursor: not-allowed;
  }


</style>