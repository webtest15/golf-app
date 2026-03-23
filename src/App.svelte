<script>
  // STEP 管理
  let step = 1;

  // 初期人数（STEP1 のセレクト用）
  let count = 6;

  // 参加者データ
  let players = [];

  // 組み合わせ結果
  let groups = [];

  // 時間/inout記入欄
  let startInfo = [];

  // STEP1 → STEP2（初期人数ぶんの行を作る）
  function goToStep2() {
    players = Array.from({ length: count }, (_, i) => ({
      no: i + 1,
      name: "",
      team: "",
      hdcp: 0
    }));
    step = 2;
  }

  // ★ 追加ボタンで人数を増やす
  function addPlayer() {
    players = [
      ...players,
      {
        no: players.length + 1,
        name: "",
        team: "",
        hdcp: 0
      }
    ];
  }

  // STEP2 → STEP3（柔軟な組み合わせ生成）
  function goToStep3() {
    groups = makeBalancedGroups(players);

    // ★ これを必ず入れる！
    startInfo = groups.map(() => ({
      course: "OUT",
      time: ""
    }));

    step = 3;
  }

  // ★ 何人でも対応できる組み合わせアルゴリズム
  function makeBalancedGroups(list) {
    // HDCP の弱い順に並べる
    const sorted = [...list].sort((a, b) => a.hdcp - b.hdcp);

    const perGroup = 4; // 4人組を基本とする
    const groupCount = Math.ceil(sorted.length / perGroup);

    // 空のグループを作成
    const result = Array.from({ length: groupCount }, () => []);

    // 1人ずつ順番にグループへ入れていく（人数が偏らない）
    let index = 0;
    for (let i = 0; i < sorted.length; i++) {
      result[index].push(sorted[i]);
      index = (index + 1) % groupCount;
    }

    return result;
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
          <div>No</div>
          <div>名前</div>
          <div>所属</div>
          <div>HDCP</div>
        </div>

        {#each players as p, i}
          <div class="row">
            <div>{p.no}</div>
            <input placeholder="名前" bind:value={p.name} />
            <input placeholder="所属" bind:value={p.team} />
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

      <div class="groups">
        {#each groups as group, gIndex}
          <div class="group">
            <h3>{gIndex + 1} 組</h3>
                <!-- ★ ここに追加する！ -->
                <div class="start-settings">
                  <select bind:value={startInfo[gIndex].course}>
                    <option value="OUT">OUT</option>
                    <option value="IN">IN</option>
                  </select>

                  <input type="time" bind:value={startInfo[gIndex].time} />
                </div>
                <!-- ★ ここまで -->

            <ul>
              {#each group as p}
                <li>
                  {p.name || `No.${p.no}`}（{p.team} / HDCP {p.hdcp}）
                </li>
              {/each}
            </ul>
          </div>
        {/each}
      </div>

      <button on:click={() => step = 2}>戻る</button>
    </div>
  {/if}
</main>

<style>
  main {
    font-family: "Meiryo", "メイリオ", sans-serif;
    padding: 28px;
    background: #f5f7fa;
    color: #333;

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
    .card {
      width: 100%;
      padding: 20px;
    }
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

  button:hover {
    background: #125ca8;
  }

  .table {
    margin-top: 20px;
  }

  /* ★ はみ出しを完全に防ぐ最適な列幅 */
  .header, .row {
    display: grid;
    grid-template-columns: 40px 1fr 1fr 60px; 
    /* No | 名前 | 所属 | HDCP */
    gap: 10px;
    align-items: center;
    font-size: 14px;
  }

  @media (max-width: 600px) {
  .header, .row {
      grid-template-columns: 30px 1fr 1fr 50px;
      gap: 6px;
    }
  }

  .header {
    font-weight: 600;
    margin-bottom: 10px;
    padding-bottom: 6px;
    border-bottom: 1px solid #ddd;
  }

  .row {
    padding: 6px 0;
  }

  .row input {
    width: 100%;          /* ← これで列幅にフィット */
    box-sizing: border-box; /* ← padding で横幅が増えない */

  }

  .groups {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
    gap: 18px;
    margin-top: 20px;
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
    font-size: 14px;
  }

  .group li:last-child {
    border-bottom: none;
  }

  .buttons {
    display: flex;
    justify-content: space-between;
    margin-top: 20px;
  }

  .buttons button {
    width: 48%;
  }

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

.add:hover {
  background: #3e8e41;
}

.start-settings {
  display: flex;
  gap: 12px;
  margin: 8px 0 14px 0;
}

</style>