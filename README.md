<html lang="zh-Hant" class="h-full">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>零碳防線：2050 (Net Zero Frontier)</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    /* 自訂滾動條樣式 */
    .custom-scrollbar::-webkit-scrollbar {
      width: 4px;
    }
    .custom-scrollbar::-webkit-scrollbar-track {
      background: transparent;
    }
    .custom-scrollbar::-webkit-scrollbar-thumb {
      background: #334155;
      border-radius: 4px;
    }

    /* 實作四周 1 公分物理邊界 */
    .outer-container {
      padding: 1cm;
      box-sizing: border-box;
      background-color: #020617; /* bg-slate-950 */
    }

    /* 響應式微調：避免在極小螢幕上因邊界擠壓內容 */
    @media (max-width: 640px) {
      .outer-container {
        padding: 0.5cm;
      }
    }
    @media (max-width: 380px) {
      .outer-container {
        padding: 0.2cm;
      }
    }
  </style>
</head>
<body class="h-full text-slate-100 font-sans select-none overflow-hidden bg-slate-950">

  <!-- 1公分邊界外包裝容器 -->
  <div class="outer-container h-full w-full flex items-center justify-center">
    <!-- 遊戲主體視窗 -->
    <div id="app" class="h-full w-full bg-slate-900 rounded-2xl border border-slate-800 shadow-2xl shadow-emerald-950/20 overflow-hidden flex flex-col relative"></div>
  </div>

  <script>
    // --- 遊戲常數與設定 ---
    const MAX_TURNS = 30;
    const TEMP_CRITICAL = 2.0;
    const TEMP_START = 1.15;
    const BUDGET_START = 100;
    const OPINION_START = 80;
    const MAP_SIZE = 25;

    // SVG 圖示資源
    const ICONS = {
      globe: `<svg class="w-16 h-16 text-emerald-500 mx-auto" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"></circle><path d="M2 12h20M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"></path></svg>`,
      thermometer: `<svg class="w-5 h-5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path d="M14 14.76V3.5a2.5 2.5 0 0 0-5 0v11.26a4.5 4.5 0 1 0 5 0z"></path></svg>`,
      cloudRain: `<svg class="w-5 h-5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path d="M20 17.58A5 5 0 0 0 18 8h-1.26A8 8 0 1 0 4 16.25M8 16v6M12 16v6M16 16v6"></path></svg>`,
      dollar: `<svg class="w-4 h-4" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><line x1="12" y1="1" x2="12" y2="23"></line><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"></path></svg>`,
      zap: `<svg class="w-4 h-4 text-yellow-400" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><polyline points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"></polyline></svg>`,
      users: `<svg class="w-4 h-4 text-blue-400" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path><circle cx="9" cy="7" r="4"></circle><path d="M23 21v-2a4 4 0 0 0-3-3.87"></path><path d="M16 3.13a4 4 0 0 1 0 7.75"></path></svg>`,
      factory: `<svg class="w-4 h-4 text-slate-400" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path d="M2 20h20M20 10v10M14 4v16M8 12v8M4 16v4"></path></svg>`,
      wind: `<svg class="w-4 h-4 text-sky-300" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path d="M9.59 4.59A2 2 0 1 1 11 8H2m10.59 11.41A2 2 0 1 0 14 16H2m15.73-8.27A2.5 2.5 0 1 1 19.5 12H2"></path></svg>`,
      tree: `<svg class="w-4 h-4 text-green-300" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path d="M12 2L2 22h20L12 2z"></path></svg>`,
      alert: `<svg class="w-4 h-4 mr-1.5 text-amber-200" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0zM12 9v4M12 17h.01"></path></svg>`,
      skull: `<svg class="w-16 h-16 mx-auto text-red-400 mb-3" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path d="M9 10h.01M15 10h.01M12 2a8 8 0 0 0-8 8v4a4 4 0 0 0 4 4h8a4 4 0 0 0 4-4v-4a8 8 0 0 0-8-8zM10 18h4m-3-2h2"></path></svg>`,
      leaf: `<svg class="w-16 h-16 mx-auto text-emerald-400 mb-3" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path d="M11 20A7 7 0 0 1 9.8 6.1C15.5 5 17 4.48 19 2c1 2 2 3.5 1 8a7 7 0 0 1-9 10zm0 0v-8"></path></svg>`
    };

    // 卡牌庫範本
    const CARD_TEMPLATES = [
      { id: 't1', type: 'policy', name: '徵收碳稅', desc: '下回合預算 +40，民意 -10', cost: 0, ap: 1, effect: (state) => ({ budget: state.budget + 40, opinion: state.opinion - 10 }), iconName: 'dollar' },
      { id: 't2', type: 'policy', name: '綠能補貼', desc: '花費 $20。全域電力需求 -10，民意 +5', cost: 20, ap: 1, effect: (state) => ({ baseDemand: Math.max(0, state.baseDemand - 10), opinion: state.opinion + 5 }), iconName: 'zap' },
      { id: 't3', type: 'policy', name: '推動居家辦公', desc: '全域碳排 -10，電力需求 -5，民意 -5', cost: 0, ap: 1, effect: (state) => ({ baseEmissions: Math.max(0, state.baseEmissions - 10), baseDemand: Math.max(0, state.baseDemand - 5), opinion: state.opinion - 5 }), iconName: 'users' },
      { id: 't4', type: 'policy', name: '研發 CCUS', desc: '花費 $40, AP 2。突破性減碳：全域碳排永久 -20', cost: 40, ap: 2, effect: (state) => ({ baseEmissions: Math.max(0, state.baseEmissions - 20) }), iconName: 'factory' },
      { id: 't5', type: 'build', buildType: 'coal', name: '建燃煤電廠', desc: '花費 $10。建設：供電 +40，碳排 +20', cost: 10, ap: 1, iconName: 'factory' },
      { id: 't6', type: 'build', buildType: 'wind', name: '建風力發電', desc: '花費 $25。建設：供電 +20，無碳排', cost: 25, ap: 1, iconName: 'wind' },
      { id: 't7', type: 'build', buildType: 'nuclear', name: '建核能電廠', desc: '花費 $60, AP 2。建設：供電 +60，民意 -10', cost: 60, ap: 2, effect: (state) => ({ opinion: state.opinion - 10 }), iconName: 'zap' },
      { id: 't8', type: 'build', buildType: 'forest', name: '復育森林', desc: '花費 $15。建設：碳匯 +15', cost: 15, ap: 1, iconName: 'tree' },
      { id: 't9', type: 'policy', name: '環保宣導', desc: '花費 $10。民意 +15', cost: 10, ap: 1, effect: (state) => ({ opinion: state.opinion + 15 }), iconName: 'leaf' },
    ];

    // --- 遊戲核心狀態管理 ---
    let state = {
      gameState: 'menu', // menu, playing, won, lost_temp, lost_budget, lost_opinion
      turn: 1,
      ap: 3,
      budget: BUDGET_START,
      negativeBudgetTurns: 0,
      opinion: OPINION_START,
      temperature: TEMP_START,
      baseEmissions: 30,
      baseDemand: 50,
      mapData: [],
      deck: [],
      hand: [],
      discard: [],
      selectedCard: null,
      logs: [],
      activeEvent: null,
      customModal: null // 自訂通知對話框訊息
    };

    // 產生初始地圖
    function getInitialMap() {
      return Array.from({ length: MAP_SIZE }, (_, i) => {
        if (i >= 0 && i <= 4) return { id: i, type: 'coast', name: '海岸線', emits: 0, sink: 0, energy: 0 };
        if (i === 12 || i === 13 || i === 17) return { id: i, type: 'city', name: '城市', emits: 15, sink: 0, energy: -20 };
        return { id: i, type: 'empty', name: '空地', emits: 0, sink: 0, energy: 0 };
      });
    }

    // 新增日誌
    function addLog(msg) {
      state.logs.unshift(`[回合 ${state.turn}] ${msg}`);
      if (state.logs.length > 10) state.logs.pop();
    }

    // 自訂對話框通知 (取代 browser alert)
    function showNotification(message) {
      state.customModal = message;
      render();
    }

    function closeNotification() {
      state.customModal = null;
      render();
    }

    // 產生並洗牌
    function generateDeck() {
      let deck = [];
      let uid = 0;
      const addCards = (id, count) => {
        const template = CARD_TEMPLATES.find(c => c.id === id);
        for (let i = 0; i < count; i++) {
          deck.push({ ...template, uid: `card_${uid++}` });
        }
      };
      addCards('t1', 3);
      addCards('t2', 3);
      addCards('t3', 2);
      addCards('t4', 1);
      addCards('t5', 4);
      addCards('t6', 4);
      addCards('t7', 1);
      addCards('t8', 3);
      addCards('t9', 2);
      return deck.sort(() => Math.random() - 0.5);
    }

    // 啟動遊戲
    function startGame() {
      state.gameState = 'playing';
      state.turn = 1;
      state.ap = 3;
      state.budget = BUDGET_START;
      state.negativeBudgetTurns = 0;
      state.opinion = OPINION_START;
      state.temperature = TEMP_START;
      state.baseEmissions = 30;
      state.baseDemand = 50;
      state.mapData = getInitialMap();
      state.logs = [];
      state.activeEvent = null;
      state.selectedCard = null;
      state.customModal = null;

      const newDeck = generateDeck();
      state.hand = newDeck.slice(0, 5);
      state.deck = newDeck.slice(5);
      state.discard = [];
      
      addLog("遊戲開始，長官。請在 30 回合內拯救世界。");
      render();
    }

    // 抽卡
    function drawCards(count) {
      for (let i = 0; i < count; i++) {
        if (state.deck.length === 0) {
          if (state.discard.length === 0) break;
          state.deck = state.discard.sort(() => Math.random() - 0.5);
          state.discard = [];
          addLog("牌組已重新洗牌。");
        }
        state.hand.push(state.deck.pop());
      }
    }

    // 點擊卡牌
    window.handleCardClick = function(uid) {
      const card = state.hand.find(c => c.uid === uid);
      if (!card) return;

      if (state.ap < card.ap) {
        showNotification("行動點 (AP) 不足！");
        return;
      }
      if (state.budget < card.cost) {
        showNotification("政府預算不足！");
        return;
      }

      if (card.type === 'policy') {
        state.ap -= card.ap;
        state.budget -= card.cost;
        if (card.effect) {
          const updates = card.effect(state);
          if (updates.budget !== undefined) state.budget = updates.budget;
          if (updates.opinion !== undefined) state.opinion = updates.opinion;
          if (updates.baseDemand !== undefined) state.baseDemand = updates.baseDemand;
          if (updates.baseEmissions !== undefined) state.baseEmissions = updates.baseEmissions;
        }
        addLog(`頒布政策：${card.name}`);
        state.hand = state.hand.filter(c => c.uid !== uid);
        state.discard.push(card);
        state.selectedCard = null;
        render();
      } else {
        state.selectedCard = state.selectedCard?.uid === uid ? null : card;
        render();
      }
    };

    // 點擊地圖格子
    window.handleCellClick = function(cellIndex) {
      if (!state.selectedCard || state.selectedCard.type !== 'build') return;
      const cell = state.mapData[cellIndex];
      if (cell.type === 'city') {
        showNotification("無法在城市上建設！");
        return;
      }
      if (cell.type === 'flooded') {
        showNotification("該區域已遭海水淹沒！");
        return;
      }
      if (cell.type !== 'empty' && cell.type !== 'coast') {
        showNotification("該地塊已被佔用！");
        return;
      }

      state.ap -= state.selectedCard.ap;
      state.budget -= state.selectedCard.cost;

      if (state.selectedCard.effect) {
        const updates = state.selectedCard.effect(state);
        if (updates.opinion !== undefined) state.opinion = updates.opinion;
      }

      let buildStats = { emits: 0, sink: 0, energy: 0 };
      if (state.selectedCard.buildType === 'coal') buildStats = { emits: 20, energy: 40 };
      if (state.selectedCard.buildType === 'wind') buildStats = { emits: 0, energy: 20 };
      if (state.selectedCard.buildType === 'nuclear') buildStats = { emits: 0, energy: 60 };
      if (state.selectedCard.buildType === 'forest') buildStats = { sink: 15 };

      state.mapData[cellIndex] = {
        ...cell,
        type: state.selectedCard.buildType,
        name: state.selectedCard.name,
        ...buildStats
      };

      addLog(`建設完成：在區域建立 ${state.selectedCard.name}`);
      state.hand = state.hand.filter(c => c.uid !== state.selectedCard.uid);
      state.discard.push(state.selectedCard);
      state.selectedCard = null;
      render();
    };

    // 計算當前產出數值
    function calculateCurrentStats() {
      let mapEmissions = 0;
      let mapSinks = 0;
      let mapEnergySupply = 0;
      let mapEnergyDemand = 0;

      state.mapData.forEach(cell => {
        if (cell.emits) mapEmissions += cell.emits;
        if (cell.sink) mapSinks += cell.sink;
        if (cell.energy > 0) mapEnergySupply += cell.energy;
        if (cell.energy < 0) mapEnergyDemand += Math.abs(cell.energy);
      });

      const totalEmissions = state.baseEmissions + mapEmissions;
      const totalDemand = state.baseDemand + mapEnergyDemand;
      const netEmissions = totalEmissions - mapSinks;
      const energyShortage = Math.max(0, totalDemand - mapEnergySupply);
      const supplyRate = totalDemand > 0 ? Math.min(100, Math.floor((mapEnergySupply / totalDemand) * 100)) : 100;

      return { totalEmissions, mapSinks, netEmissions, totalDemand, mapEnergySupply, energyShortage, supplyRate };
    }

    // 結算回合
    window.handleEndTurn = function() {
      const stats = calculateCurrentStats();
      let currentOpinion = state.opinion;
      let currentBudget = state.budget;
      let currentTemp = state.temperature;
      let evtMsg = null;

      // 1. 結算暖化
      if (stats.netEmissions > 0) {
        currentTemp += (stats.netEmissions * 0.002);
      } else if (stats.netEmissions < 0) {
        currentTemp += (stats.netEmissions * 0.0005);
      }

      // 2. 結算電力
      if (stats.energyShortage > 0) {
        const penalty = Math.ceil(stats.energyShortage * 0.5);
        currentOpinion -= penalty;
        addLog(`⚠️ 電力短缺！民意下降 ${penalty} 點。`);
      }

      // 3. 固定稅收
      currentBudget += 30;

      // 4. 隨機事件
      if (Math.random() < 0.25) {
        const events = [
          { name: '極端熱浪', effect: () => { state.baseDemand += 15; evtMsg = "極端熱浪襲來，全域用電量暴增！"; } },
          { name: '無風日', effect: () => { evtMsg = "氣候異常導致無風，風力發電本回合效率受限。"; } },
          { name: '國際油價大漲', effect: () => { currentBudget -= 20; evtMsg = "國際能源價格攀升，預算遭重挫！"; } },
          { name: '氣候峰會', effect: () => { currentOpinion += 10; evtMsg = "國際氣候峰會反響極佳，民意獲得提升。"; } }
        ];
        const ev = events[Math.floor(Math.random() * events.length)];
        ev.effect();
        if (evtMsg) {
          state.activeEvent = evtMsg;
          addLog(`事件：${evtMsg}`);
        }
      } else {
        state.activeEvent = null;
      }

      // 5. 海平面上升 (溫升 >= 1.6)
      if (currentTemp >= 1.6) {
        let floodedCount = 0;
        state.mapData = state.mapData.map(cell => {
          if (cell.type === 'coast' || (cell.id < 5 && cell.type !== 'flooded')) {
            floodedCount++;
            return { ...cell, type: 'flooded', name: '遭淹沒', emits: 0, sink: 0, energy: 0 };
          }
          return cell;
        });
        if (floodedCount > 0) addLog("🌊 全球暖化加劇，海岸線遭遇海水淹沒！");
      }

      state.temperature = currentTemp;
      state.opinion = Math.max(0, currentOpinion);
      state.budget = currentBudget;

      const negTurns = currentBudget < 0 ? state.negativeBudgetTurns + 1 : 0;
      state.negativeBudgetTurns = negTurns;

      // 勝利 / 失敗條件判定
      if (currentTemp >= TEMP_CRITICAL) {
        state.gameState = 'lost_temp';
        render();
        return;
      }
      if (negTurns >= 3) {
        state.gameState = 'lost_budget';
        render();
        return;
      }
      if (state.opinion <= 0) {
        state.gameState = 'lost_opinion';
        render();
        return;
      }
      if (state.turn >= MAX_TURNS) {
        if (stats.netEmissions <= 0 && state.opinion > 0) {
          state.gameState = 'won';
        } else {
          state.gameState = 'lost_temp';
        }
        render();
        return;
      }

      // 下一回合準備
      state.turn += 1;
      state.ap = 3;

      // 丟棄手牌並抽取 5 張
      const discardAll = [...state.discard, ...state.hand];
      state.hand = [];
      state.discard = discardAll;
      drawCards(5);
      state.selectedCard = null;
      render();
    };

    // --- UI 動態渲染引擎 ---
    function render() {
      const root = document.getElementById('app');
      
      // 1. Menu 畫面
      if (state.gameState === 'menu') {
        root.innerHTML = `
          <div class="min-h-full bg-slate-950 text-white flex flex-col items-center justify-center p-6 font-sans">
            <div class="max-w-2xl text-center space-y-6">
              ${ICONS.globe}
              <h1 class="text-5xl font-bold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-emerald-400 to-cyan-400">
                零碳防線：2050
              </h1>
              <p class="text-xl text-slate-300">
                身為永續發展委員會最高首長，您必須在 30 回合內帶領國家達成「淨零碳排」。
              </p>
              <div class="bg-slate-900 p-6 rounded-xl text-left text-sm text-slate-400 space-y-3 border border-slate-800">
                <p>🌍 <strong>目標：</strong> 在第 30 回合結束時，達成「排放量 ≤ 碳匯量」，且民意 &gt; 0。</p>
                <p>🔥 <strong>危機：</strong> 若全球升溫達到 2.0°C，將引發不可逆的氣候災難（遊戲結束）。</p>
                <p>💰 <strong>財政：</strong> 建設需要預算，連續 3 回合預算低於零即宣告破產。</p>
                <p>⚡ <strong>電網：</strong> 若電力供不應求，會重創經濟與民意支持度。</p>
              </div>
              <button onclick="startGame()" class="mt-8 px-8 py-4 bg-emerald-600 hover:bg-emerald-500 text-white rounded-full font-bold text-lg transition-all shadow-lg shadow-emerald-900/50 flex items-center justify-center mx-auto">
                開始執政
              </button>
            </div>
          </div>
        `;
        return;
      }

      // 2. 結算/輸贏畫面
      if (state.gameState.startsWith('lost') || state.gameState === 'won') {
        const isWin = state.gameState === 'won';
        let failDesc = "";
        if (state.gameState === 'lost_temp') failDesc = "全球升溫突破 2.0°C，生態氣候崩潰。";
        if (state.gameState === 'lost_budget') failDesc = "預算赤字連續 3 回合，國家財政破產。";
        if (state.gameState === 'lost_opinion') failDesc = "民意歸零，引發全國性大罷工，您已被罷免下台。";

        root.innerHTML = `
          <div class="min-h-full bg-slate-950 text-white flex flex-col items-center justify-center p-6">
            <div class="max-w-md w-full p-8 rounded-2xl text-center border-2 ${isWin ? 'border-emerald-500 bg-emerald-900/20' : 'border-red-500 bg-red-900/20'}">
              ${isWin ? ICONS.leaf : ICONS.skull}
              <h1 class="text-4xl font-bold mb-4">${isWin ? '淨零防線達成！' : '執政任務失敗'}</h1>
              <p class="text-lg text-slate-300 mb-6">
                ${isWin ? '您成功在 2050 年前引導國家完成艱難的綠能與零碳轉型，守護了我們的未來！' : failDesc}
              </p>
              <div class="bg-black/40 p-4 rounded-lg text-left text-sm mb-6 space-y-2">
                <p>最終升溫: ${state.temperature.toFixed(2)} °C</p>
                <p>最終回合: ${state.turn} / 30</p>
                <p>民意支持: ${state.opinion}%</p>
                <p>國庫結餘: $${state.budget}</p>
              </div>
              <button onclick="startGame()" class="w-full py-3 bg-slate-800 hover:bg-slate-700 rounded-lg font-bold flex justify-center items-center transition-all">
                重新挑戰
              </button>
            </div>
          </div>
        `;
        return;
      }

      // 3. 玩遊戲主畫面
      const stats = calculateCurrentStats();

      // 地圖溫度背景色變化
      let mapBg = 'bg-emerald-900';
      if (state.temperature >= 1.4 && state.temperature < 1.7) mapBg = 'bg-yellow-950';
      else if (state.temperature >= 1.7 && state.temperature < 1.9) mapBg = 'bg-orange-950';
      else if (state.temperature >= 1.9) mapBg = 'bg-red-950';

      // 產生手牌 HTML
      const handHTML = state.hand.map(card => {
        const isSelected = state.selectedCard?.uid === card.uid;
        const canAfford = state.ap >= card.ap && state.budget >= card.cost;
        return `
          <div onclick="handleCardClick('${card.uid}')" 
               class="w-32 h-44 rounded-xl bg-slate-800 border flex flex-col relative cursor-pointer transition-all duration-200 shadow-lg
               ${isSelected ? 'border-cyan-400 -translate-y-6 shadow-[0_0_12px_rgba(34,211,238,0.5)]' : 'border-slate-600 hover:-translate-y-2 hover:border-slate-400'}
               ${!canAfford ? 'opacity-50 grayscale cursor-not-allowed' : ''}">
            <div class="bg-slate-900 px-2 py-1 rounded-t-xl flex justify-between items-center text-[10px] font-bold border-b border-slate-700">
              <span class="flex items-center text-cyan-300">AP: ${card.ap}</span>
              <span class="flex items-center text-yellow-400">$${card.cost}</span>
            </div>
            <div class="absolute -top-2.5 left-1/2 transform -translate-x-1/2 bg-slate-700 text-[9px] px-1.5 py-0.5 rounded-full border border-slate-600">
              ${card.type === 'build' ? '基礎建設' : '政策'}
            </div>
            <div class="p-2.5 flex-1 flex flex-col justify-between">
              <div class="text-xs font-bold text-center mb-1 text-slate-200 mt-1">${card.name}</div>
              <div class="text-[10px] text-slate-400 leading-snug text-center mb-1">
                ${card.desc}
              </div>
            </div>
          </div>
        `;
      }).join('');

      // 產生 AP 點數指示器
      const apDots = Array.from({ length: 3 }, (_, i) => {
        const active = i < state.ap;
        return `<div class="w-2.5 h-2.5 rounded-full ${active ? 'bg-cyan-400 shadow-[0_0_8px_rgba(34,211,238,0.8)]' : 'bg-slate-700'}"></div>`;
      }).join('');

      // 產生網格地圖
      const mapHTML = state.mapData.map((cell, idx) => {
        let cellColor = 'bg-gray-800';
        let iconMarkup = '';

        if (cell.type === 'city') {
          cellColor = 'bg-slate-700 border-slate-500 shadow-inner';
          iconMarkup = ICONS.factory;
        } else if (cell.type === 'coast') {
          cellColor = 'bg-emerald-800/40 border-emerald-700/30';
        } else if (cell.type === 'empty') {
          cellColor = 'bg-transparent border-white/10 hover:bg-white/5';
        } else if (cell.type === 'coal') {
          cellColor = 'bg-neutral-800 border-neutral-600';
          iconMarkup = ICONS.cloudRain;
        } else if (cell.type === 'wind') {
          cellColor = 'bg-sky-800 border-sky-400';
          iconMarkup = ICONS.wind;
        } else if (cell.type === 'nuclear') {
          cellColor = 'bg-purple-900 border-purple-400';
          iconMarkup = ICONS.zap;
        } else if (cell.type === 'forest') {
          cellColor = 'bg-green-700 border-green-500';
          iconMarkup = ICONS.tree;
        } else if (cell.type === 'flooded') {
          cellColor = 'bg-blue-900 border-blue-950 opacity-50';
          iconMarkup = ICONS.alert;
        }

        const buildActiveHighlight = state.selectedCard?.type === 'build' && (cell.type === 'empty' || cell.type === 'coast');

        return `
          <div onclick="handleCellClick(${idx})" 
               class="w-16 h-16 sm:w-20 sm:h-20 rounded-lg border-2 flex flex-col items-center justify-center cursor-pointer transition-all relative overflow-hidden group
               ${cellColor} ${buildActiveHighlight ? 'ring-2 ring-cyan-400 ring-offset-2 ring-offset-slate-900 hover:scale-105' : ''}">
            ${iconMarkup}
            <span class="text-[10px] font-bold mt-1 opacity-80 z-10 text-center pointer-events-none">${cell.name}</span>
            
            ${cell.type !== 'empty' && cell.type !== 'flooded' ? `
              <div class="absolute inset-0 bg-black/80 flex flex-col items-center justify-center opacity-0 group-hover:opacity-100 transition-opacity z-20 text-[9px] space-y-0.5">
                ${cell.emits > 0 ? `<span class="text-red-400">碳排 +${cell.emits}</span>` : ''}
                ${cell.sink > 0 ? `<span class="text-green-400">碳匯 +${cell.sink}</span>` : ''}
                ${cell.energy > 0 ? `<span class="text-yellow-400">電力 +${cell.energy}</span>` : ''}
                ${cell.energy < 0 ? `<span class="text-orange-300">耗電 ${cell.energy}</span>` : ''}
              </div>
            ` : ''}
          </div>
        `;
      }).join('');

      // 組合主畫面 HTML
      root.innerHTML = `
        <!-- 頂部狀態控制列 -->
        <header class="bg-slate-900 border-b border-slate-800 p-2.5 grid grid-cols-3 items-center shadow-md z-10">
          <!-- 左：氣候溫度計 -->
          <div class="flex space-x-4">
            <div class="flex items-center space-x-2">
              <div class="${state.temperature > 1.8 ? 'text-red-500 animate-pulse' : 'text-amber-500'}">
                ${ICONS.thermometer}
              </div>
              <div>
                <div class="text-[10px] text-slate-400">全球升溫</div>
                <div class="font-bold text-base ${state.temperature >= 1.8 ? 'text-red-400' : ''}">
                  +${state.temperature.toFixed(2)}°C
                </div>
              </div>
            </div>
            <div class="flex items-center space-x-2">
              <div class="text-slate-400">${ICONS.cloudRain}</div>
              <div>
                <div class="text-[10px] text-slate-400">淨碳排/回合</div>
                <div class="font-bold text-base">
                  ${stats.netEmissions > 0 ? `+${stats.netEmissions}` : stats.netEmissions}
                  <span class="text-[10px] text-slate-500 ml-1">(排 ${stats.totalEmissions} - 匯 ${stats.mapSinks})</span>
                </div>
              </div>
            </div>
          </div>

          <!-- 中間：回合數與 AP -->
          <div class="text-center">
            <div class="text-xs font-bold tracking-widest text-emerald-500 mb-0.5">
              ROUND ${state.turn} <span class="text-slate-600">/ ${MAX_TURNS}</span>
            </div>
            <div class="flex justify-center space-x-1">${apDots}</div>
          </div>

          <!-- 右：關鍵資源 -->
          <div class="flex justify-end space-x-4">
            <div class="flex items-center space-x-2">
              <div class="${state.budget < 0 ? 'text-red-500' : 'text-yellow-500'}">${ICONS.dollar}</div>
              <div>
                <div class="text-[10px] text-slate-400">預算</div>
                <div class="font-bold text-base ${state.budget < 0 ? 'text-red-400' : ''}">$${state.budget}</div>
              </div>
            </div>
            <div class="flex items-center space-x-2">
              <div class="${stats.supplyRate < 100 ? 'text-red-500' : 'text-yellow-400'}">${ICONS.zap}</div>
              <div>
                <div class="text-[10px] text-slate-400">電力供應</div>
                <div class="font-bold text-base ${stats.supplyRate < 100 ? 'text-red-400' : ''}">${stats.supplyRate}%</div>
              </div>
            </div>
            <div class="flex items-center space-x-2">
              <div class="${state.opinion < 30 ? 'text-red-500' : 'text-blue-400'}">${ICONS.users}</div>
              <div>
                <div class="text-[10px] text-slate-400">民意</div>
                <div class="font-bold text-base">${state.opinion}%</div>
              </div>
            </div>
          </div>
        </header>

        <!-- 主要控制面板 -->
        <main class="flex-1 flex overflow-hidden">
          <!-- 左側通報 -->
          <aside class="w-52 bg-slate-900 border-r border-slate-800 p-3 flex flex-col">
            <h2 class="text-[10px] font-bold text-slate-500 tracking-wider mb-2.5 uppercase">狀態與通報</h2>
            
            ${state.activeEvent ? `
              <div class="bg-amber-900/30 border border-amber-700/50 rounded-lg p-2.5 mb-3 text-xs text-amber-200 flex items-start">
                ${ICONS.alert}
                <span>${state.activeEvent}</span>
              </div>
            ` : ''}

            <div class="flex-1 overflow-y-auto space-y-1.5 pr-1.5 custom-scrollbar">
              ${state.logs.map(log => `<div class="text-[11px] text-slate-400 bg-slate-800/50 p-1.5 rounded">${log}</div>`).join('')}
            </div>
            
            <div class="mt-3 pt-3 border-t border-slate-800">
              <button onclick="handleEndTurn()" class="w-full py-2 bg-indigo-600 hover:bg-indigo-500 text-white text-sm font-bold rounded-lg transition-colors flex justify-center items-center shadow-lg">
                結束回合
              </button>
            </div>
          </aside>

          <!-- 地圖主視角 -->
          <section class="flex-1 relative flex flex-col items-center justify-center transition-colors duration-1000 ${mapBg}">
            <div class="absolute inset-0 bg-[linear-gradient(to_right,#ffffff05_1px,transparent_1px),linear-gradient(to_bottom,#ffffff05_1px,transparent_1px)] bg-[size:40px_40px]"></div>
            
            ${state.selectedCard?.type === 'build' ? `
              <div class="absolute top-4 bg-blue-900/80 border border-blue-500 text-blue-100 px-4 py-1.5 rounded-full text-xs font-bold shadow-lg flex items-center animate-bounce z-10">
                請點擊地圖選擇建設位置
                <button onclick="state.selectedCard = null; render();" class="ml-3 text-blue-300 hover:text-white">✕</button>
              </div>
            ` : ''}

            <div class="relative z-0 grid grid-cols-5 gap-1 p-3 bg-slate-900/40 rounded-xl backdrop-blur-sm border border-slate-700/50 shadow-2xl">
              ${mapHTML}
            </div>
          </section>
        </main>

        <!-- 手牌區 -->
        <footer class="bg-slate-900 border-t border-slate-800 p-4 h-52 relative">
          <div class="absolute top-1 left-4 text-[10px] text-slate-500 font-bold tracking-widest uppercase">
            政策手牌 (${state.hand.length}/5) | 牌庫: ${state.deck.length}
          </div>
          <div class="flex justify-center items-end space-x-3 h-full mt-1">
            ${handHTML.length > 0 ? handHTML : `<div class="w-full text-center text-slate-500 text-xs italic flex items-center justify-center h-full">手牌已空，請結束回合以重新抽卡。</div>`}
          </div>
        </footer>

        <!-- 自訂通知 Modal (替代 alert) -->
        ${state.customModal ? `
          <div class="fixed inset-0 bg-black/70 flex items-center justify-center z-50">
            <div class="bg-slate-900 border border-slate-700 rounded-xl max-w-xs w-full p-5 text-center space-y-3 shadow-2xl animate-in fade-in zoom-in-95 duration-150">
              <div class="text-amber-500 mx-auto w-10 h-10 flex items-center justify-center bg-amber-500/10 rounded-full">
                <svg class="w-5 h-5" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                  <path d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"></path>
                </svg>
              </div>
              <h3 class="text-base font-bold text-slate-100">決策指示</h3>
              <p class="text-xs text-slate-300 leading-relaxed">${state.customModal}</p>
              <button onclick="closeNotification()" class="px-5 py-1.5 bg-indigo-600 hover:bg-indigo-500 text-white rounded-lg font-bold text-xs transition-all shadow-md">
                確認
              </button>
            </div>
          </div>
        ` : ''}
      `;
    }

    // 當文件完全加載完成時
    window.onload = function() {
      startGame();
      state.gameState = 'menu'; // 重設至主選單開始畫面
      render();
    };
  </script>
</body>
</html>
