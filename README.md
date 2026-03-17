<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>体重トラッカー</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;600&family=Noto+Sans+JP:wght@300;400;500&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #ffffff;
            --surface: #f8f9fb;
            --surface2: #f0f2f5;
            --border: #dde2ea;
            --text: #1a2236;
            --text-dim: #8a96aa;
            --accent: #0077cc;
            --accent2: #005fa3;
            --gain: #d63a3a;
            --loss: #0077cc;
            --safe: #1a7a4a;
            --safe-bg: #f0f7f4;
            --safe-border: #b8d8c8;
            --danger: #c0392b;
            --danger-bg: #fff5f5;
            --danger-border: #f0b8b8;
            --mono: 'JetBrains Mono', monospace;
            --sans: 'Noto Sans JP', sans-serif;
        }
        * { box-sizing: border-box; margin: 0; padding: 0; -webkit-tap-highlight-color: transparent; }
        body { font-family: var(--sans); background: var(--bg); color: var(--text); min-height: 100vh; padding: 20px 16px 60px; }
        .container { max-width: 480px; margin: 0 auto; }

        /* Header */
        .header { display: flex; align-items: center; gap: 10px; margin-bottom: 24px; padding-bottom: 14px; border-bottom: 1px solid var(--border); }
        .header h1 { font-family: var(--mono); font-size: 0.8rem; font-weight: 600; letter-spacing: 0.15em; color: var(--accent); text-transform: uppercase; }
        .header-line { flex: 1; height: 1px; background: linear-gradient(to right, var(--border), transparent); }
        .header-date { font-family: var(--mono); font-size: 0.7rem; color: var(--text-dim); }
        .status-dot { width: 7px; height: 7px; border-radius: 50%; background: var(--accent); flex-shrink: 0; animation: pulse 2s infinite; }
        @keyframes pulse { 0%,100%{opacity:1} 50%{opacity:0.3} }

        /* Metrics */
        .metrics-3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 8px; margin-bottom: 14px; }
        .metric { background: var(--surface); border: 1px solid var(--border); border-radius: 10px; padding: 16px; position: relative; overflow: hidden; }
        .metric::before { content: ''; position: absolute; top: 0; left: 0; right: 0; height: 3px; background: linear-gradient(to right, var(--accent), transparent); }
        .metric-label { font-size: 0.72rem; color: var(--text-dim); margin-bottom: 6px; display: flex; align-items: center; gap: 5px; }
        .metric-val { font-family: var(--mono); font-size: 1.1rem; font-weight: 600; color: var(--accent); line-height: 1; }
        .metric-sub { font-size: 0.68rem; color: var(--text-dim); margin-top: 5px; }
        .metric-val.positive { color: var(--gain); }
        .metric-val.negative { color: var(--loss); }
        .help-icon { display: inline-flex; align-items: center; justify-content: center; width: 16px; height: 16px; border-radius: 50%; background: var(--surface2); border: 1px solid var(--border); font-size: 0.6rem; color: var(--text-dim); cursor: pointer; font-family: var(--mono); font-weight: 600; flex-shrink: 0; }

        /* Equilibrium metric */
        .metric-equil { background: var(--surface); border: 1px solid var(--border); border-radius: 10px; padding: 14px 16px; margin-bottom: 14px; position: relative; overflow: hidden; display: none; }
        .metric-equil::before { content: ''; position: absolute; top: 0; left: 0; right: 0; height: 3px; background: linear-gradient(to right, var(--accent), transparent); }

        /* Card */
        .card { background: var(--surface); border: 1px solid var(--border); border-radius: 12px; padding: 18px; margin-bottom: 14px; }
        .card-title { font-size: 0.72rem; font-weight: 500; color: var(--text-dim); letter-spacing: 0.08em; margin-bottom: 14px; padding-bottom: 10px; border-bottom: 1px solid var(--border); }

        /* Inline explain */
        .inline-explain { display: none; margin-bottom: 14px; background: var(--surface2); border: 1px solid var(--border); border-radius: 10px; padding: 14px; font-size: 0.82rem; line-height: 1.8; color: var(--text-dim); }
        .inline-explain strong { color: var(--text); }

        /* Profile */
        .profile-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
        .profile-field label { font-size: 0.75rem; color: var(--text-dim); display: block; margin-bottom: 6px; }
        .profile-field input[type="number"] { width: 100%; background: var(--surface2); border: 1.5px solid var(--border); color: var(--text); padding: 11px 12px; border-radius: 8px; font-family: var(--mono); font-size: 1rem; font-weight: 600; text-align: center; outline: none; min-height: 48px; -webkit-appearance: none; }
        .profile-field input[type="number"]:focus { border-color: var(--accent); }
        .gender-toggle { display: flex; }
        .gender-btn { flex: 1; min-height: 48px; padding: 10px; background: var(--surface2); border: 1.5px solid var(--border); color: var(--text-dim); font-family: var(--sans); font-size: 0.9rem; cursor: pointer; transition: all 0.15s; text-align: center; }
        .gender-btn:first-child { border-radius: 8px 0 0 8px; border-right: none; }
        .gender-btn:last-child  { border-radius: 0 8px 8px 0; }
        .gender-btn.active { background: rgba(0,119,204,0.08); border-color: var(--accent); color: var(--accent); font-weight: 500; }
        .activity-select { width: 100%; background: var(--surface2); border: 1.5px solid var(--border); color: var(--text); padding: 11px 12px; border-radius: 8px; font-family: var(--sans); font-size: 1rem; outline: none; min-height: 48px; grid-column: 1 / -1; -webkit-appearance: none; appearance: none; }
        .activity-select:focus { border-color: var(--accent); }

        /* Input */
        .field { margin-bottom: 14px; }
        .field-label { font-size: 0.78rem; color: var(--text-dim); margin-bottom: 7px; }
        .quick-btns { display: flex; gap: 8px; margin-bottom: 8px; }
        .quick-btn { flex: 1; min-height: 44px; background: var(--surface2); border: 1.5px solid var(--border); color: var(--text-dim); border-radius: 8px; font-family: var(--mono); font-size: 0.75rem; cursor: pointer; text-align: center; transition: all 0.15s; }
        .quick-btn.active { border-color: var(--accent); color: var(--accent); background: rgba(0,119,204,0.06); font-weight: 600; }
        input[type="date"] { width: 100%; min-height: 48px; background: var(--surface2); border: 1.5px solid var(--border); color: var(--text); padding: 12px 14px; border-radius: 8px; font-family: var(--sans); font-size: 1rem; outline: none; -webkit-appearance: none; }
        input[type="date"]:focus { border-color: var(--accent); }
        .weight-stepper { display: flex; margin-bottom: 6px; }
        .weight-stepper input[type="number"] { flex: 1; min-height: 56px; background: var(--surface2); border: 1.5px solid var(--border); border-left: none; border-right: none; color: var(--text); padding: 12px; font-family: var(--mono); font-size: max(1.4rem, 16px); font-weight: 600; text-align: center; outline: none; }
        .step-btn { width: 60px; min-height: 56px; background: var(--surface2); border: 1.5px solid var(--border); color: var(--text); font-size: 1.6rem; cursor: pointer; display: flex; align-items: center; justify-content: center; user-select: none; }
        .step-btn:first-child { border-radius: 8px 0 0 8px; }
        .step-btn:last-child  { border-radius: 0 8px 8px 0; }
        .step-btn:active { background: var(--border); }
        .last-hint { font-size: 0.75rem; color: var(--text-dim); min-height: 18px; margin-bottom: 12px; }
        .target-row { display: flex; align-items: center; gap: 10px; margin-top: 14px; padding-top: 14px; border-top: 1px solid var(--border); }
        .target-row label { font-size: 0.8rem; color: var(--text-dim); white-space: nowrap; }
        .target-row input[type="number"] { width: 90px; min-height: 44px; background: var(--surface2); border: 1.5px solid var(--border); color: var(--text); padding: 10px; border-radius: 8px; font-family: var(--mono); font-size: 1rem; font-weight: 600; text-align: center; outline: none; }

        /* Buttons */
        .btn { width: 100%; min-height: 52px; padding: 14px; border: none; border-radius: 8px; font-family: var(--sans); font-size: 1rem; font-weight: 500; cursor: pointer; transition: all 0.15s; }
        .btn-primary { background: var(--accent); color: #fff; }
        .btn-primary:active { background: var(--accent2); }
        .btn-secondary { background: transparent; color: var(--text-dim); border: 1.5px solid var(--border); font-size: 0.88rem; margin-top: 8px; min-height: 44px; }
        .btn-danger { background: transparent; color: var(--danger); border: 1.5px solid #f0d0ce; font-size: 0.88rem; margin-top: 8px; min-height: 44px; }

        /* Analysis */
        .analysis-row { display: flex; justify-content: space-between; align-items: baseline; padding: 10px 0; border-bottom: 1px solid var(--border); font-size: 0.88rem; }
        .analysis-row:last-of-type { border-bottom: none; }
        .analysis-key { color: var(--text-dim); font-size: 0.8rem; }
        .analysis-val { font-family: var(--mono); font-weight: 600; color: var(--accent); }
        .analysis-val.warn { color: #d97706; }
        .pace-card { border-radius: 9px; padding: 14px; margin-top: 12px; }
        .pace-card.safe { background: var(--safe-bg); border: 1px solid var(--safe-border); }
        .pace-card.risky { background: var(--danger-bg); border: 1px solid var(--danger-border); }
        .pace-title { font-size: 0.78rem; font-weight: 500; margin-bottom: 9px; }
        .pace-card.safe .pace-title { color: var(--safe); }
        .pace-card.risky .pace-title { color: var(--danger); }
        .pace-row { display: flex; justify-content: space-between; font-size: 0.82rem; margin-bottom: 5px; }
        .pace-row .k { color: var(--text-dim); }
        .pace-card.safe .pace-row .v { color: var(--safe); font-family: var(--mono); font-weight: 600; }
        .pace-card.risky .pace-row .v { color: var(--danger); font-family: var(--mono); font-weight: 600; }
        .pace-note { font-size: 0.75rem; line-height: 1.65; margin-top: 9px; padding-top: 9px; border-top: 1px dashed; }
        .pace-card.safe .pace-note { color: #4a8a6a; border-color: var(--safe-border); }
        .pace-card.risky .pace-note { color: #a06060; border-color: var(--danger-border); }
        .warn-box { background: #fffbeb; border: 1px solid #fcd34d; border-radius: 8px; padding: 12px 14px; font-size: 0.82rem; color: #92400e; margin-top: 10px; line-height: 1.6; }
        .ok-note { font-size: 0.8rem; color: var(--safe); margin-top: 10px; }
        .kcal-box { background: var(--surface2); border: 1px solid var(--border); border-radius: 9px; padding: 13px 14px; margin-top: 10px; }
        .kcal-box-title { font-size: 0.72rem; color: var(--text-dim); margin-bottom: 10px; }
        .kcal-row { display: flex; justify-content: space-between; align-items: baseline; padding: 5px 0; border-bottom: 1px solid var(--border); font-size: 0.82rem; }
        .kcal-row:last-child { border-bottom: none; }
        .kcal-key { color: var(--text-dim); font-size: 0.78rem; }
        .kcal-val { font-family: var(--mono); font-weight: 600; color: var(--accent); }

        /* Collapsible */
        .collapse-btn { width: 100%; display: flex; justify-content: space-between; align-items: center; padding: 11px 14px; background: var(--surface2); border: 1px solid var(--border); border-radius: 8px 8px 0 0; cursor: pointer; font-size: 0.82rem; color: var(--text-dim); margin-top: 10px; }
        .collapse-arrow { transition: transform 0.2s; font-size: 0.7rem; }
        .collapse-arrow.open { transform: rotate(180deg); }
        .collapse-body { display: none; background: var(--surface2); border: 1px solid var(--border); border-top: none; border-radius: 0 0 8px 8px; padding: 14px; font-size: 0.82rem; line-height: 1.85; color: var(--text-dim); }
        .collapse-body.open { display: block; }
        .collapse-body strong { color: var(--text); }

        /* Graph */
        .graph-tabs { display: flex; gap: 6px; margin-bottom: 16px; }
        .graph-tab { flex: 1; padding: 9px 4px; min-height: 40px; background: var(--surface2); border: 1.5px solid var(--border); color: var(--text-dim); border-radius: 8px; font-family: var(--mono); font-size: 0.72rem; cursor: pointer; text-align: center; }
        .graph-tab.active { background: rgba(0,119,204,0.07); border-color: var(--accent); color: var(--accent); font-weight: 600; }
        .graph-panel { display: none; }
        .graph-panel.active { display: block; }
        .graph-note { font-size: 0.73rem; color: var(--text-dim); margin-top: 10px; line-height: 1.7; }

        /* Past data */
        .past-row { display: grid; grid-template-columns: 1fr 90px 56px; gap: 6px; margin-bottom: 8px; align-items: center; }
        .past-date, .past-weight { background: var(--surface2); border: 1.5px solid var(--border); color: var(--text); padding: 9px 10px; border-radius: 8px; font-family: var(--mono); font-size: 0.85rem; outline: none; width: 100%; }
        .past-date:focus, .past-weight:focus { border-color: var(--accent); }
        .past-add-btn { background: var(--accent); color: #fff; border: none; border-radius: 8px; padding: 9px 0; font-size: 0.8rem; cursor: pointer; width: 100%; min-height: 40px; }
        .past-add-btn.done { background: var(--safe); }

        /* Privacy note */
        .privacy-note { display: flex; align-items: flex-start; gap: 10px; background: #f0f7ff; border: 1px solid #bfdbfe; border-radius: 9px; padding: 12px 14px; margin-bottom: 14px; font-size: 0.78rem; line-height: 1.7; color: #1e40af; }
        .privacy-note .icon { font-size: 1rem; flex-shrink: 0; margin-top: 1px; }

        /* Import */
        .import-area { border: 1.5px dashed var(--border); border-radius: 8px; padding: 14px; text-align: center; cursor: pointer; transition: border-color 0.2s; position: relative; margin-top: 8px; }
        .import-area:hover { border-color: var(--accent); }
        .import-area input[type="file"] { position: absolute; top:0; left:0; right:0; bottom:0; opacity: 0; cursor: pointer; width: 100%; height: 100%; }
        .import-label { font-family: var(--mono); font-size: 0.72rem; color: var(--text-dim); }

        /* History */
        .history-list { display: flex; flex-direction: column; gap: 6px; }
        .history-item { display: flex; justify-content: space-between; align-items: center; padding: 11px 13px; background: var(--surface2); border-radius: 8px; font-size: 0.85rem; }
        .history-item .date { color: var(--text-dim); font-size: 0.78rem; }
        .history-item .weight { font-family: var(--mono); font-weight: 600; }
        .delta { font-family: var(--mono); font-size: 0.78rem; }
        .delta.up { color: var(--gain); }
        .delta.down { color: var(--loss); }

        /* Empty */
        .empty { text-align: center; padding: 28px 16px; }
        .empty-title { font-size: 0.95rem; color: var(--text); font-weight: 500; margin-bottom: 8px; }
        .empty-sub { font-size: 0.82rem; color: var(--text-dim); line-height: 1.7; }
    </style>
</head>
<body>
<div class="container">

    <div class="header">
        <span class="status-dot"></span>
        <h1>体重トラッカー</h1>
        <div class="header-line"></div>
        <span class="header-date" id="headerDate"></span>
    </div>

    <div class="metrics-3">
        <div class="metric">
            <div class="metric-label">推定体重</div>
            <div class="metric-val" id="curW">—</div>
            <div class="metric-sub" id="curWsub">データなし</div>
        </div>
        <div class="metric">
            <div class="metric-label">体重トレンド <span class="help-icon" onclick="toggleInline('explainBalance')">?</span></div>
            <div class="metric-val" id="balance">—</div>
            <div class="metric-sub" id="balanceSub">体重変化から推定</div>
        </div>
        <div class="metric">
            <div class="metric-label">摂取目安 <span class="help-icon" onclick="toggleInline('explainTdee')">?</span></div>
            <div class="metric-val" id="tdeeTarget">—</div>
            <div class="metric-sub" id="tdeeTargetSub">プロフィール未設定</div>
        </div>
    </div>

    <div class="metric-equil" id="equilibriumMetric">
        <div class="metric-label">平衡体重（現状維持） <span class="help-icon" onclick="toggleInline('explainEquilib')">?</span></div>
        <div class="metric-val" id="equilibW">—</div>
        <div class="metric-sub" id="equilibWSub">今の生活を続けた場合</div>
    </div>

    <div class="inline-explain" id="explainEquilib">
        <strong>平衡体重とは？</strong><br>
        今の食生活・活動量をそのまま続けた場合、カロリー収支がゼロになる体重の推定値です。<br><br>
        基礎代謝（BMR）は体重に比例するため、体重が下がると消費カロリーも下がり、やがて摂取カロリーと釣り合う点に収束します。プロフィールの設定精度に依存します。
    </div>
    <div class="inline-explain" id="explainBalance">
        <strong>体重トレンドとは？</strong><br>
        今のペースが続いた場合、1ヶ月後に体重がどれだけ変化するかの推定値です。プラスなら増加方向、マイナスなら減少方向を示します。<br><br>
        体重の時系列変化から推定しているため、データが増えるほど精度が上がります。7日分以上になると移動平均で日々のノイズを除去した上で計算します。
    </div>
    <div class="inline-explain" id="explainTdee">
        <strong>摂取目安とは？</strong><br>
        基礎代謝（BMR）× 活動量係数で算出した消費カロリー（TDEE）から、安全ペースの赤字分を引いた値です。プロフィールに性別・年齢・身長・活動量を設定すると表示されます。
    </div>

    <!-- Profile -->
    <div class="card">
        <div class="card-title">プロフィール設定</div>
        <div class="profile-grid">
            <div class="profile-field" style="grid-column: 1 / -1;">
                <label>性別</label>
                <div class="gender-toggle">
                    <button class="gender-btn" id="genderMale" onclick="setGender('male')">男性</button>
                    <button class="gender-btn" id="genderFemale" onclick="setGender('female')">女性</button>
                </div>
            </div>
            <div class="profile-field">
                <label>年齢</label>
                <input type="number" id="profileAge" placeholder="30" min="10" max="100" inputmode="numeric" oninput="saveProfile(); updateUI()">
            </div>
            <div class="profile-field">
                <label>身長（cm）</label>
                <input type="number" id="profileHeight" placeholder="170" min="100" max="220" inputmode="numeric" oninput="saveProfile(); updateUI()">
            </div>
            <div class="profile-field" style="grid-column: 1 / -1;">
                <label>活動量</label>
                <select class="activity-select" id="profileActivity" onchange="saveProfile(); updateUI()">
                    <option value="">選択してください</option>
                    <option value="1.2">ほぼ運動しない（デスクワーク中心）</option>
                    <option value="1.375">軽い運動（週1〜2回）</option>
                    <option value="1.55">適度な運動（週3〜5回）</option>
                    <option value="1.725">活発な運動（週6〜7回）</option>
                    <option value="1.9">非常に活発（肉体労働・アスリート）</option>
                </select>
            </div>
        </div>
    </div>

    <!-- Input -->
    <div class="card">
        <div class="card-title">今日の体重を記録する</div>
        <div class="field">
            <div class="field-label">日付</div>
            <div class="quick-btns" id="quickDateBtns"></div>
            <input type="date" id="inputDate" onchange="syncDateBtns()">
        </div>
        <div class="field">
            <div class="field-label">体重（kg）</div>
            <div class="weight-stepper">
                <button class="step-btn" onclick="stepWeight(-0.1)" ontouchstart="">−</button>
                <input type="number" id="inputWeight" step="0.1" placeholder="00.0" min="10" max="120" inputmode="decimal">
                <button class="step-btn" onclick="stepWeight(+0.1)" ontouchstart="">＋</button>
            </div>
            <div class="last-hint" id="lastWeightHint"></div>
        </div>
        <button class="btn btn-primary" onclick="addData()">記録する</button>
        <div class="target-row">
            <label for="targetW">目標体重</label>
            <input type="number" id="targetW" value="65" step="0.5" min="10" max="120" inputmode="decimal" onchange="updateUI()">
            <span style="font-size:0.85rem; color:var(--text-dim);">kg</span>
        </div>
        <div style="margin-top:16px;">
            <button class="collapse-btn" onclick="toggleCollapse('pastDataBody','pastDataArrow')" style="font-size:0.8rem;">
                <span>過去のデータをまとめて入力する</span>
                <span class="collapse-arrow" id="pastDataArrow">▼</span>
            </button>
            <div class="collapse-body" id="pastDataBody">
                <div style="font-size:0.78rem; color:var(--text-dim); margin-bottom:12px; line-height:1.7;">日付と体重を入力して「追加」を押してください。</div>
                <div id="pastInputRows">
                    <div class="past-row">
                        <input type="date" class="past-date" onchange="checkPastDate(this)">
                        <input type="number" class="past-weight" step="0.1" min="10" max="120" placeholder="kg" inputmode="decimal">
                        <button class="past-add-btn" onclick="addPastRow(this)">追加</button>
                    </div>
                </div>
                <button class="btn btn-secondary" style="margin-top:10px; font-size:0.8rem;" onclick="addPastRowEmpty()">＋ 行を追加</button>
            </div>
        </div>
    </div>

    <!-- Analysis -->
    <div class="card" id="analysisCard">
        <div class="card-title">分析</div>
        <div id="analysisContent">
            <div class="empty">
                <div class="empty-title">体重を記録してください</div>
                <div class="empty-sub">記録すると目標までの日数とカロリー収支の推定が表示されます</div>
            </div>
        </div>
    </div>

    <!-- Chart -->
    <div class="card" id="chartCard" style="display:none;">
        <div class="card-title">グラフ</div>
        <div class="graph-tabs">
            <button class="graph-tab active" onclick="switchGraph('trend', this)">推移・予測</button>
            <button class="graph-tab" onclick="switchGraph('monthly', this)">月別増減</button>
        </div>
        <div class="graph-panel active" id="panelTrend">
            <canvas id="chart"></canvas>
            <div id="forecastNote"></div>
            <div class="graph-note" style="margin-top:6px;">点線が実測値、青がトレンド、緑の破線が予測、オレンジが目標体重です。<br><span style="color:#b45309;">予測は参考値です。データが増えるほど精度が上がります。</span></div>
        </div>
        <div class="graph-panel" id="panelMonthly">
            <canvas id="monthlyChart"></canvas>
            <div class="graph-note">各月の最初と最後の記録の差。青が減少、赤が増加です。</div>
        </div>
    </div>

    <!-- History -->
    <div class="card" id="historyCard" style="display:none;">
        <div class="card-title">最近の記録</div>
        <div id="historyList" class="history-list"></div>
        <div class="privacy-note" style="margin-top:16px;">
            <span class="icon">&#128274;</span>
            <span>このアプリのデータはすべて<strong>この端末の中にのみ保存</strong>されています。外部サーバーへの送信は一切行いません。</span>
        </div>
        <button class="btn btn-secondary" onclick="exportJSON()">JSONでバックアップ</button>
        <button class="btn btn-secondary" onclick="exportCSV()" style="margin-top:6px;">CSVで書き出す</button>
        <button class="btn btn-secondary" onclick="toggleImportPaste()" style="margin-top:6px;">
            CSVを貼り付けて復元する
        </button>
        <div id="importPasteBody" style="display:none; margin-top:8px;">
            <div style="font-size:0.78rem;color:var(--text-dim);margin-bottom:10px;line-height:1.7;">CSVの内容（date,weight形式）をそのまま貼り付けてください。</div>
            <textarea id="importPasteArea" style="width:100%;height:120px;background:var(--surface2);border:1.5px solid var(--border);border-radius:8px;padding:10px;font-family:var(--mono);font-size:0.8rem;color:var(--text);outline:none;resize:vertical;" placeholder="date,weight&#10;2026-01-01,60.0&#10;2026-01-02,59.8"></textarea>
            <button class="btn btn-primary" style="margin-top:8px;min-height:44px;font-size:0.9rem;" onclick="importFromPaste()">読み込む</button>
        </div>
        <button class="btn btn-danger" onclick="clearData()">全データを削除する</button>
    </div>

</div>
<script>
// ============================================================
// 定数
// ============================================================
const STORAGE_KEYS = {
    history:  'weightHistory',
    gender:   'profileGender',
    age:      'profileAge',
    height:   'profileHeight',
    activity: 'profileActivity',
};

const KALMAN_SA     = 0.02;   // プロセスノイズ
const MA_WINDOW     = 7;      // 移動平均ウィンドウ
const FORECAST_DAYS = 90;     // 予測日数

// ============================================================
// State
// ============================================================
let history            = JSON.parse(localStorage.getItem(STORAGE_KEYS.history) || '[]');
let gender             = localStorage.getItem(STORAGE_KEYS.gender) || '';
let chartInstance      = null;
let monthlyChartInstance = null;

// ============================================================
// Kalman Filter
// ============================================================
function runKalman(days, weights, R = 0.25) {
    if (!weights.length) return null;
    const sa = KALMAN_SA;
    let x = [weights[0], 0.0];
    let P = [[10, 0], [0, 1]];
    const smoothed = [];

    for (let i = 0; i < weights.length; i++) {
        let dt = i > 0 ? days[i] - days[i - 1] : 1.0;
        if (dt <= 0) dt = 0.1;

        // Predict
        const xp = [x[0] + dt * x[1], x[1]];
        const Pp = [
            [P[0][0] + dt * (P[1][0] + P[0][1]) + dt*dt * P[1][1] + (dt**4/4)*sa**2,
             P[0][1] + dt * P[1][1] + (dt**3/2)*sa**2],
            [P[1][0] + dt * P[1][1] + (dt**3/2)*sa**2,
             P[1][1] + dt*dt * sa**2]
        ];

        // Update
        const S = Pp[0][0] + R;
        const K = [Pp[0][0] / S, Pp[1][0] / S];
        const y = weights[i] - xp[0];
        x = [xp[0] + K[0]*y, xp[1] + K[1]*y];
        P = [
            [(1 - K[0]) * Pp[0][0], (1 - K[0]) * Pp[0][1]],
            [Pp[1][0] - K[1]*Pp[0][0], Pp[1][1] - K[1]*Pp[0][1]]
        ];
        smoothed.push(x[0]);
    }
    return { curW: x[0], velocity: x[1], smoothed };
}

// ============================================================
// 統計ユーティリティ
// ============================================================
function calcStdDev(weights) {
    if (weights.length < 2) return 0;
    const mean     = weights.reduce((s, v) => s + v, 0) / weights.length;
    const variance = weights.reduce((s, v) => s + (v - mean)**2, 0) / (weights.length - 1);
    return Math.sqrt(variance);
}

function applyMovingAverage(weights, windowSize) {
    if (weights.length < windowSize) return weights;
    return weights.map((_, i) => {
        const start = Math.max(0, i - Math.floor(windowSize / 2));
        const end   = Math.min(weights.length, start + windowSize);
        const slice = weights.slice(start, end);
        return slice.reduce((s, v) => s + v, 0) / slice.length;
    });
}

// ============================================================
// カルマン結果取得（移動平均→標準偏差R自動調整→カルマン）
// ============================================================
function getKalmanResult() {
    if (!history.length) return null;
    const sorted     = [...history].sort((a, b) => new Date(a.date) - new Date(b.date));
    const base       = new Date(sorted[0].date);
    const days       = sorted.map(d => (new Date(d.date) - base) / 86400000);
    const rawWeights = sorted.map(d => parseFloat(d.weight));

    // 標準偏差からR値を自動調整（ばらつきが大きいほどノイズ耐性を強化）
    const stdDev = calcStdDev(rawWeights.slice(-14));
    const R_AUTO = Math.min(4.0, Math.max(0.5, stdDev * 4.0));

    // 7日以上あれば移動平均でノイズ除去してからカルマンへ渡す
    const usingMA   = rawWeights.length >= MA_WINDOW;
    const maWeights = usingMA ? applyMovingAverage(rawWeights, MA_WINDOW) : rawWeights;

    const r = runKalman(days, maWeights, R_AUTO);
    if (!r) return null;

    // smoothedはグラフ表示用に生データで再計算
    r.smoothed = runKalman(days, rawWeights, R_AUTO).smoothed;
    r.usingMA  = usingMA;
    r.stdDev   = stdDev;
    return r;
}

// ============================================================
// プロフィール
// ============================================================
function getProfile() {
    return {
        gender,
        age:      parseInt(document.getElementById('profileAge').value)        || 0,
        height:   parseInt(document.getElementById('profileHeight').value)     || 0,
        activity: parseFloat(document.getElementById('profileActivity').value) || 0,
    };
}

function profileReady() {
    const { gender: g, age, height, activity } = getProfile();
    return !!(g && age && height && activity);
}

function calcBMR(weightKg) {
    const { gender: g, age, height } = getProfile();
    if (!age || !height || !g) return 0;
    return g === 'male'
        ? 10*weightKg + 6.25*height - 5*age + 5
        : 10*weightKg + 6.25*height - 5*age - 161;
}

function calcTDEE(weightKg) {
    const { activity } = getProfile();
    const bmr = calcBMR(weightKg);
    return (!activity || !bmr) ? 0 : Math.round(bmr * activity);
}

// 平衡体重: TDEE = (10w + C) * activity  →  w_eq = (intake/activity - C) / 10
function calcEquilibriumWeight(curW, kcalBalance) {
    if (!profileReady()) return null;
    const { gender: g, age, height, activity } = getProfile();
    const tdee = calcTDEE(curW);
    if (!tdee) return null;
    const C      = g === 'male' ? 6.25*height - 5*age + 5 : 6.25*height - 5*age - 161;
    const intake = tdee + kcalBalance;
    const eq     = (intake / activity - C) / 10;
    return (eq > 20 && eq < 250) ? eq : null;
}

function setGender(g, skipUpdate = false) {
    gender = g;
    if (g) localStorage.setItem(STORAGE_KEYS.gender, g);
    document.getElementById('genderMale').classList.toggle('active', g === 'male');
    document.getElementById('genderFemale').classList.toggle('active', g === 'female');
    if (!skipUpdate) updateUI();
}

function saveProfile() {
    localStorage.setItem(STORAGE_KEYS.age,      document.getElementById('profileAge').value);
    localStorage.setItem(STORAGE_KEYS.height,   document.getElementById('profileHeight').value);
    localStorage.setItem(STORAGE_KEYS.activity, document.getElementById('profileActivity').value);
}

function loadProfile() {
    const age      = localStorage.getItem(STORAGE_KEYS.age);
    const height   = localStorage.getItem(STORAGE_KEYS.height);
    const activity = localStorage.getItem(STORAGE_KEYS.activity);
    if (age)      document.getElementById('profileAge').value      = age;
    if (height)   document.getElementById('profileHeight').value   = height;
    if (activity) document.getElementById('profileActivity').value = activity;
    document.getElementById('genderMale').classList.toggle('active', gender === 'male');
    document.getElementById('genderFemale').classList.toggle('active', gender === 'female');
}

// ============================================================
// Init
// ============================================================
document.getElementById('headerDate').textContent = new Date().toLocaleDateString('ja-JP');
document.getElementById('inputDate').value        = new Date().toISOString().split('T')[0];
loadProfile();
buildQuickDateBtns();
updateLastWeightHint();
updateUI();

// ============================================================
// 日付ボタン
// ============================================================
function buildQuickDateBtns() {
    const c     = document.getElementById('quickDateBtns');
    const today = new Date();
    c.innerHTML = ['今日','昨日','2日前','3日前'].map((label, i) => {
        const d = new Date(today);
        d.setDate(d.getDate() - i);
        const val = d.toISOString().split('T')[0];
        return `<button class="quick-btn" data-date="${val}" onclick="setQuickDate('${val}')">${label}</button>`;
    }).join('');
    syncDateBtns();
}
function setQuickDate(val) {
    document.getElementById('inputDate').value = val;
    syncDateBtns();
}
function syncDateBtns() {
    const cur = document.getElementById('inputDate').value;
    document.querySelectorAll('.quick-btn[data-date]').forEach(b =>
        b.classList.toggle('active', b.dataset.date === cur));
}

// ============================================================
// 体重入力ステッパー
// ============================================================
function stepWeight(delta) {
    const el  = document.getElementById('inputWeight');
    const cur = parseFloat(el.value) || getLastWeight() || 60;
    el.value  = Math.round((cur + delta) * 10) / 10;
}
function getLastWeight() {
    if (!history.length) return null;
    return parseFloat([...history].sort((a, b) => new Date(b.date) - new Date(a.date))[0].weight);
}
function updateLastWeightHint() {
    const el   = document.getElementById('lastWeightHint');
    const last = getLastWeight();
    if (last) {
        const latest = [...history].sort((a, b) => new Date(b.date) - new Date(a.date))[0];
        el.textContent = `前回 ${latest.date}：${last.toFixed(1)} kg`;
        if (!document.getElementById('inputWeight').value)
            document.getElementById('inputWeight').value = last.toFixed(1);
    } else {
        el.textContent = '';
    }
}

// ============================================================
// データ追加・保存
// ============================================================
function saveHistory() {
    localStorage.setItem(STORAGE_KEYS.history, JSON.stringify(history));
}

function addData() {
    const date = document.getElementById('inputDate').value;
    const raw  = document.getElementById('inputWeight').value;
    if (!date || !raw) { alert('日付と体重を入力してください'); return; }
    const w = parseFloat(raw);
    if (w < 10 || w > 120) { alert('体重は10〜120kgの範囲で入力してください'); return; }

    const idx = history.findIndex(d => d.date === date);
    if (idx >= 0) history[idx].weight = w;
    else history.push({ date, weight: w });

    saveHistory();
    document.getElementById('inputWeight').value = '';
    updateLastWeightHint();
    updateUI();
}

// ============================================================
// UI トグル
// ============================================================
function toggleInline(id) {
    const el = document.getElementById(id);
    el.style.display = el.style.display === 'block' ? 'none' : 'block';
}
function toggleCollapse(bodyId, arrowId) {
    const b    = document.getElementById(bodyId);
    const a    = document.getElementById(arrowId);
    const open = b.classList.contains('open');
    b.classList.toggle('open', !open);
    a.classList.toggle('open', !open);
}
function toggleImportPaste() {
    const el = document.getElementById('importPasteBody');
    el.style.display = el.style.display === 'none' ? 'block' : 'none';
}

// ============================================================
// メインUI更新
// ============================================================
function updateUI() {
    if (!history.length) {
        document.getElementById('curW').textContent           = '—';
        document.getElementById('balance').textContent        = '—';
        document.getElementById('tdeeTarget').textContent     = '—';
        document.getElementById('tdeeTargetSub').textContent  = 'データなし';
        document.getElementById('equilibriumMetric').style.display = 'none';
        document.getElementById('analysisContent').innerHTML  = `
            <div class="empty">
                <div class="empty-title">体重を記録してください</div>
                <div class="empty-sub">記録すると目標までの日数とトレンドが表示されます</div>
            </div>`;
        document.getElementById('chartCard').style.display    = 'none';
        document.getElementById('historyCard').style.display  = 'none';
        return;
    }

    const sorted   = [...history].sort((a, b) => new Date(a.date) - new Date(b.date));
    const weights  = sorted.map(d => parseFloat(d.weight));
    const r        = getKalmanResult();
    if (!r) return;

    const targetW  = parseFloat(document.getElementById('targetW').value) || 65;
    const diff     = r.curW - targetW;
    const kcal     = r.velocity * 7000;
    const kgPerMonth = r.velocity * 30;
    const n        = history.length;
    const tdee     = calcTDEE(r.curW);
    const safeDeficit  = Math.round(r.curW * 0.001 * 7000);
    const intakeTarget = tdee > 0 ? (diff > 0 ? tdee - safeDeficit : tdee) : 0;

    // 推定体重
    document.getElementById('curW').textContent    = r.curW.toFixed(2) + ' kg';
    document.getElementById('curWsub').textContent = diff > 0 ? `目標まで ${diff.toFixed(1)} kg` : '目標達成済み';

    // 体重トレンド
    const balEl   = document.getElementById('balance');
    balEl.textContent = (kgPerMonth >= 0 ? '+' : '') + kgPerMonth.toFixed(1) + ' kg/月';
    balEl.className   = 'metric-val ' + (kgPerMonth > 0.3 ? 'positive' : kgPerMonth < -0.3 ? 'negative' : '');
    const noiseLabel  = r.stdDev > 1.0 ? '・変動大' : r.stdDev > 0.5 ? '・変動中' : '';
    document.getElementById('balanceSub').textContent = r.usingMA
        ? (n < 7  ? `参考値（${n}日分・精度低${noiseLabel}）` :
           n < 14 ? `目安値（${n}日分・精度中${noiseLabel}）` :
                    `推定値（${n}日分・精度高${noiseLabel}）`)
        : `参考値（${n}日分・7日未満）`;

    // 摂取目安
    if (tdee > 0) {
        document.getElementById('tdeeTarget').textContent    = intakeTarget.toLocaleString() + ' kcal';
        document.getElementById('tdeeTargetSub').textContent = diff > 0 ? '目標向け摂取目安' : '維持カロリー';
    } else {
        document.getElementById('tdeeTarget').textContent    = '—';
        document.getElementById('tdeeTargetSub').textContent = 'プロフィールを入力';
    }

    // 平衡体重
    const eqEl = document.getElementById('equilibriumMetric');
    if (profileReady() && n >= 5) {
        const eq = calcEquilibriumWeight(r.curW, kcal);
        eqEl.style.display = '';
        if (eq !== null) {
            const eqDiff = eq - r.curW;
            document.getElementById('equilibW').textContent    = eq.toFixed(1) + ' kg';
            document.getElementById('equilibWSub').textContent =
                Math.abs(eqDiff) < 0.5 ? 'ほぼ現状維持' :
                eqDiff > 0 ? `現在より +${eqDiff.toFixed(1)} kg で収束` :
                             `現在より ${eqDiff.toFixed(1)} kg で収束`;
        } else {
            document.getElementById('equilibW').textContent    = '計測中';
            document.getElementById('equilibWSub').textContent = 'データ蓄積中';
        }
    } else {
        eqEl.style.display = 'none';
    }

    renderAnalysis({ r, diff, kcal, kgPerMonth, targetW, tdee, intakeTarget, n });
    renderHistory(sorted);
    renderChart({ weights, smoothed: r.smoothed, targetW, velocity: r.velocity, sorted, n });
    renderMonthlyChart(sorted);
    document.getElementById('chartCard').style.display   = '';
    document.getElementById('historyCard').style.display = '';
}

// ============================================================
// 分析パネル
// ============================================================
function renderAnalysis({ r, diff, kcal, kgPerMonth, targetW, tdee, intakeTarget, n }) {
    const el = document.getElementById('analysisContent');
    if (n < 2) {
        el.innerHTML = `<div class="empty"><div class="empty-title">もう1日記録すると分析が始まります</div><div class="empty-sub">毎日続けるほど推定の精度が上がります</div></div>`;
        return;
    }

    let html = '';
    const bmr = Math.round(calcBMR(r.curW));

    // 精度警告
    if (n < 5) {
        html += `<div class="warn-box"><strong style="display:block;margin-bottom:4px;">数値が極端に見える場合があります</strong>現在 ${n} 日分のデータのため、日々のばらつきをトレンドと誤って拾うことがあります。7日以上続けると現実的な値に近づきます。</div>`;
    } else if (n < 14) {
        html += `<div style="background:#f0f7ff;border:1px solid #bfdbfe;border-radius:9px;padding:11px 14px;margin-bottom:14px;font-size:0.8rem;line-height:1.7;color:#1e40af;">${n} 日分のデータをもとに推定しています。<strong>14日以上になるとさらに精度が上がります。</strong></div>`;
    }

    if (diff > 0) {
        // 減量中
        const safeKg    = r.curW * 0.001;
        const safeDays  = Math.ceil(diff / safeKg);
        const safeDate  = new Date(); safeDate.setDate(safeDate.getDate() + safeDays);
        const safeKcal  = Math.round(safeKg * 7000);
        const riskyKg   = r.curW * 0.005;
        const riskyDays = Math.ceil(diff / riskyKg);
        const riskyDate = new Date(); riskyDate.setDate(riskyDate.getDate() + riskyDays);
        const riskyKcal = Math.round(riskyKg * 7000);
        const shortfall = safeKcal - Math.round(-kcal);

        html += `
            <div class="analysis-row"><span class="analysis-key">目標まで</span><span class="analysis-val">${diff.toFixed(1)} kg</span></div>
            <div class="analysis-row"><span class="analysis-key">推定収支</span><span class="analysis-val ${kgPerMonth > 0.3 ? 'warn' : ''}">${kcal >= 0 ? '+' : ''}${Math.round(kcal)} kcal/日</span></div>
            <div class="pace-card safe">
                <div class="pace-title">無理なく続けられるペース</div>
                <div class="pace-row"><span class="k">目標到達の目安</span><span class="v">${safeDate.toLocaleDateString('ja-JP')}（${safeDays}日後）</span></div>
                <div class="pace-row"><span class="k">1日あたりの必要赤字</span><span class="v">${safeKcal} kcal</span></div>
                <div class="pace-note">体重の0.1%/日以内のペース。筋肉を保ちながら体脂肪を落とせる、生理学的に持続可能な速さの上限の目安です。</div>
            </div>
            <div class="pace-card risky">
                <div class="pace-title">身体に負担がかかるペース（参考）</div>
                <div class="pace-row"><span class="k">目標到達の目安</span><span class="v">${riskyDate.toLocaleDateString('ja-JP')}（${riskyDays}日後）</span></div>
                <div class="pace-row"><span class="k">1日あたりの必要赤字</span><span class="v">${riskyKcal} kcal</span></div>
                <div class="pace-note">体重の0.5%/日超のペース。筋肉の分解・基礎代謝の低下・リバウンドリスクが高まるため推奨しません。</div>
            </div>`;
        html += shortfall > 0
            ? `<div class="warn-box">現在のペースから、あと 1日 ${shortfall} kcal 減らすと安全ペースに届きます。</div>`
            : `<div class="ok-note">現在のペースは安全ペースの範囲内です。</div>`;
    } else {
        // 目標達成済み
        html += `
            <div class="analysis-row"><span class="analysis-key">目標体重</span><span class="analysis-val">${targetW} kg</span></div>
            <div class="analysis-row"><span class="analysis-key">現在の状況</span><span class="analysis-val">目標達成済み</span></div>
            <div class="analysis-row"><span class="analysis-key">推定収支</span><span class="analysis-val ${kgPerMonth > 0.3 ? 'warn' : ''}">${kcal >= 0 ? '+' : ''}${Math.round(kcal)} kcal/日</span></div>`;
        if (kcal > 100) html += `<div class="warn-box">増加傾向があります。目標体重を維持するため収支を意識してみてください。</div>`;
    }

    // カロリー内訳
    if (tdee > 0 && bmr > 0) {
        html += `
        <div class="kcal-box">
            <div class="kcal-box-title">カロリー内訳（プロフィールから計算）</div>
            <div class="kcal-row"><span class="kcal-key">基礎代謝（BMR）</span><span class="kcal-val">${bmr.toLocaleString()} kcal/日</span></div>
            <div class="kcal-row"><span class="kcal-key">活動込みの消費量（TDEE）</span><span class="kcal-val">${tdee.toLocaleString()} kcal/日</span></div>
            <div class="kcal-row"><span class="kcal-key">目標向け摂取目安</span><span class="kcal-val">${intakeTarget.toLocaleString()} kcal/日</span></div>
        </div>`;
    }

    // 折りたたみ説明
    const kgPerMonthAbs = Math.abs(kgPerMonth).toFixed(1);
    const direction     = kgPerMonth > 0.3 ? '増加' : kgPerMonth < -0.3 ? '減少' : '維持';
    html += `
    <button class="collapse-btn" onclick="toggleCollapse('bodyTrend','arrowTrend')">
        <span>トレンドの読み方</span><span class="collapse-arrow" id="arrowTrend">▼</span>
    </button>
    <div class="collapse-body" id="bodyTrend">
        現在のトレンドから、<strong>このペースが続くと月間約 ${kgPerMonthAbs} kg の${direction}傾向</strong>が推定されています。<br><br>
        体重の変化から逆算した推定値です。7日分以上のデータがあると移動平均でノイズを除去して精度が上がります。
    </div>
    <button class="collapse-btn" onclick="toggleCollapse('bodyKalman','arrowKalman')">
        <span>なぜ「カルマンフィルター」を使うのか</span><span class="collapse-arrow" id="arrowKalman">▼</span>
    </button>
    <div class="collapse-body" id="bodyKalman">
        体重は食事・水分・排泄などで毎日1〜2 kg程度ばらつきます。カルマンフィルターは<strong>「今の体重」と「変化の速さ」を同時に推定</strong>する手法で、測定間隔が不規則でも補正し、最新データほど重く扱います。もともと宇宙船の軌道推定に使われた手法です。
    </div>
    <button class="collapse-btn" onclick="toggleCollapse('bodyDisclaimer','arrowDisclaimer')" style="margin-top:6px;border-color:#f0d0ce;color:#c0392b;">
        <span>このアプリの測定限界について</span><span class="collapse-arrow" id="arrowDisclaimer">▼</span>
    </button>
    <div class="collapse-body" id="bodyDisclaimer" style="border-color:#f0d0ce;">
        <strong style="color:#c0392b;display:block;margin-bottom:8px;">このアプリが計測・推定できないもの</strong>
        体脂肪率・筋肉量・骨量・水分量は区別できません。カロリー収支は体重変化から逆算した参考値です。<strong style="color:#c0392b;">このアプリは医療機器ではありません。</strong>健康上の懸念がある場合は医師または管理栄養士にご相談ください。
    </div>`;

    el.innerHTML = html;
}

// ============================================================
// 履歴パネル
// ============================================================
function renderHistory(sorted) {
    const el = document.getElementById('historyList');
    el.innerHTML = [...sorted].reverse().slice(0, 5).map(d => {
        const idx    = sorted.findIndex(h => h.date === d.date);
        const prev   = idx > 0 ? parseFloat(sorted[idx - 1].weight) : null;
        const curr   = parseFloat(d.weight);
        const delta  = prev !== null ? curr - prev : null;
        const dHtml  = delta !== null
            ? `<span class="delta ${delta > 0 ? 'up' : 'down'}">${delta > 0 ? '+' : ''}${delta.toFixed(1)} kg</span>`
            : '';
        return `<div class="history-item"><span class="date">${d.date}</span><span class="weight">${curr.toFixed(1)} kg</span>${dHtml}</div>`;
    }).join('');
}

// ============================================================
// 予測モデル（非線形）
// ============================================================
function buildForecast(startWeight, velocity, targetW) {
    const forecast       = [];
    let w                = startWeight;
    const initialGap     = Math.abs(startWeight - targetW);
    const targetDirection = targetW < startWeight ? -1 : 1;
    const isTowardTarget  = (velocity * targetDirection) > 0;

    if (initialGap < 0.01) {
        return Array(FORECAST_DAYS + 1).fill(parseFloat(startWeight.toFixed(2)));
    }
    for (let i = 0; i <= FORECAST_DAYS; i++) {
        forecast.push(parseFloat(w.toFixed(2)));
        if (isTowardTarget) {
            const gapRatio = Math.min(Math.abs(w - targetW) / initialGap, 1.0);
            const dv       = velocity * Math.pow(gapRatio, 0.6);
            const nextW    = w + dv;
            w = targetDirection < 0 ? Math.max(nextW, targetW) : Math.min(nextW, targetW);
        } else {
            const dv = velocity * Math.exp(-Math.abs(w - startWeight) * 0.15);
            if (Math.abs(dv) < 0.0001) { for (let j = i+1; j <= FORECAST_DAYS; j++) forecast.push(parseFloat(w.toFixed(2))); break; }
            w += dv;
        }
    }
    return forecast;
}

// ============================================================
// グラフ：推移 + 予測
// ============================================================
function renderChart({ weights, smoothed, targetW, velocity, sorted, n }) {
    const ctx = document.getElementById('chart').getContext('2d');
    if (chartInstance) chartInstance.destroy();

    const lastDate     = new Date(sorted[sorted.length - 1].date);
    const lastSmoothed = smoothed[smoothed.length - 1];
    const useNonlinear = n >= 14;

    const forecastValues = useNonlinear
        ? buildForecast(lastSmoothed, velocity, targetW)
        : Array.from({ length: FORECAST_DAYS + 1 }, (_, i) => parseFloat((lastSmoothed + velocity*i).toFixed(2)));

    const labels = sorted.map(d => d.date.slice(5));
    for (let i = 1; i <= FORECAST_DAYS; i++) {
        const d = new Date(lastDate);
        d.setDate(d.getDate() + i);
        labels.push(`${String(d.getMonth()+1).padStart(2,'0')}/${String(d.getDate()).padStart(2,'0')}`);
    }

    const forecastData = new Array(weights.length).fill(null);
    forecastData[weights.length - 1] = lastSmoothed;
    forecastValues.slice(1).forEach(v => forecastData.push(v));

    const forecastColor = useNonlinear ? 'rgba(22,163,74,0.75)' : 'rgba(180,100,0,0.7)';
    const pointR        = n > 60 ? 0 : n > 30 ? 1.5 : 3;
    const lineW         = n > 60 ? 0.5 : 1;

    chartInstance = new Chart(ctx, {
        type: 'line',
        data: { labels, datasets: [
            { label: '実測値',   data: [...weights, ...new Array(FORECAST_DAYS).fill(null)], borderColor: 'rgba(0,0,0,0.12)', backgroundColor: 'transparent', borderDash: [3,3], pointRadius: pointR, pointBackgroundColor: 'rgba(0,0,0,0.2)', tension: 0, borderWidth: lineW },
            { label: 'トレンド', data: [...smoothed, ...new Array(FORECAST_DAYS).fill(null)], borderColor: '#0077cc', backgroundColor: 'rgba(0,119,204,0.06)', fill: true, tension: 0.4, pointRadius: 0, borderWidth: 2 },
            { label: useNonlinear ? '予測（平衡考慮）' : '短期トレンド（参考）', data: forecastData, borderColor: forecastColor, backgroundColor: useNonlinear ? 'rgba(22,163,74,0.04)' : 'rgba(180,100,0,0.03)', borderDash: [5,4], pointRadius: 0, tension: useNonlinear ? 0.4 : 0, borderWidth: 1.5, fill: false },
            { label: '目標', data: new Array(weights.length + FORECAST_DAYS).fill(targetW), borderColor: 'rgba(200,140,0,0.55)', borderDash: [6,4], pointRadius: 0, tension: 0, borderWidth: 1.5, fill: false }
        ]},
        options: {
            responsive: true, interaction: { mode: 'index', intersect: false },
            plugins: {
                legend: { labels: { color:'#8a96aa', font:{family:'JetBrains Mono',size:11}, boxWidth:12, padding:12 } },
                tooltip: { backgroundColor:'#fff', borderColor:'#dde2ea', borderWidth:1, titleColor:'#1a2236', bodyColor:'#8a96aa', titleFont:{family:'JetBrains Mono'}, bodyFont:{family:'JetBrains Mono',size:12},
                    callbacks: { afterBody: (items) => items[0]?.dataIndex >= weights.length ? ['', useNonlinear ? '▲ 予測値（参考）' : '▲ 短期トレンド延長（参考）'] : [] } }
            },
            scales: {
                x: { ticks:{ color:'#8a96aa', font:{family:'JetBrains Mono',size:10}, maxTicksLimit:8 }, grid:{color:'#eaecf0'} },
                y: { ticks:{ color:'#8a96aa', font:{family:'JetBrains Mono',size:11}, callback: v => v.toFixed(1)+' kg' }, grid:{color:'#eaecf0'} }
            }
        }
    });

    const noteEl = document.getElementById('forecastNote');
    if (noteEl) {
        const modeNote = useNonlinear
            ? 'データ14日以上：目標を平衡点とした生理学的モデルで予測。'
            : `データ${n}日分：現在のトレンドを線形延長した短期予測です。14日を超えると非線形予測に切り替わります。`;
        noteEl.innerHTML = `
            <div style="display:flex;flex-wrap:wrap;gap:8px;margin-top:10px;">
                ${[30,60,90].map(d => {
                    const val  = forecastValues[Math.min(d, forecastValues.length-1)];
                    const diff = val - lastSmoothed;
                    const col  = diff < 0 ? 'var(--loss)' : diff > 0 ? 'var(--gain)' : 'var(--text-dim)';
                    return `<div style="flex:1;min-width:80px;padding:9px 10px;background:var(--surface2);border-radius:8px;text-align:center;">
                        <div style="font-family:var(--mono);font-size:0.6rem;color:var(--text-dim);margin-bottom:4px;">${d}日後</div>
                        <div style="font-family:var(--mono);font-size:0.9rem;font-weight:600;color:var(--text);">${val.toFixed(1)} kg</div>
                        <div style="font-family:var(--mono);font-size:0.72rem;color:${col};">${diff >= 0 ? '+' : ''}${diff.toFixed(1)} kg</div>
                    </div>`;
                }).join('')}
            </div>
            <div style="font-size:0.72rem;color:var(--text-dim);margin-top:8px;line-height:1.7;">${modeNote}</div>`;
    }
}

// ============================================================
// グラフ：月別増減
// ============================================================
function renderMonthlyChart(sorted) {
    const ctx = document.getElementById('monthlyChart').getContext('2d');
    if (monthlyChartInstance) monthlyChartInstance.destroy();

    const byMonth = {};
    sorted.forEach(d => {
        const key = d.date.slice(0, 7);
        if (!byMonth[key]) byMonth[key] = { first: d.weight, last: d.weight };
        else byMonth[key].last = d.weight;
    });
    const months = Object.keys(byMonth).sort();
    if (!months.length) return;

    const labels       = months.map(m => { const [y, mo] = m.split('-'); return `${y}年${parseInt(mo)}月`; });
    const deltas       = months.map(m => parseFloat((byMonth[m].last - byMonth[m].first).toFixed(2)));
    const colors       = deltas.map(d => d > 0 ? 'rgba(214,58,58,0.7)' : 'rgba(0,119,204,0.7)');
    const borderColors = deltas.map(d => d > 0 ? '#d63a3a' : '#0077cc');

    monthlyChartInstance = new Chart(ctx, {
        type: 'bar',
        data: { labels, datasets: [{ label: '月間増減（kg）', data: deltas, backgroundColor: colors, borderColor: borderColors, borderWidth: 1.5, borderRadius: 5 }] },
        options: {
            responsive: true,
            plugins: { legend: { display: false },
                tooltip: { backgroundColor:'#fff', borderColor:'#dde2ea', borderWidth:1, titleColor:'#1a2236', bodyColor:'#8a96aa', titleFont:{family:'JetBrains Mono'}, bodyFont:{family:'JetBrains Mono',size:12},
                    callbacks: { label: (item) => ` ${item.raw > 0 ? '+' : ''}${item.raw} kg` } } },
            scales: {
                x: { ticks:{ color:'#8a96aa', font:{family:'JetBrains Mono',size:11} }, grid:{display:false} },
                y: { ticks:{ color:'#8a96aa', font:{family:'JetBrains Mono',size:11}, callback: v => (v>0?'+':'')+v+' kg' }, grid:{color:'#eaecf0'} }
            }
        }
    });
}

// ============================================================
// グラフタブ切替
// ============================================================
function switchGraph(tab, el) {
    document.querySelectorAll('.graph-tab').forEach(t => t.classList.remove('active'));
    document.querySelectorAll('.graph-panel').forEach(p => p.classList.remove('active'));
    el.classList.add('active');
    document.getElementById(tab === 'trend' ? 'panelTrend' : 'panelMonthly').classList.add('active');

    const sorted  = [...history].sort((a, b) => new Date(a.date) - new Date(b.date));
    const n       = history.length;
    if (tab === 'monthly') {
        renderMonthlyChart(sorted);
    } else {
        const weights = sorted.map(d => parseFloat(d.weight));
        const r       = getKalmanResult();
        if (r) renderChart({ weights, smoothed: r.smoothed, targetW: parseFloat(document.getElementById('targetW').value) || 65, velocity: r.velocity, sorted, n });
    }
}

// ============================================================
// 過去データ入力
// ============================================================
function checkPastDate(dateInput) {
    const row         = dateInput.closest('.past-row');
    const dateVal     = dateInput.value;
    const weightInput = row.querySelector('.past-weight');
    const btn         = row.querySelector('.past-add-btn');
    if (!dateVal) return;
    const existing = history.find(h => h.date === dateVal);
    if (existing) {
        weightInput.value = existing.weight; weightInput.disabled = true;
        dateInput.disabled = true; btn.textContent = '登録済み';
        btn.classList.add('done'); btn.disabled = true; row.style.opacity = '0.6';
    } else {
        weightInput.disabled = false; btn.textContent = '追加';
        btn.classList.remove('done'); btn.disabled = false;
        row.style.opacity = '1'; weightInput.focus();
    }
}

function addPastRow(btn) {
    const row       = btn.closest('.past-row');
    const dateVal   = row.querySelector('.past-date').value;
    const weightVal = parseFloat(row.querySelector('.past-weight').value);
    if (!dateVal) { alert('日付を入力してください'); return; }
    if (isNaN(weightVal) || weightVal < 10 || weightVal > 120) { alert('体重を正しく入力してください（10〜120kg）'); return; }

    const idx = history.findIndex(h => h.date === dateVal);
    if (idx >= 0) history[idx].weight = weightVal;
    else history.push({ date: dateVal, weight: weightVal });

    saveHistory();
    btn.textContent = '✓'; btn.classList.add('done'); btn.disabled = true;
    row.querySelector('.past-date').disabled   = true;
    row.querySelector('.past-weight').disabled = true;

    const wasOpen = document.getElementById('pastDataBody').classList.contains('open');
    updateUI();
    if (wasOpen) {
        document.getElementById('pastDataBody').classList.add('open');
        document.getElementById('pastDataArrow').classList.add('open');
    }
}

function addPastRowEmpty() {
    const container = document.getElementById('pastInputRows');
    if (container.querySelectorAll('.past-row').length >= 30) { alert('一度に入力できるのは30行までです'); return; }
    const div = document.createElement('div');
    div.className = 'past-row';
    div.innerHTML = `<input type="date" class="past-date" onchange="checkPastDate(this)"><input type="number" class="past-weight" step="0.1" min="10" max="120" placeholder="kg" inputmode="decimal"><button class="past-add-btn" onclick="addPastRow(this)">追加</button>`;
    container.appendChild(div);
    div.querySelector('.past-date').focus();
}

// ============================================================
// CSV解析ユーティリティ（インポート共通）
// ============================================================
function parseCSVText(text) {
    return text.trim().split('\n')
        .filter(l => l.trim() && !l.startsWith('date'))
        .map(l => { const p = l.split(','); return { date: p[0].trim(), weight: parseFloat(p[1]) }; })
        .filter(d => d.date && !isNaN(d.weight));
}

function mergeIncoming(incoming) {
    incoming.forEach(d => {
        const idx = history.findIndex(h => h.date === d.date);
        if (idx >= 0) history[idx].weight = d.weight;
        else history.push({ date: d.date, weight: d.weight });
    });
    saveHistory();
    updateLastWeightHint();
    updateUI();
}

// ============================================================
// エクスポート
// ============================================================
function triggerDownload(blob, filename) {
    const url = URL.createObjectURL(blob);
    // iOS Safari は a[download] が動かないので新しいタブで開く
    const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent) && !window.MSStream;
    if (isIOS) {
        window.open(url, '_blank');
        setTimeout(() => URL.revokeObjectURL(url), 3000);
    } else {
        const link = document.createElement('a');
        link.href = url; link.download = filename; link.style.display = 'none';
        document.body.appendChild(link);
        link.dispatchEvent(new MouseEvent('click', { bubbles: true, cancelable: true, view: window }));
        document.body.removeChild(link);
        setTimeout(() => URL.revokeObjectURL(url), 1000);
    }
}
function exportJSON() {
    triggerDownload(
        new Blob([JSON.stringify({ version: 1, exportedAt: new Date().toISOString(), data: history }, null, 2)], { type: 'application/json' }),
        'weight_backup_' + new Date().toISOString().slice(0,10) + '.json'
    );
}
function exportCSV() {
    triggerDownload(
        new Blob(["date,weight\n" + history.map(d => d.date + ',' + d.weight).join("\n")], { type: 'text/csv' }),
        'weight_data_' + new Date().toISOString().slice(0,10) + '.csv'
    );
}

// ============================================================
// インポート（貼り付け）
// ============================================================
function importFromPaste() {
    const text = document.getElementById('importPasteArea').value.trim();
    if (!text) { alert('データを貼り付けてください'); return; }
    try {
        const incoming = parseCSVText(text);
        if (!incoming.length) throw new Error('有効なデータが見つかりませんでした');
        mergeIncoming(incoming);
        document.getElementById('importPasteArea').value  = '';
        document.getElementById('importPasteBody').style.display = 'none';
        alert(incoming.length + ' 件のデータを読み込みました。');
    } catch (err) { alert('読み込みに失敗しました：' + err.message); }
}

// ============================================================
// 全データ削除
// ============================================================
function clearData() {
    const overlay = document.createElement('div');
    overlay.style.cssText = 'position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.5);z-index:9999;display:flex;align-items:center;justify-content:center;';
    overlay.innerHTML = `
        <div style="background:#fff;border-radius:12px;padding:24px;margin:20px;max-width:320px;text-align:center;">
            <div style="font-weight:600;font-size:1rem;margin-bottom:12px;color:#1a2236;">全データを削除しますか？</div>
            <div style="font-size:0.82rem;color:#8a96aa;margin-bottom:20px;">この操作は元に戻せません。<br>削除前にJSONバックアップを推奨します。</div>
            <div style="display:flex;gap:10px;">
                <button id="cancelDel" style="flex:1;padding:10px;border:1px solid #dde2ea;border-radius:8px;background:#f8f9fb;cursor:pointer;font-size:0.9rem;">キャンセル</button>
                <button id="confirmDel" style="flex:1;padding:10px;border:none;border-radius:8px;background:#d63a3a;color:#fff;cursor:pointer;font-size:0.9rem;font-weight:600;">削除する</button>
            </div>
        </div>`;
    document.body.appendChild(overlay);
    document.getElementById('cancelDel').onclick = () => document.body.removeChild(overlay);
    document.getElementById('confirmDel').onclick = () => {
        document.body.removeChild(overlay);
        localStorage.clear();
        history = [];
        gender  = '';
        document.getElementById('profileAge').value      = '';
        document.getElementById('profileHeight').value   = '';
        document.getElementById('profileActivity').value = '';
        setGender('', true);
        updateLastWeightHint();
        updateUI();
    };
}
</script>
</body>
</html>
