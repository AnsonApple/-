<!DOCTYPE html>
<html lang="zh-TW" class="light">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>五子棋模擬器 (Gomoku)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            darkMode: 'class', 
            theme: {
                extend: {
                    colors: {
                        board: {
                            light: '#E6C28F'
                        }
                    }
                }
            }
        }
    </script>
    <style>
        :root {
            --top-spacing: 0px;
            --bottom-spacing: 0px;
        }
        body {
            height: 100dvh;
            width: 100vw;
            transition: background-color 0.3s, color 0.3s;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            touch-action: manipulation;
            overflow: hidden; 
        }
        canvas {
            -webkit-tap-highlight-color: transparent;
            touch-action: none;
        }
        .panel-glass {
            background: rgba(15, 23, 42, 0.94);
            backdrop-filter: blur(16px);
            border: 1px solid rgba(255, 255, 255, 0.15);
        }
        .drag-handle { cursor: grab; }
        .drag-handle:active { cursor: grabbing; }
        
        .cheat-btn-active {
            background-color: #ef4444 !important; 
            border-color: #f87171 !important;
            color: white !important;
        }

        #topGroup {
            padding-top: var(--top-spacing);
            transition: padding-top 0.1s ease-out;
        }
        #bottomGroup {
            padding-bottom: var(--bottom-spacing);
            transition: padding-bottom 0.1s ease-out;
        }

        /* 🍏 蘋果原生 Switch 樣式 */
        .apple-native-switch {
            appearance: switch;
            -webkit-appearance: switch;
            width: 51px;
            height: 31px;
            accent-color: #34c759; /* 其他普通開關全部回歸標準 iOS 經典綠色 */
            outline: none;
            cursor: pointer;
            -webkit-tap-highlight-color: transparent;
        }

        /* 🟡 開發者模式開關：獨享黃色 */
        .apple-native-switch.switch-dev {
            accent-color: #eab308;
        }

        /* 🔴 外掛/作弊模式開關：獨享紅色 */
        .apple-native-switch.switch-cheat {
            accent-color: #ef4444;
        }
    </style>
</head>
<body class="flex flex-col bg-slate-50 dark:bg-slate-950 text-slate-800 dark:text-slate-100 select-none">

    <div id="topGroup" class="flex flex-col shrink-0">
        <header class="flex justify-between items-center px-4 md:px-8 py-3 bg-white/80 dark:bg-slate-900/80 backdrop-blur-md shadow-sm z-20">
            <button id="settingsBtn" class="p-2 rounded-full hover:bg-slate-200 dark:hover:bg-slate-800 transition-colors text-slate-600 dark:text-slate-300 focus:outline-none">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
                  <path stroke-linecap="round" stroke-linejoin="round" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
                </svg>
            </button>
            
            <h1 class="text-xl md:text-2xl font-black tracking-widest absolute left-1/2 transform -translate-x-1/2">五子棋</h1>
            <div id="roomIndicator" class="text-xs font-mono font-bold bg-indigo-100 text-indigo-700 dark:bg-indigo-950/50 dark:text-indigo-300 px-2.5 py-1 rounded-full hidden">房號: ----</div>
            <div class="w-10"></div>
        </header>

        <div class="px-4 md:px-8 py-2 bg-white dark:bg-slate-900 shadow-sm z-10 flex flex-col sm:flex-row sm:justify-between items-center gap-2 border-b border-slate-200 dark:border-slate-800">
            <div class="flex items-center gap-4">
                <div id="statusIndicator" class="text-base md:text-lg font-bold flex items-center gap-2">
                    <span id="statusDot" class="w-3.5 h-3.5 rounded-full bg-black inline-block shadow-md ring-1 ring-gray-400"></span>
                    <span id="statusText">輪到：黑子</span>
                </div>
                
                <div id="practiceAnalysisContainer" class="hidden flex items-center gap-1.5 border-l border-slate-300 dark:border-slate-700 pl-4">
                    <label class="relative inline-flex items-center cursor-pointer">
                        <input type="checkbox" id="realTimeAnalysisToggle" class="sr-only peer">
                        <div class="w-8 h-4 bg-slate-300 dark:bg-slate-700 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-3 after:w-3 after:transition-all peer-checked:bg-green-500 shadow-inner"></div>
                        <span class="ml-1.5 text-xs font-bold text-green-600 dark:text-green-400">即時 AI 分析</span>
                    </label>
                </div>
            </div>
            
            <div class="w-full sm:w-1/2 max-w-md flex flex-col justify-center">
                <div class="flex justify-between text-[11px] font-bold mb-1 items-end">
                    <span id="winRateTextBlack" class="text-slate-700 dark:text-gray-300">黑 50.0%</span>
                    <span class="text-slate-400 dark:text-slate-500 font-normal">AI 局勢分析</span>
                    <span id="winRateTextWhite" class="text-slate-700 dark:text-gray-300">白 50.0%</span>
                </div>
                <div class="w-full h-1.5 rounded-full overflow-hidden flex shadow-inner bg-gray-200 dark:bg-gray-800 relative">
                    <div id="winRateBarBlack" class="h-full bg-slate-800 dark:bg-black transition-all duration-700 ease-out" style="width: 50%;"></div>
                    <div id="winRateBarWhite" class="h-full flex-grow bg-white dark:bg-slate-300 transition-all duration-700 ease-out"></div>
                    <div class="absolute left-1/2 top-0 h-full w-[1px] bg-red-500/60 transform -translate-x-1/2"></div>
                </div>
            </div>
        </div>
    </div>

    <!-- 主遊戲介面，局後覆盤導航條 -->
    <main id="mainGrid" class="flex-1 min-h-0 w-full flex flex-col items-center justify-center p-4 relative bg-slate-50 dark:bg-slate-950 gap-4">
        <div id="aiCalculatingText" class="absolute top-4 left-1/2 transform -translate-x-1/2 text-red-500 font-bold bg-white/95 dark:bg-slate-900/95 px-4 py-1.5 rounded-full shadow-lg z-20 hidden animate-pulse border border-red-200 dark:border-red-900 text-xs sm:text-sm">
            AI 運算中...
        </div>
        
        <div id="boardContainer" class="relative flex items-center justify-center transition-all duration-100">
            <canvas id="board" class="shadow-2xl rounded-sm w-full h-full object-contain aspect-square bg-board-light transition-colors duration-300"></canvas>
        </div>

        <!-- 局後覆盤導航器面板 -->
        <div id="reviewNavigator" class="hidden w-full max-w-md bg-white/90 dark:bg-slate-900/90 backdrop-blur border border-slate-200 dark:border-slate-800 rounded-xl p-2.5 flex items-center justify-between gap-3 shadow-md">
            <button id="revFirstBtn" class="p-2 rounded-lg hover:bg-slate-100 dark:hover:bg-slate-850 active:scale-95 text-slate-600 dark:text-slate-300 text-xs font-bold">⏮ 首步</button>
            <button id="revPrevBtn" class="flex-1 py-2 rounded-lg bg-slate-100 dark:bg-slate-800 hover:bg-slate-200 dark:hover:bg-slate-700 active:scale-95 text-slate-700 dark:text-slate-200 text-xs font-black">◀ 上一步</button>
            <div id="reviewCounter" class="text-xs font-mono font-bold text-blue-600 dark:text-blue-400 whitespace-nowrap min-w-[50px] text-center">0 / 0 手</div>
            <button id="revNextBtn" class="flex-1 py-2 rounded-lg bg-slate-100 dark:bg-slate-800 hover:bg-slate-200 dark:hover:bg-slate-700 active:scale-95 text-slate-700 dark:text-slate-200 text-xs font-black">下一步 ▶</button>
            <button id="revLastBtn" class="p-2 rounded-lg hover:bg-slate-100 dark:hover:bg-slate-850 active:scale-95 text-slate-600 dark:text-slate-300 text-xs font-bold">末步 ⏭</button>
        </div>
    </main>

    <footer id="bottomGroup" class="shrink-0 bg-white/95 dark:bg-slate-900/95 backdrop-blur-md shadow-lg border-t border-slate-200 dark:border-slate-800 p-3 sm:p-4 z-20 w-full">
        <div class="max-w-4xl mx-auto flex flex-wrap gap-2 sm:gap-3 justify-center items-center">
            <select id="gameMode" class="flex-1 min-w-[130px] appearance-none bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-200 text-xs sm:text-sm font-semibold py-2.5 px-3 rounded-lg border border-slate-300 dark:border-slate-700 focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:opacity-40 disabled:cursor-not-allowed cursor-pointer shadow-sm text-center transition-opacity">
                <option value="pve">單人對戰 AI</option>
                <option value="pvp">雙人對戰 PVP</option>
                <option value="practice">自主練習模式</option>
            </select>

            <div id="aiDifficultyContainer" class="flex-1 min-w-[100px] transition-all duration-300">
                <select id="aiDifficulty" class="w-full appearance-none bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-200 text-xs sm:text-sm font-semibold py-2.5 px-3 rounded-lg border border-slate-300 dark:border-slate-700 focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:opacity-40 disabled:cursor-not-allowed cursor-pointer shadow-sm text-center">
                    <option value="1">難度 1</option>
                    <option value="2">難度 2</option>
                    <option value="3">難度 3</option>
                    <option value="4">難度 4</option>
                    <option value="5">難度 5</option>
                    <option value="6" class="text-red-600 dark:text-red-400">難度 6</option>
                    <option value="7" selected class="text-purple-600 dark:text-purple-400 font-bold bg-purple-50 dark:bg-purple-900/20">難度 7</option>
                </select>
            </div>

            <div id="aiColorContainer" class="flex-1 min-w-[110px] transition-all duration-300">
                <select id="aiColor" class="w-full appearance-none bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-200 text-xs sm:text-sm font-semibold py-2.5 px-3 rounded-lg border border-slate-300 dark:border-slate-700 focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:opacity-40 disabled:cursor-not-allowed cursor-pointer shadow-sm text-center">
                    <option value="2" selected>玩家執黑</option>
                    <option value="1">AI 執黑</option>
                </select>
            </div>

            <button id="practiceUndoBtn" class="hidden flex-1 min-w-[90px] bg-slate-600 hover:bg-slate-700 text-white text-xs sm:text-sm font-bold py-2.5 px-4 rounded-lg transition duration-200 shadow-md whitespace-nowrap">
                ↩ 悔棋
            </button>

            <button id="startBtn" class="flex-1 min-w-[90px] bg-blue-600 hover:bg-blue-700 text-white text-xs sm:text-sm font-bold py-2.5 px-4 rounded-lg transition duration-200 shadow-md whitespace-nowrap">
                開始遊戲
            </button>
        </div>
    </footer>

    <!-- 🍏 遊戲設定彈窗 (原生 Switch 重構) -->
    <div id="settingsModal" class="fixed inset-0 z-[70] hidden bg-slate-900/60 backdrop-blur-sm transition-opacity">
        <div class="flex items-center justify-center w-full h-full p-4 pointer-events-none">
            <div class="bg-white dark:bg-slate-900 p-6 rounded-2xl shadow-2xl max-w-xs w-full border border-slate-200 dark:border-slate-800 relative transform transition-all pointer-events-auto">
                <button id="closeSettingsBtn" class="absolute top-4 right-4 text-slate-400 hover:text-slate-600 dark:hover:text-white transition-colors">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" /></svg>
                </button>
                <h2 class="text-lg font-bold mb-6 text-slate-800 dark:text-white flex items-center gap-2">
                    遊戲設定
                </h2>
                
                <div class="space-y-4">
                    <div class="flex items-center justify-between border-b border-slate-100 dark:border-slate-800 pb-3 mb-2">
                        <div class="flex flex-col pr-2">
                            <span class="text-slate-700 dark:text-slate-200 font-medium text-sm">連珠禁手規則 (Renju)</span>
                            <span class="text-[10px] text-slate-400">限制黑先手三三、四四、長連</span>
                        </div>
                        <input type="checkbox" id="renjuToggle" class="apple-native-switch" switch>
                    </div>

                    <div class="flex items-center justify-between border-b border-slate-100 dark:border-slate-800 pb-3 mb-2">
                        <span class="text-slate-700 dark:text-slate-200 font-medium text-sm">夜間模式</span>
                        <input type="checkbox" id="darkModeToggle" class="apple-native-switch" switch>
                    </div>

                    <div class="flex items-center justify-between pb-1">
                        <span class="text-slate-700 dark:text-slate-200 font-medium text-sm text-yellow-600 dark:text-yellow-400">開發者模式</span>
                        <!-- 🟡 開發者開關：專屬黃色 -->
                        <input type="checkbox" id="devModeToggle" class="apple-native-switch switch-dev" switch>
                    </div>

                    <!-- 開發者子選單：落子手數設定隱藏於此 -->
                    <div id="devPanelToggleContainer" class="hidden pl-4 opacity-0 transition-opacity duration-300 space-y-4">
                        <div class="flex items-center justify-between">
                            <span class="text-sm text-slate-500 dark:text-slate-400 font-medium">顯示懸浮面板</span>
                            <input type="checkbox" id="devPanelToggle" class="apple-native-switch" switch>
                        </div>

                        <!-- 移入開發者模式設定內之：顯示棋子落子手數開關 -->
                        <div class="flex items-center justify-between border-t border-slate-100 dark:border-slate-800 pt-3">
                            <span class="text-sm text-slate-500 dark:text-slate-400 font-medium">顯示棋子落子手數</span>
                            <input type="checkbox" id="showNumbersToggle" class="apple-native-switch" switch>
                        </div>
                        
                        <div class="flex items-center justify-between border-t border-slate-100 dark:border-slate-800 pt-3">
                            <span class="text-sm font-bold text-red-500 dark:text-red-400">🔥 外掛模式</span>
                            <!-- 🔴 外掛開關：專屬紅色 -->
                            <input type="checkbox" id="cheatModeToggle" class="apple-native-switch switch-cheat" switch>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div id="layoutSettingsModal" class="fixed inset-0 z-[75] hidden bg-slate-900/60 backdrop-blur-sm transition-opacity">
        <div class="flex items-center justify-center w-full h-full p-4 pointer-events-none">
            <div class="bg-white dark:bg-slate-900 p-6 rounded-2xl shadow-2xl max-w-sm w-full border border-slate-200 dark:border-slate-800 relative transform transition-all pointer-events-auto">
                <button id="closeLayoutBtn" class="absolute top-4 right-4 text-slate-400 hover:text-slate-600 dark:hover:text-white transition-colors">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" /></svg>
                </button>
                <h2 class="text-lg font-bold mb-6 text-slate-800 dark:text-white flex items-center gap-2">
                    版面與大小調整
                </h2>
                
                <div class="space-y-6">
                    <div class="flex flex-col gap-2">
                        <div class="flex justify-between items-center text-sm font-medium">
                            <span class="text-slate-700 dark:text-slate-200">頂部邊距</span>
                            <span id="topMarginVal" class="text-xs text-blue-500 font-bold">0px</span>
                        </div>
                        <input type="range" id="topMarginSlider" min="0" max="150" value="0" class="w-full h-1.5 bg-gray-200 dark:bg-gray-700 rounded-lg appearance-none cursor-pointer accent-blue-600">
                    </div>

                    <div class="flex flex-col gap-2">
                        <div class="flex justify-between items-center text-sm font-medium">
                            <span class="text-slate-700 dark:text-slate-200">底部邊距</span>
                            <span id="bottomMarginVal" class="text-xs text-blue-500 font-bold">0px</span>
                        </div>
                        <input type="range" id="bottomMarginSlider" min="0" max="150" value="0" class="w-full h-1.5 bg-gray-200 dark:bg-gray-700 rounded-lg appearance-none cursor-pointer accent-blue-600">
                    </div>

                    <hr class="border-slate-100 dark:border-slate-800">

                    <div class="flex flex-col gap-2">
                        <div class="flex justify-between items-center text-sm font-medium">
                            <span class="text-slate-700 dark:text-slate-200">棋盤縮放比例</span>
                            <span id="boardScaleVal" class="text-xs text-green-500 font-bold">100%</span>
                        </div>
                        <input type="range" id="boardScaleSlider" min="30" max="100" value="100" class="w-full h-1.5 bg-gray-200 dark:bg-gray-700 rounded-lg appearance-none cursor-pointer accent-green-500">
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div id="cheatConfirmModal" class="fixed inset-0 z-[80] hidden bg-slate-900/70 backdrop-blur-sm transition-opacity">
        <div class="flex items-center justify-center w-full h-full p-4 pointer-events-none">
            <div class="bg-white dark:bg-slate-900 p-6 rounded-2xl shadow-2xl max-w-sm w-full border-2 border-red-500/50 relative pointer-events-auto">
                <div class="flex justify-center mb-4 text-red-500">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-12 w-12" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" /></svg>
                </div>
                <h2 class="text-xl font-bold mb-2 text-center text-slate-800 dark:text-white">外掛警告</h2>
                <p class="text-sm text-slate-600 dark:text-slate-300 text-center mb-6">開啟外掛面板將破壞遊戲平衡及原有規則（可任意刪除或增加棋子），是否確定開啟？</p>
                <div class="flex gap-3">
                    <button id="cheatCancelBtn" class="flex-1 bg-slate-200 dark:bg-slate-700 hover:bg-slate-300 dark:hover:bg-slate-600 text-slate-800 dark:text-white font-bold py-2 px-4 rounded-xl transition">取消</button>
                    <button id="cheatConfirmBtn" class="flex-1 bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-xl transition shadow-lg shadow-red-500/30">確定開啟</button>
                </div>
            </div>
        </div>
    </div>

    <div id="gameOverModal" class="fixed inset-0 z-[90] hidden bg-slate-900/70 backdrop-blur-sm transition-opacity">
        <div class="flex items-center justify-center w-full h-full p-4 pointer-events-none">
            <div class="bg-white dark:bg-slate-900 p-8 rounded-2xl shadow-2xl text-center transform scale-95 transition-transform duration-300 max-w-[280px] w-full border border-slate-200 dark:border-slate-800 pointer-events-auto">
                <h2 id="modalTitle" class="text-2xl font-black mb-4 text-slate-800 dark:text-white">遊戲結束</h2>
                <p id="modalMsg" class="text-base mb-8 text-slate-600 dark:text-slate-400 font-medium">黑子獲勝！</p>
                <button id="modalBtn" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-4 rounded-xl transition duration-200 shadow-lg shadow-blue-500/30">
                    再來一局
                </button>
            </div>
        </div>
    </div>

    <div id="devFloatingPanel" class="hidden fixed top-[12%] left-[4%] panel-glass text-green-400 p-0 rounded-xl shadow-2xl z-[50] font-mono text-sm w-52 flex flex-col overflow-hidden border border-green-500/30">
        <div id="devPanelHeader" class="drag-handle bg-slate-800/90 p-2.5 border-b border-slate-700/50 flex justify-between items-center">
            <span class="text-white font-bold select-none text-xs tracking-wider">🛠️ 開發者工具</span>
            <span class="text-slate-400 text-[10px] select-none uppercase">Drag</span>
        </div>
        <div class="p-3 space-y-2">
            <div class="flex justify-between items-center bg-black/40 p-2 rounded border border-slate-700/50">
                <span class="text-slate-300 text-[11px]">上次耗時</span>
                <span id="thinkTimeDisp" class="text-yellow-400 font-bold text-xs">0 ms</span>
            </div>
            <button id="undoBtn" class="w-full bg-slate-700 hover:bg-slate-600 text-white text-[11px] font-bold py-2 px-3 rounded transition duration-150 shadow border border-slate-600">
                ↩ 返回上一步
            </button>
            <button id="hintBtn" class="w-full bg-indigo-600 hover:bg-indigo-500 text-white text-[11px] font-bold py-2 px-3 rounded transition duration-150 shadow border border-indigo-500">
                💡 顯示最佳落子
            </button>
        </div>
    </div>

    <div id="cheatFloatingPanel" class="hidden fixed top-[12%] right-[4%] panel-glass text-red-400 p-0 rounded-xl shadow-2xl z-[60] font-sans text-sm w-52 flex flex-col overflow-hidden border border-red-500/30">
        <div id="cheatPanelHeader" class="drag-handle bg-red-950/90 p-2.5 border-b border-red-900/50 flex justify-between items-center">
            <span class="text-white font-bold select-none text-xs tracking-wider">🔥 外掛控制面板</span>
            <span class="text-red-400 text-[10px] select-none uppercase">Drag</span>
        </div>
        <div class="p-3 grid grid-cols-1 gap-2">
            <button id="cheatBtnNormal" class="cheat-action-btn cheat-btn-active bg-slate-700 text-slate-300 text-xs font-bold py-2 px-2 rounded border border-slate-600 transition">
                ⚪ 正常落子
            </button>
            <button id="cheatBtnRemove" class="cheat-action-btn bg-slate-700 hover:bg-slate-600 text-slate-300 hover:text-white text-xs font-bold py-2 px-2 rounded border border-slate-600 transition">
                🗑️ 移除棋子
            </button>
            <button id="cheatBtnAddBlack" class="cheat-action-btn bg-slate-700 hover:bg-slate-600 text-slate-300 hover:text-white text-xs font-bold py-2 px-2 rounded border border-slate-600 transition">
                ⚫ 強制增加黑子
            </button>
            <button id="cheatBtnAddWhite" class="cheat-action-btn bg-slate-700 hover:bg-slate-600 text-slate-300 hover:text-white text-xs font-bold py-2 px-2 rounded border border-slate-600 transition">
                ⚪ 強制增加白子
            </button>
        </div>
    </div>

    <script>
        // --- 核心變數宣告 ---
        const canvas = document.getElementById('board');
        const ctx = canvas.getContext('2d');
        const BOARD_SIZE = 15;
        const BASE_SIZE = 900; 
        let cellSize = BASE_SIZE / BOARD_SIZE;
        let margin = cellSize / 2;
        let stoneRadius = cellSize * 0.43;

        let board = Array(BOARD_SIZE).fill().map(() => Array(BOARD_SIZE).fill(0));
        let curP = 1; // 1: 黑子, 2: 白子
        let active = false;
        let aiLock = false;
        let lastMove = null;
        let hoverPos = null;
        let moveHistory = [];
        let hintPos = null;
        let isDarkMode = false;
        
        let currentCheatAction = 'normal'; 
        let globalBoardScale = 1.0; 
        
        let practiceHints = []; 
        let renjuRulesActive = false; 

        // 步數、動畫新增變數
        let showStepNumbers = false; 
        let isReviewMode = false; 
        let reviewStepIndex = 0; 
        let stoneAnimations = {}; // 棋子落子微縮放動效快照

        // 固定的經典木紋風格主題，不隨系統夜間模式改變棋盤與棋子，確保高度擬真對弈體驗
        const theme = {
            boardBg: '#E6C28F',
            gridLine: '#8b5a2b',
            starColor: '#4a3018',
            blackStone: ['#5a5a5a', '#0c0c0c'],
            whiteStone: ['#ffffff', '#c3c3c3'],
            stoneShadow: 'rgba(0, 0, 0, 0.28)'
        };

        // --- DOM 選擇器 ---
        const mainGrid = document.getElementById('mainGrid');
        const boardContainer = document.getElementById('boardContainer');
        const topGroup = document.getElementById('topGroup');
        const bottomGroup = document.getElementById('bottomGroup');

        const modeSelect = document.getElementById('gameMode');
        const diffSelect = document.getElementById('aiDifficulty');
        const colorSelect = document.getElementById('aiColor');
        
        const aiDifficultyContainer = document.getElementById('aiDifficultyContainer');
        const aiColorContainer = document.getElementById('aiColorContainer');

        const startBtn = document.getElementById('startBtn');
        const practiceUndoBtn = document.getElementById('practiceUndoBtn');
        const statusDot = document.getElementById('statusDot');
        const statusText = document.getElementById('statusText');
        const aiCalculatingText = document.getElementById('aiCalculatingText');
        
        const gameOverModal = document.getElementById('gameOverModal');
        const modalTitle = document.getElementById('modalTitle');
        const modalMsg = document.getElementById('modalMsg');
        const modalBtn = document.getElementById('modalBtn');
        
        const settingsBtn = document.getElementById('settingsBtn');
        const closeSettingsBtn = document.getElementById('closeSettingsBtn');
        const settingsModal = document.getElementById('settingsModal');
        const darkModeToggle = document.getElementById('darkModeToggle');
        const renjuToggle = document.getElementById('renjuToggle'); 
        const showNumbersToggle = document.getElementById('showNumbersToggle'); 
        
        const devModeToggle = document.getElementById('devModeToggle');
        const devPanelToggleContainer = document.getElementById('devPanelToggleContainer');
        const devPanelToggle = document.getElementById('devPanelToggle');
        const devFloatingPanel = document.getElementById('devFloatingPanel');
        const devPanelHeader = document.getElementById('devPanelHeader');
        
        const cheatModeToggle = document.getElementById('cheatModeToggle');
        const cheatConfirmModal = document.getElementById('cheatConfirmModal');
        const cheatCancelBtn = document.getElementById('cheatCancelBtn');
        const cheatConfirmBtn = document.getElementById('cheatConfirmBtn');
        const cheatFloatingPanel = document.getElementById('cheatFloatingPanel');
        const cheatPanelHeader = document.getElementById('cheatPanelHeader');
        
        const thinkTimeDisp = document.getElementById('thinkTimeDisp');
        const undoBtn = document.getElementById('undoBtn');
        const hintBtn = document.getElementById('hintBtn');
        const winRateBarBlack = document.getElementById('winRateBarBlack');
        const winRateTextBlack = document.getElementById('winRateTextBlack');
        const winRateTextWhite = document.getElementById('winRateTextWhite');

        const practiceAnalysisContainer = document.getElementById('practiceAnalysisContainer');
        const realTimeAnalysisToggle = document.getElementById('realTimeAnalysisToggle');

        const layoutSettingsModal = document.getElementById('layoutSettingsModal');
        const closeLayoutBtn = document.getElementById('closeLayoutBtn');
        const topMarginSlider = document.getElementById('topMarginSlider');
        const topMarginVal = document.getElementById('topMarginVal');
        const bottomMarginSlider = document.getElementById('bottomMarginSlider');
        const bottomMarginVal = document.getElementById('bottomMarginVal');
        const boardScaleSlider = document.getElementById('boardScaleSlider');
        const boardScaleVal = document.getElementById('boardScaleVal');

        // 局後覆盤導航條 DOM
        const reviewNavigator = document.getElementById('reviewNavigator');
        const revFirstBtn = document.getElementById('revFirstBtn');
        const revPrevBtn = document.getElementById('revPrevBtn');
        const reviewCounter = document.getElementById('reviewCounter');
        const revNextBtn = document.getElementById('revNextBtn');
        const revLastBtn = document.getElementById('revLastBtn');

        // --- 拖曳模組 ---
        function makeDraggable(headerEl, panelEl) {
            let isDragging = false, currentX = 0, currentY = 0, initialX = 0, initialY = 0;
            let xOffset = 0, yOffset = 0;
            let hasBeenMoved = false;

            headerEl.addEventListener("touchstart", dragStart, { passive: false });
            headerEl.addEventListener("mousedown", dragStart);
            document.addEventListener("touchend", dragEnd);
            document.addEventListener("mouseup", dragEnd);
            document.addEventListener("touchmove", drag, { passive: false });
            document.addEventListener("mousemove", drag);

            function dragStart(e) {
                if (e.touches && e.touches.length > 1) return; 
                
                if (!hasBeenMoved) {
                    const rect = panelEl.getBoundingClientRect();
                    xOffset = rect.left;
                    yOffset = rect.top;
                    
                    panelEl.style.bottom = 'auto';
                    panelEl.style.right = 'auto';
                    panelEl.style.left = `${xOffset}px`;
                    panelEl.style.top = `${yOffset}px`;
                    hasBeenMoved = true;
                }

                if (e.target === headerEl || headerEl.contains(e.target)) {
                    initialX = (e.type === "touchstart" ? e.touches[0].clientX : e.clientX) - xOffset;
                    initialY = (e.type === "touchstart" ? e.touches[0].clientY : e.clientY) - yOffset;
                    isDragging = true;
                    e.preventDefault(); 
                }
            }

            function dragEnd() { initialX = currentX; initialY = currentY; isDragging = false; }

            function drag(e) {
                if (!isDragging) return;
                e.preventDefault();
                currentX = (e.type === "touchmove" ? e.touches[0].clientX : e.clientX) - initialX;
                currentY = (e.type === "touchmove" ? e.touches[0].clientY : e.clientY) - initialY;
                
                const panelRect = panelEl.getBoundingClientRect();
                if (currentX < 0) currentX = 0;
                if (currentX + panelRect.width > window.innerWidth) currentX = window.innerWidth - panelRect.width;
                if (currentY < 0) currentY = 0;
                if (currentY + panelRect.height > window.innerHeight) currentY = window.innerHeight - panelRect.height;
                
                xOffset = currentX; yOffset = currentY;
                panelEl.style.left = `${currentX}px`; panelEl.style.top = `${currentY}px`;
            }

            window.addEventListener('resize', () => {
                if (hasBeenMoved && !panelEl.classList.contains('hidden')) {
                    const panelRect = panelEl.getBoundingClientRect();
                    if (xOffset + panelRect.width > window.innerWidth) xOffset = window.innerWidth - panelRect.width;
                    if (yOffset + panelRect.height > window.innerHeight) yOffset = window.innerHeight - panelRect.height;
                    if (xOffset < 0) xOffset = 0; if (yOffset < 0) yOffset = 0;
                    panelEl.style.left = `${xOffset}px`; panelEl.style.top = `${yOffset}px`;
                }
            });
        }
        
        makeDraggable(devPanelHeader, devFloatingPanel); 
        makeDraggable(cheatPanelHeader, cheatFloatingPanel); 

        // --- 版面設定邏輯 ---
        function closeLayoutSettings() {
            layoutSettingsModal.classList.add('hidden');
        }
        closeLayoutBtn.addEventListener('click', closeLayoutSettings);
        layoutSettingsModal.addEventListener('click', (e) => {
            if(e.target === layoutSettingsModal) closeLayoutSettings();
        });

        topMarginSlider.addEventListener('input', (e) => {
            const val = e.target.value;
            topMarginVal.innerText = `${val}px`;
            document.documentElement.style.setProperty('--top-spacing', `${val}px`);
            resizeBoardContainer();
        });

        bottomMarginSlider.addEventListener('input', (e) => {
            const val = e.target.value;
            bottomMarginVal.innerText = `${val}px`;
            document.documentElement.style.setProperty('--bottom-spacing', `${val}px`);
            resizeBoardContainer();
        });

        boardScaleSlider.addEventListener('input', (e) => {
            const val = e.target.value;
            boardScaleVal.innerText = `${val}%`;
            globalBoardScale = parseInt(val) / 100;
            resizeBoardContainer();
        });

        // --- 兩指長按手勢 ---
        let gestureTimer = null;
        let isGestureActive = false;
        let gestureTouchStartPoints = [];

        document.addEventListener('touchstart', (e) => {
            if (e.touches.length === 2) {
                e.preventDefault(); 
                startGestureTimer(e);
            } else if (e.touches.length > 2) {
                cancelGestureTimer();
            }
        }, { passive: false });

        document.addEventListener('touchmove', (e) => {
            if (isGestureActive) {
                e.preventDefault(); 
                if (e.touches.length === 2 && gestureTouchStartPoints.length === 2) {
                    for (let i = 0; i < 2; i++) {
                        const dx = e.touches[i].clientX - gestureTouchStartPoints[i].x;
                        const dy = e.touches[i].clientY - gestureTouchStartPoints[i].y;
                        const distance = Math.sqrt(dx * dx + dy * dy);
                        if (distance > 30) { 
                            cancelGestureTimer();
                            break;
                        }
                    }
                }
            }
        }, { passive: false });

        document.addEventListener('touchend', (e) => {
            if (e.touches.length < 2) {
                cancelGestureTimer();
            }
        });

        function startGestureTimer(e) {
            if (isGestureActive) return;
            isGestureActive = true;
            
            gestureTouchStartPoints = [
                { x: e.touches[0].clientX, y: e.touches[0].clientY },
                { x: e.touches[1].clientX, y: e.touches[1].clientY }
            ];

            gestureTimer = setTimeout(() => {
                triggerLayoutSettingsModal();
            }, 3000);
        }

        function cancelGestureTimer() {
            if (!isGestureActive) return;
            clearTimeout(gestureTimer);
            gestureTimer = null;
            isGestureActive = false;
            gestureTouchStartPoints = [];
        }

        function triggerLayoutSettingsModal() {
            clearTimeout(gestureTimer);
            gestureTimer = null;
            isGestureActive = false;
            gestureTouchStartPoints = [];

            if (navigator.vibrate) {
                navigator.vibrate(100);
            }

            layoutSettingsModal.classList.remove('hidden');
        }

        // --- 棋盤自適應計算 ---
        function resizeBoardContainer() {
            if (!mainGrid || !boardContainer) return;
            const availW = mainGrid.clientWidth;
            const availH = mainGrid.clientHeight;
            const optimalSize = Math.max(100, Math.min(availW, availH)) * globalBoardScale;
            boardContainer.style.width = `${optimalSize}px`;
            boardContainer.style.height = `${optimalSize}px`;
        }

        window.addEventListener('resize', () => {
            resizeBoardContainer();
            setupCanvasDPI();
        });

        // --- 🌓 智慧偵測手機系統夜間模式並綁定 ---
        function setupAutoDarkMode() {
            const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
            
            // 首次加載：偵測目前系統狀態並套用
            isDarkMode = mediaQuery.matches;
            darkModeToggle.checked = isDarkMode;
            applyDarkModeClass();

            // 監聽系統狀態動態切換
            mediaQuery.addEventListener('change', (e) => {
                isDarkMode = e.matches;
                darkModeToggle.checked = isDarkMode;
                applyDarkModeClass();
                requestAnimationFrame(drawBoard);
            });
        }

        function applyDarkModeClass() {
            if (isDarkMode) document.documentElement.classList.add('dark');
            else document.documentElement.classList.remove('dark');
        }

        // --- 一般設定事件 ---
        settingsBtn.addEventListener('click', () => settingsModal.classList.remove('hidden'));
        closeSettingsBtn.addEventListener('click', () => settingsModal.classList.add('hidden'));
        settingsModal.addEventListener('click', (e) => {
            if(e.target === settingsModal) settingsModal.classList.add('hidden');
        });

        // 手動點選 Toggle 時的邏輯
        darkModeToggle.addEventListener('change', (e) => {
            isDarkMode = e.target.checked;
            applyDarkModeClass();
            requestAnimationFrame(drawBoard);
        });

        renjuToggle.addEventListener('change', (e) => {
            renjuRulesActive = e.target.checked;
            requestAnimationFrame(drawBoard);
        });

        showNumbersToggle.addEventListener('change', (e) => {
            showStepNumbers = e.target.checked;
            requestAnimationFrame(drawBoard);
        });

        devModeToggle.addEventListener('change', (e) => {
            if(e.target.checked) {
                devPanelToggleContainer.classList.remove('hidden');
                setTimeout(() => devPanelToggleContainer.classList.remove('opacity-0'), 10);
            } else {
                devPanelToggleContainer.classList.add('opacity-0');
                setTimeout(() => devPanelToggleContainer.classList.add('hidden'), 300);
                devPanelToggle.checked = false; devFloatingPanel.classList.add('hidden'); hintPos = null;
                cheatModeToggle.checked = false; cheatFloatingPanel.classList.add('hidden'); currentCheatAction = 'normal'; updateCheatBtns();
                showNumbersToggle.checked = false; showStepNumbers = false; // 退出開發者模式時自動重設手數功能
                requestAnimationFrame(drawBoard);
            }
        });

        devPanelToggle.addEventListener('change', (e) => {
            if(e.target.checked) devFloatingPanel.classList.remove('hidden');
            else { devFloatingPanel.classList.add('hidden'); hintPos = null; requestAnimationFrame(drawBoard); }
        });

        cheatModeToggle.addEventListener('change', (e) => {
            if(e.target.checked) {
                cheatConfirmModal.classList.remove('hidden');
            } else {
                cheatFloatingPanel.classList.add('hidden');
                currentCheatAction = 'normal';
                updateCheatBtns();
                requestAnimationFrame(drawBoard);
            }
        });

        cheatCancelBtn.addEventListener('click', () => {
            cheatConfirmModal.classList.add('hidden');
            cheatModeToggle.checked = false;
        });

        cheatConfirmBtn.addEventListener('click', () => {
            cheatConfirmModal.classList.add('hidden');
            cheatFloatingPanel.classList.remove('hidden');
        });

        const cheatBtns = [
            { id: 'cheatBtnNormal', action: 'normal' },
            { id: 'cheatBtnRemove', action: 'remove' },
            { id: 'cheatBtnAddBlack', action: 'add_black' },
            { id: 'cheatBtnAddWhite', action: 'add_white' }
        ];

        cheatBtns.forEach(btnInfo => {
            document.getElementById(btnInfo.id).addEventListener('click', () => {
                currentCheatAction = btnInfo.action;
                updateCheatBtns();
                requestAnimationFrame(drawBoard);
            });
        });

        function updateCheatBtns() {
            cheatBtns.forEach(btnInfo => {
                const el = document.getElementById(btnInfo.id);
                if (currentCheatAction === btnInfo.action) {
                    el.classList.add('cheat-btn-active');
                    el.classList.remove('bg-slate-700', 'text-slate-300', 'hover:bg-slate-600', 'hover:text-white');
                } else {
                    el.classList.remove('cheat-btn-active');
                    el.classList.add('bg-slate-700', 'text-slate-300', 'hover:bg-slate-600', 'hover:text-white');
                }
            });
        }

        function setupCanvasDPI() {
            canvas.width = BASE_SIZE; canvas.height = BASE_SIZE; drawBoard();
        }

        // 模式切換
        modeSelect.addEventListener('change', () => {
            const mode = modeSelect.value;
            practiceHints = [];
            updateSelectorsInteractable();
            
            if (mode === 'practice') {
                practiceAnalysisContainer.classList.remove('hidden');
                if (realTimeAnalysisToggle.checked) {
                    calculatePracticeAnalysis();
                }
            } else {
                practiceAnalysisContainer.classList.add('hidden');
            }
            requestAnimationFrame(drawBoard);
        });

        realTimeAnalysisToggle.addEventListener('change', (e) => {
            if (e.target.checked && modeSelect.value === 'practice') {
                calculatePracticeAnalysis();
            } else {
                practiceHints = [];
                requestAnimationFrame(drawBoard);
            }
        });

        function calculatePracticeAnalysis() {
            if (modeSelect.value !== 'practice' || !realTimeAnalysisToggle.checked) return;
            
            let stones = 0;
            for(let r=0; r<BOARD_SIZE; r++) {
                for(let c=0; c<BOARD_SIZE; c++) {
                    if(board[r][c] !== 0) stones++;
                }
            }
            
            if (stones === 0) {
                practiceHints = [{ r: 7, c: 7, score: 9999 }];
                requestAnimationFrame(drawBoard);
            } else {
                requestAnimationFrame(() => {
                    const opp = curP === 1 ? 2 : 1;
                    practiceHints = getTopCandidates(board, curP, opp, 3);
                    requestAnimationFrame(drawBoard);
                });
            }
        }

        function updateSelectorsInteractable() {
            const isGameStarted = active;
            const mode = modeSelect.value;

            if (mode === 'pve') {
                aiDifficultyContainer.classList.remove('hidden');
                aiColorContainer.classList.remove('hidden');
                practiceUndoBtn.classList.add('hidden'); 
            } else if (mode === 'practice') {
                aiDifficultyContainer.classList.add('hidden');
                aiColorContainer.classList.add('hidden');
                practiceUndoBtn.classList.remove('hidden'); 
            } else { 
                aiDifficultyContainer.classList.add('hidden');
                aiColorContainer.classList.add('hidden');
                practiceUndoBtn.classList.add('hidden'); 
            }

            if (isGameStarted) {
                modeSelect.disabled = true;
                diffSelect.disabled = true;
                colorSelect.disabled = true;
                modeSelect.classList.add('opacity-50', 'cursor-not-allowed');
                diffSelect.classList.add('opacity-50', 'cursor-not-allowed');
                colorSelect.classList.add('opacity-50', 'cursor-not-allowed');
            } else {
                modeSelect.disabled = false;
                modeSelect.classList.remove('opacity-50', 'cursor-not-allowed');
                if (mode === 'pve') {
                    diffSelect.disabled = false;
                    colorSelect.disabled = false;
                    diffSelect.classList.remove('opacity-50', 'cursor-not-allowed');
                    colorSelect.classList.remove('opacity-50', 'cursor-not-allowed');
                } else {
                    diffSelect.disabled = true;
                    colorSelect.disabled = true;
                }
            }
        }

        // --- 🚫 連珠禁手規則 (3-3, 4-4, 長連) ---
        function checkRenjuForbidden(boardState, r, c) {
            if (boardState[r][c] !== 0) return false;
            boardState[r][c] = 1;

            let isOverline = false;
            let liveThreeCount = 0;
            let fourCount = 0;

            const dirs = [[1,0], [0,1], [1,1], [1,-1]];
            for (let [dr, dc] of dirs) {
                let consecutive = 1;
                let i = 1;
                while (r + dr * i >= 0 && r + dr * i < BOARD_SIZE && c + dc * i >= 0 && c + dc * i < BOARD_SIZE && boardState[r + dr * i][c + dc * i] === 1) { consecutive++; i++; }
                i = 1;
                while (r - dr * i >= 0 && r - dr * i < BOARD_SIZE && c - dc * i >= 0 && c - dc * i < BOARD_SIZE && boardState[r - dr * i][c - dc * i] === 1) { consecutive++; i++; }
                if (consecutive >= 6) {
                    isOverline = true;
                }

                let segment = [];
                for (let step = -5; step <= 5; step++) {
                    let nr = r + dr * step;
                    let nc = c + dc * step;
                    if (nr < 0 || nr >= BOARD_SIZE || nc < 0 || nc >= BOARD_SIZE) {
                        segment.push(2); 
                    } else {
                        segment.push(boardState[nr][nc]);
                    }
                }

                let formsFour = false;
                let formsLiveThree = false;

                for (let j = 0; j < segment.length; j++) {
                    if (segment[j] === 0) {
                        segment[j] = 1;
                        if (hasExactlyFive(segment)) {
                            formsFour = true;
                        }
                        segment[j] = 0;
                    }
                }

                for (let j = 0; j < segment.length; j++) {
                    if (segment[j] === 0) {
                        segment[j] = 1;
                        if (hasOpenFour(segment)) {
                            formsLiveThree = true;
                        }
                        segment[j] = 0;
                    }
                }

                if (formsFour) fourCount++;
                if (formsLiveThree) liveThreeCount++;
            }

            boardState[r][c] = 0;

            if (isOverline) return "長連";
            if (liveThreeCount >= 2) return "三三";
            if (fourCount >= 2) return "四四";

            return false;
        }

        function hasExactlyFive(arr) {
            let maxConsec = 0;
            let current = 0;
            for (let val of arr) {
                if (val === 1) {
                    current++;
                    if (current > maxConsec) maxConsec = current;
                } else {
                    current = 0;
                }
            }
            return maxConsec === 5;
        }

        function hasOpenFour(arr) {
            for (let i = 0; i <= arr.length - 6; i++) {
                if (arr[i] === 0 && arr[i+1] === 1 && arr[i+2] === 1 && arr[i+3] === 1 && arr[i+4] === 1 && arr[i+5] === 0) {
                    return true;
                }
            }
            return false;
        }

        // --- 勝率評估 ---
        function updateWinRateBar(forcedWinRate = null) {
            let winRate = 50;
            if (forcedWinRate !== null) {
                winRate = forcedWinRate;
            } else {
                let bMax = getPureScore(board, 1);
                let wMax = getPureScore(board, 2);
                if (curP === 1 && bMax >= 10000000) winRate = 99.5;
                else if (curP === 2 && wMax >= 10000000) winRate = 0.5;
                else if (curP === 2 && bMax >= 10000000) winRate = (wMax >= 10000000) ? 0.5 : 88.0;
                else if (curP === 1 && wMax >= 10000000) winRate = (bMax >= 10000000) ? 99.5 : 12.0;
                else {
                    let bPower = bMax > 0 ? Math.log10(bMax) : 0;
                    let wPower = wMax > 0 ? Math.log10(wMax) : 0;
                    if (curP === 1) bPower += 0.35; else wPower += 0.35;
                    let powerDiff = bPower - wPower;
                    let sigmoid = 1 / (1 + Math.exp(-0.85 * powerDiff));
                    winRate = sigmoid * 100;
                }
            }
            winRate = Math.max(0.1, Math.min(99.9, winRate));
            
            winRateBarBlack.style.width = `${winRate}%`;
            winRateTextBlack.innerText = `黑 ${winRate.toFixed(1)}%`;
            winRateTextWhite.innerText = `白 ${(100 - winRate).toFixed(1)}%`;

            if (winRate >= 54) {
                winRateTextBlack.classList.add('text-blue-500', 'dark:text-blue-400');
                winRateTextWhite.classList.remove('text-blue-500', 'dark:text-blue-400');
            } else if (winRate <= 46) {
                winRateTextWhite.classList.add('text-blue-500', 'dark:text-blue-400');
                winRateTextBlack.classList.remove('text-blue-500', 'dark:text-blue-400');
            } else {
                winRateTextBlack.classList.remove('text-blue-500', 'dark:text-blue-400');
                winRateTextWhite.classList.remove('text-blue-500', 'dark:text-blue-400');
            }
        }

        // --- 核心修改：狀態管理生命週期 ---
        function updateStartBtn() {
            if (active) {
                startBtn.innerText = "結束遊戲";
                startBtn.className = "flex-1 min-w-[90px] bg-red-600 hover:bg-red-700 text-white text-xs sm:text-sm font-bold py-2.5 px-4 rounded-lg transition duration-200 shadow-md whitespace-nowrap active:scale-95";
            } else {
                startBtn.innerText = "開始遊戲";
                startBtn.className = "flex-1 min-w-[90px] bg-blue-600 hover:bg-blue-700 text-white text-xs sm:text-sm font-bold py-2.5 px-4 rounded-lg transition duration-200 shadow-md whitespace-nowrap active:scale-95";
            }
        }

        // 開始對局
        function startGame() {
            board = Array.from({ length: BOARD_SIZE }, () => Array(BOARD_SIZE).fill(0));
            moveHistory = [];
            curP = 1; 
            active = true;
            aiLock = false;
            lastMove = null;
            hoverPos = null;
            hintPos = null;
            practiceHints = [];
            stoneAnimations = {};
            
            // 隱藏覆盤導航條
            isReviewMode = false;
            reviewStepIndex = 0;
            reviewNavigator.classList.add('hidden');

            thinkTimeDisp.innerText = "0 ms";
            gameOverModal.classList.add('hidden');
            settingsModal.classList.add('hidden');
            layoutSettingsModal.classList.add('hidden');
            
            updateStatus();
            updateWinRateBar();
            updateSelectorsInteractable(); 
            updateStartBtn();
            resizeBoardContainer();
            setupCanvasDPI();

            const mode = modeSelect.value;
            if (mode === 'pve' && parseInt(colorSelect.value) === 1) triggerAI();
            else if (mode === 'practice' && realTimeAnalysisToggle.checked) {
                calculatePracticeAnalysis();
            }
        }

        // 結束並還原對局
        function stopGame() {
            board = Array.from({ length: BOARD_SIZE }, () => Array(BOARD_SIZE).fill(0));
            moveHistory = [];
            curP = 1; 
            active = false;
            aiLock = false;
            lastMove = null;
            hoverPos = null;
            hintPos = null;
            practiceHints = [];
            stoneAnimations = {};
            isReviewMode = false;
            reviewStepIndex = 0;
            reviewNavigator.classList.add('hidden');

            thinkTimeDisp.innerText = "0 ms";
            gameOverModal.classList.add('hidden');
            
            updateStatus();
            updateWinRateBar();
            updateSelectorsInteractable(); 
            updateStartBtn();
            drawBoard();
        }

        startBtn.addEventListener('click', () => {
            if (active) stopGame();
            else startGame();
        });
        modalBtn.addEventListener('click', startGame);

        // --- 棋盤與極致 3D 質感棋子渲染 ---
        function drawBoard() {
            ctx.clearRect(0, 0, BASE_SIZE, BASE_SIZE);
            
            const boardBg = theme.boardBg;
            const gridColor = theme.gridLine;
            const starColor = theme.starColor;

            ctx.fillStyle = boardBg;
            ctx.fillRect(0, 0, BASE_SIZE, BASE_SIZE);

            ctx.beginPath();
            ctx.strokeStyle = gridColor;
            ctx.lineWidth = 2.5;
            for (let i = 0; i < BOARD_SIZE; i++) {
                ctx.moveTo(margin, margin + i * cellSize);
                ctx.lineTo(BASE_SIZE - margin, margin + i * cellSize);
                ctx.moveTo(margin + i * cellSize, margin);
                ctx.lineTo(margin + i * cellSize, BASE_SIZE - margin);
            }
            ctx.stroke();

            const stars = [[3, 3], [11, 3], [3, 11], [11, 11], [7, 7]];
            ctx.fillStyle = starColor;
            stars.forEach(([r, c]) => {
                ctx.beginPath();
                ctx.arc(margin + c * cellSize, margin + r * cellSize, 5.5, 0, Math.PI * 2);
                ctx.fill();
            });

            // 根據覆盤狀態，確認當前需要繪製的棋子子集
            const stepsToDraw = isReviewMode ? moveHistory.slice(0, reviewStepIndex) : moveHistory;
            const tempBoard = Array(BOARD_SIZE).fill().map(() => Array(BOARD_SIZE).fill(0));
            stepsToDraw.forEach(m => { tempBoard[m.r][m.c] = m.player; });

            // 繪製棋子
            stepsToDraw.forEach((move, index) => {
                drawStone(move.r, move.c, move.player, 1.0, index + 1);
            });

            if (modeSelect.value === 'practice' && realTimeAnalysisToggle.checked && practiceHints.length > 0 && !isReviewMode) {
                const colors = ['#22c55e', '#a3e635', '#eab308']; 
                const rings = ['①', '②', '③'];
                
                practiceHints.forEach((hint, index) => {
                    if (board[hint.r][hint.c] === 0) {
                        const cx = margin + hint.c * cellSize;
                        const cy = margin + hint.r * cellSize;
                        
                        ctx.beginPath();
                        ctx.strokeStyle = colors[index];
                        ctx.lineWidth = index === 0 ? 5 : 3;
                        ctx.arc(cx, cy, stoneRadius * 0.5, 0, Math.PI * 2);
                        ctx.stroke();
                        
                        ctx.fillStyle = colors[index];
                        ctx.font = `bold 24px -apple-system, sans-serif`;
                        ctx.textAlign = 'center';
                        ctx.textBaseline = 'middle';
                        ctx.fillText(rings[index], cx, cy);
                    }
                });
            }

            if (hoverPos && active && !aiLock && !isGestureActive && !isReviewMode) {
                if (currentCheatAction === 'remove') {
                    if (board[hoverPos.r][hoverPos.c] !== 0) drawCross(hoverPos.r, hoverPos.c);
                } else if (currentCheatAction === 'add_black') {
                    if (board[hoverPos.r][hoverPos.c] === 0) drawStone(hoverPos.r, hoverPos.c, 1, 0.4);
                } else if (currentCheatAction === 'add_white') {
                    if (board[hoverPos.r][hoverPos.c] === 0) drawStone(hoverPos.r, hoverPos.c, 2, 0.4);
                } else {
                    if (board[hoverPos.r][hoverPos.c] === 0) {
                        if (curP === 1 && renjuRulesActive) {
                            const forbiddenType = checkRenjuForbidden(board, hoverPos.r, hoverPos.c);
                            if (forbiddenType) {
                                drawStone(hoverPos.r, hoverPos.c, 1, 0.25);
                                drawForbiddenWarning(hoverPos.r, hoverPos.c, forbiddenType);
                            } else {
                                drawStone(hoverPos.r, hoverPos.c, curP, 0.4);
                            }
                        } else {
                            drawStone(hoverPos.r, hoverPos.c, curP, 0.4);
                        }
                    }
                }
            }

            const currentLastMove = isReviewMode ? (stepsToDraw.length > 0 ? stepsToDraw[stepsToDraw.length-1] : null) : lastMove;
            if (currentLastMove) {
                ctx.beginPath();
                ctx.fillStyle = tempBoard[currentLastMove.r][currentLastMove.c] === 1 ? '#fff' : '#ef4444';
                ctx.arc(margin + currentLastMove.c * cellSize, margin + currentLastMove.r * cellSize, 4, 0, Math.PI * 2);
                ctx.fill();
            }

            if (hintPos && board[hintPos.r][hintPos.c] === 0 && !isReviewMode) {
                const cx = margin + hintPos.c * cellSize;
                const cy = margin + hintPos.r * cellSize;
                ctx.beginPath(); ctx.strokeStyle = '#3b82f6'; ctx.lineWidth = 4;
                ctx.arc(cx, cy, stoneRadius * 0.6, 0, Math.PI * 2); ctx.stroke();
            }
        }

        // 具備縮放 Popper 動效與 3D spec 高光邊緣環境反光的棋子渲染器
        function drawStone(r, c, player, alpha = 1.0, stepNum = null) {
            const cx = margin + c * cellSize;
            const cy = margin + r * cellSize;
            
            // 彈出動態縮放比計算
            let scale = 1.0;
            if (alpha === 1.0 && !isReviewMode) {
                const animId = `${r}_${c}`;
                if (!stoneAnimations[animId]) {
                    stoneAnimations[animId] = { start: performance.now(), duration: 180 };
                }
                const progress = (performance.now() - stoneAnimations[animId].start) / stoneAnimations[animId].duration;
                if (progress < 1.0) {
                    scale = Math.sin(progress * Math.PI) * 0.15 + 1.0;
                    requestAnimationFrame(drawBoard); 
                } else {
                    scale = 1.0;
                }
            }

            const currentRadius = stoneRadius * scale;

            // 1. 3D 立體深投影
            if (alpha === 1.0) {
                ctx.beginPath();
                ctx.arc(cx + 4, cy + 5, currentRadius, 0, Math.PI * 2);
                ctx.fillStyle = theme.stoneShadow;
                ctx.fill();
            }

            // 2. 碁石本體渲染
            ctx.beginPath();
            ctx.arc(cx, cy, currentRadius, 0, Math.PI * 2);
            
            // 棋子球體漸層 (徑向漸層，光源位於左上角偏上)
            let gradient = ctx.createRadialGradient(
                cx - currentRadius * 0.35, 
                cy - currentRadius * 0.35, 
                currentRadius * 0.05, 
                cx, 
                cy, 
                currentRadius
            );

            const pColors = player === 1 ? theme.blackStone : theme.whiteStone;
            gradient.addColorStop(0, hexToRgba(pColors[0], alpha));
            gradient.addColorStop(1, hexToRgba(pColors[1], alpha));
            ctx.fillStyle = gradient; 
            ctx.fill();

            // 3. Specular 高光與邊緣環境描邊 (只適用於 100% 實體子)
            if (alpha === 1.0) {
                // 3D 邊緣環境反光圈
                ctx.beginPath();
                ctx.strokeStyle = player === 1 ? 'rgba(255, 255, 255, 0.06)' : 'rgba(0, 0, 0, 0.08)';
                ctx.lineWidth = 1.5;
                ctx.arc(cx, cy, currentRadius - 0.7, 0, Math.PI * 2);
                ctx.stroke();

                // 左上高光點
                ctx.beginPath();
                ctx.ellipse(
                    cx - currentRadius * 0.3, 
                    cy - currentRadius * 0.3, 
                    currentRadius * 0.22, 
                    currentRadius * 0.11, 
                    -Math.PI / 4, 
                    0, 
                    Math.PI * 2
                );
                ctx.fillStyle = player === 1 ? 'rgba(255, 255, 255, 0.22)' : 'rgba(255, 255, 255, 0.9)';
                ctx.fill();
            }

            // 4. 實數標記 (可選)
            if (showStepNumbers && stepNum !== null && alpha === 1.0) {
                ctx.font = `bold ${Math.floor(cellSize * 0.36)}px -apple-system, sans-serif`;
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillStyle = player === 1 ? 'rgba(255, 255, 255, 0.85)' : 'rgba(80, 80, 80, 0.9)';
                ctx.fillText(stepNum, cx, cy);
            }
        }

        function hexToRgba(hex, alpha) {
            let c;
            if(/^#([A-Fa-f0-9]{3}){1,2}$/.test(hex)){
                c= hex.substring(1).split('');
                if(c.length== 3){
                    c= [c[0], c[0], c[1], c[1], c[2], c[2]];
                }
                c= '0x'+c.join('');
                return 'rgba('+[(c>>16)&255, (c>>8)&255, c&255].join(',')+','+alpha+')';
            }
            return hex;
        }

        // 繪製叉叉 (用於外掛移除棋子狀態)
        function drawCross(r, c) {
            const cx = margin + c * cellSize;
            const cy = margin + r * cellSize;
            const size = stoneRadius * 0.6;
            ctx.beginPath();
            ctx.strokeStyle = 'rgba(239, 68, 68, 0.8)'; 
            ctx.lineWidth = 6;
            ctx.moveTo(cx - size, cy - size); ctx.lineTo(cx + size, cy + size);
            ctx.moveTo(cx + size, cy - size); ctx.lineTo(cx - size, cy + size);
            ctx.stroke();
        }

        // 繪製禁手警告資訊
        function drawForbiddenWarning(r, c, typeText) {
            const cx = margin + c * cellSize;
            const cy = margin + r * cellSize;
            
            ctx.beginPath();
            ctx.strokeStyle = '#ef4444';
            ctx.lineWidth = 4;
            const size = stoneRadius * 0.4;
            ctx.moveTo(cx - size, cy - size); ctx.lineTo(cx + size, cy + size);
            ctx.moveTo(cx + size, cy - size); ctx.lineTo(cx - size, cy + size);
            ctx.stroke();

            ctx.fillStyle = '#ef4444';
            ctx.font = 'bold 15px -apple-system, sans-serif';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText(typeText, cx, cy + stoneRadius * 0.7);
        }

        function getMousePos(evt) {
            const rect = canvas.getBoundingClientRect();
            const scaleX = BASE_SIZE / rect.width;
            const scaleY = BASE_SIZE / rect.height;
            let clientX, clientY;
            if (evt.touches && evt.touches.length > 0) { clientX = evt.touches[0].clientX; clientY = evt.touches[0].clientY; } 
            else { clientX = evt.clientX; clientY = evt.clientY; }
            const x = (clientX - rect.left) * scaleX;
            const y = (clientY - rect.top) * scaleY;
            const c = Math.round((x - margin) / cellSize);
            const r = Math.round((y - margin) / cellSize);
            if (r >= 0 && r < BOARD_SIZE && c >= 0 && c < BOARD_SIZE) return { r, c };
            return null;
        }

        canvas.addEventListener('mousemove', handleHover);
        canvas.addEventListener('mouseout', () => { hoverPos = null; if (active) requestAnimationFrame(drawBoard); });
        canvas.addEventListener('click', handleClick);
        
        canvas.addEventListener('touchstart', (e) => { 
            if(e.touches.length > 1) return; 
            e.preventDefault(); 
            handleHover(e); 
        }, { passive: false });

        canvas.addEventListener('touchend', (e) => { 
            if(isGestureActive) return; 
            e.preventDefault(); 
            handleClick(e); 
        }, { passive: false });
        
        function handleHover(e) {
            if (!active || aiLock || isGestureActive) return;
            const pos = getMousePos(e);
            if (pos) {
                if (!hoverPos || hoverPos.r !== pos.r || hoverPos.c !== pos.c) {
                    hoverPos = pos;
                    requestAnimationFrame(drawBoard);
                }
            } else if (hoverPos) {
                hoverPos = null;
                requestAnimationFrame(drawBoard);
            }
        }

        function handleClick(e) {
            if (!active || aiLock || isGestureActive || isReviewMode) return;
            const pos = hoverPos || getMousePos(e);
            if (!pos) return;

            // 外掛模式落子
            if (currentCheatAction !== 'normal') {
                if (currentCheatAction === 'remove') {
                    if (board[pos.r][pos.c] !== 0) {
                        board[pos.r][pos.c] = 0;
                        moveHistory = moveHistory.filter(m => !(m.r === pos.r && m.c === pos.c)); 
                        if (lastMove && lastMove.r === pos.r && lastMove.c === pos.c) lastMove = null;
                        drawBoard(); updateWinRateBar();
                        if (modeSelect.value === 'practice' && realTimeAnalysisToggle.checked) calculatePracticeAnalysis();
                    }
                } else if (currentCheatAction === 'add_black' || currentCheatAction === 'add_white') {
                    if (board[pos.r][pos.c] === 0) {
                        let p = currentCheatAction === 'add_black' ? 1 : 2;
                        
                        if (p === 1 && renjuRulesActive) {
                            const forbiddenType = checkRenjuForbidden(board, pos.r, pos.c);
                            if (forbiddenType) {
                                board[pos.r][pos.c] = 1;
                                lastMove = { r: pos.r, c: pos.c };
                                moveHistory.push({ r: pos.r, c: pos.c, player: p });
                                drawBoard();
                                endGame(2); 
                                return;
                            }
                        }

                        board[pos.r][pos.c] = p;
                        lastMove = { r: pos.r, c: pos.c };
                        moveHistory.push({ r: pos.r, c: pos.c, player: p });
                        
                        drawBoard(); updateWinRateBar();
                        if (checkWin(pos.r, pos.c, p)) endGame(p);
                        else if (checkDraw()) endGame(0);
                        else if (modeSelect.value === 'practice' && realTimeAnalysisToggle.checked) calculatePracticeAnalysis();
                    }
                }
                return; 
            }

            // 正常落子
            if (board[pos.r][pos.c] === 0) {
                if (curP === 1 && renjuRulesActive) {
                    const forbiddenType = checkRenjuForbidden(board, pos.r, pos.c);
                    if (forbiddenType) {
                        return;
                    }
                }
                
                // 震動反饋
                if (navigator.vibrate) {
                    navigator.vibrate(10);
                }
                makeMove(pos.r, pos.c);
            }
        }

        function makeMove(r, c) {
            board[r][c] = curP;
            moveHistory.push({ r, c, player: curP });
            lastMove = { r, c };
            hoverPos = null;
            hintPos = null;
            drawBoard(); 

            if (checkWin(r, c, curP)) { endGame(curP); return; }
            if (checkDraw()) { endGame(0); return; }

            curP = curP === 1 ? 2 : 1;
            updateStatus();
            updateWinRateBar();
            updateSelectorsInteractable(); 

            const mode = modeSelect.value;
            if (mode === 'pve') {
                const aiCol = parseInt(colorSelect.value);
                if (curP === aiCol) triggerAI();
            } else if (mode === 'practice' && realTimeAnalysisToggle.checked) {
                calculatePracticeAnalysis();
            }
        }

        function triggerAI() {
            aiLock = true;
            aiCalculatingText.classList.remove('hidden');
            requestAnimationFrame(() => {
                requestAnimationFrame(() => {
                    const difficulty = parseInt(diffSelect.value);
                    const t0 = performance.now();
                    try { playAI(difficulty); } 
                    catch (err) { console.error("AI計算錯誤:", err); } 
                    finally {
                        const t1 = performance.now();
                        thinkTimeDisp.innerText = `${(t1 - t0).toFixed(1)} ms`;
                        aiCalculatingText.classList.add('hidden');
                        aiLock = false;
                        updateSelectorsInteractable(); 
                    }
                });
            });
        }

        function updateStatus() {
            const colorName = curP === 1 ? '黑子' : '白子';
            statusDot.className = `w-3.5 h-3.5 rounded-full inline-block shadow-md ring-1 ring-gray-400 ${curP === 1 ? 'bg-black' : 'bg-white'}`;
            statusText.innerText = `輪到：${colorName}`;
            statusText.className = curP === 1 ? 'text-slate-800 dark:text-slate-100' : 'text-slate-500 dark:text-slate-400';
        }

        function endGame(winner) {
            active = false;
            hintPos = null;
            practiceHints = [];
            
            if (winner === 0) {
                modalTitle.innerText = "平局";
                modalMsg.innerText = "棋盤已滿，雙方平手！";
                updateWinRateBar(50);
            } else {
                modalTitle.innerText = "對局結束";
                modalMsg.innerText = winner === 1 ? "恭喜！黑子獲勝！" : "恭喜！白子獲勝！";
                updateWinRateBar(winner === 1 ? 100 : 0);
            }
            updateSelectorsInteractable(); 
            updateStartBtn();

            // 啟用局後覆盤模式
            initReviewMode();

            setTimeout(() => gameOverModal.classList.remove('hidden'), 300);
        }

        // 對長連判定規則的嚴格遵守（黑長連禁手算輸，白長連合法獲勝）
        function checkWin(r, c, player) {
            const dirs = [[1, 0], [0, 1], [1, 1], [1, -1]];
            for (let [dr, dc] of dirs) {
                let count = 1, i = 1;
                while (r + dr * i >= 0 && r + dr * i < BOARD_SIZE && c + dc * i >= 0 && c + dc * i < BOARD_SIZE && board[r + dr * i][c + dc * i] === player) { count++; i++; }
                i = 1;
                while (r - dr * i >= 0 && r - dr * i < BOARD_SIZE && c - dc * i >= 0 && c - dc * i < BOARD_SIZE && board[r - dr * i][c - dc * i] === player) { count++; i++; }
                
                if (count >= 5) {
                    if (player === 1 && renjuRulesActive && count > 5) {
                        return false; 
                    }
                    return true; 
                }
            }
            return false;
        }

        function checkDraw() {
            for (let r = 0; r < BOARD_SIZE; r++) {
                for (let c = 0; c < BOARD_SIZE; c++) {
                    if (board[r][c] === 0) return false;
                }
            }
            return true;
        }

        function performUndo() {
            if (moveHistory.length === 0 || aiLock) return;
            if (!active) {
                active = true;
                gameOverModal.classList.add('hidden');
                updateStartBtn();
            }
            
            // 關閉覆盤
            isReviewMode = false;
            reviewNavigator.classList.add('hidden');

            let movesToUndo = (modeSelect.value === 'pve') ? 2 : 1;
            if (moveHistory.length < movesToUndo) movesToUndo = moveHistory.length;
            for(let i = 0; i < movesToUndo; i++) {
                let last = moveHistory.pop();
                board[last.r][last.c] = 0;
                curP = last.player; 
            }
            lastMove = moveHistory.length > 0 ? { r: moveHistory[moveHistory.length-1].r, c: moveHistory[moveHistory.length-1].c } : null;
            hintPos = null;
            updateStatus();
            updateWinRateBar();
            updateSelectorsInteractable(); 
            
            if (modeSelect.value === 'practice' && realTimeAnalysisToggle.checked) {
                calculatePracticeAnalysis();
            } else {
                drawBoard();
            }
            
            if (modeSelect.value === 'pve' && curP === parseInt(colorSelect.value) && active) triggerAI();
        }

        undoBtn.addEventListener('click', performUndo);
        practiceUndoBtn.addEventListener('click', performUndo);

        hintBtn.addEventListener('click', () => {
            if (!active || aiLock) return;
            const originalText = hintBtn.innerHTML;
            hintBtn.innerHTML = "⏳ 運算中...";
            hintBtn.disabled = true;
            hintBtn.classList.add('opacity-50');
            requestAnimationFrame(() => {
                requestAnimationFrame(() => {
                    const t0 = performance.now();
                    let stones = 0;
                    for(let r=0; r<BOARD_SIZE; r++) for(let c=0; c<BOARD_SIZE; c++) if(board[r][c]!==0) stones++;
                    if(stones === 0) hintPos = { r: 7, c: 7 };
                    else hintPos = getAlphaBetaMove(board, 5, curP, curP === 1 ? 2 : 1);
                    thinkTimeDisp.innerText = `${(performance.now() - t0).toFixed(1)} ms`;
                    hintBtn.innerHTML = originalText;
                    hintBtn.disabled = false;
                    hintBtn.classList.remove('opacity-50');
                    drawBoard();
                });
            });
        });

        // ==========================================
        // 📐 局後覆盤控制面板與狀態管理
        // ==========================================
        function initReviewMode() {
            if (moveHistory.length === 0) return;
            isReviewMode = true;
            reviewStepIndex = moveHistory.length;
            reviewNavigator.classList.remove('hidden');
            updateReviewUI();
        }

        function updateReviewUI() {
            reviewCounter.innerText = `${reviewStepIndex} / ${moveHistory.length}`;
            drawBoard();
        }

        revFirstBtn.addEventListener('click', () => {
            if (!isReviewMode) return;
            reviewStepIndex = 0;
            updateReviewUI();
        });

        revPrevBtn.addEventListener('click', () => {
            if (!isReviewMode) return;
            if (reviewStepIndex > 0) {
                reviewStepIndex--;
                updateReviewUI();
            }
        });

        revNextBtn.addEventListener('click', () => {
            if (!isReviewMode) return;
            if (reviewStepIndex < moveHistory.length) {
                reviewStepIndex++;
                updateReviewUI();
            }
        });

        revLastBtn.addEventListener('click', () => {
            if (!isReviewMode) return;
            reviewStepIndex = moveHistory.length;
            updateReviewUI();
        });

        // --- AI 搜尋與評估核心 ---
        function playAI(difficulty) {
            const aiPlayer = curP;
            const huPlayer = aiPlayer === 1 ? 2 : 1;
            let bestMove = null;
            let stones = 0;
            for(let r=0; r<BOARD_SIZE; r++) for(let c=0; c<BOARD_SIZE; c++) if(board[r][c]!==0) stones++;
            
            if (stones === 0) {
                bestMove = { r: 7, c: 7 }; 
            } else {
                if (difficulty === 1) {
                    let cands = getTopCandidates(board, aiPlayer, huPlayer, 20); 
                    bestMove = cands[Math.floor(Math.random() * cands.length)];
                } else if (difficulty === 2) {
                    let cands = getTopCandidates(board, aiPlayer, huPlayer, 5);
                    bestMove = cands[Math.floor(Math.random() * cands.length)];
                } else if (difficulty === 3) {
                    let cands = getTopCandidates(board, aiPlayer, huPlayer, 2);
                    if (cands.length > 1 && Math.random() > 0.7 && cands[0].score < 100000) bestMove = cands[1]; 
                    else bestMove = cands[0];
                } else {
                    let depth = difficulty - 2; 
                    bestMove = getAlphaBetaMove(board, depth, aiPlayer, huPlayer);
                }
            }

            if (bestMove) {
                board[bestMove.r][bestMove.c] = curP;
                moveHistory.push({ r: bestMove.r, c: bestMove.c, player: curP });
                lastMove = { r: bestMove.r, c: bestMove.c };
                hintPos = null;
                
                if (checkWin(bestMove.r, bestMove.c, curP)) { drawBoard(); endGame(curP); return; }
                if (checkDraw()) { drawBoard(); endGame(0); return; }
                curP = curP === 1 ? 2 : 1;
                updateStatus();
                updateWinRateBar();
                drawBoard();
            }
        }

        function getAlphaBetaMove(currentBoard, depth, aiPlayer, huPlayer) {
            let bestMove = null, bestScore = -Infinity, alpha = -Infinity, beta = Infinity;
            let searchWidth = depth >= 5 ? 6 : (depth === 4 ? 8 : 10); 
            let cands = getTopCandidates(currentBoard, aiPlayer, huPlayer, searchWidth);
            if (cands.length === 0) return null;
            if (cands[0].score >= 20000000) return cands[0]; 

            for (let i = 0; i < cands.length; i++) {
                let cand = cands[i];
                currentBoard[cand.r][cand.c] = aiPlayer;
                if (checkWin(cand.r, cand.c, aiPlayer)) { currentBoard[cand.r][cand.c] = 0; return cand; }
                let score = alphaBeta(currentBoard, depth - 1, alpha, beta, false, aiPlayer, huPlayer, searchWidth);
                currentBoard[cand.r][cand.c] = 0;
                if (score > bestScore) { bestScore = score; bestMove = cand; }
                alpha = Math.max(alpha, bestScore);
            }
            return bestMove || cands[0];
        }

        function alphaBeta(currentBoard, depth, alpha, beta, isMaximizing, aiPlayer, huPlayer, searchWidth) {
            if (depth === 0) return getPureScore(currentBoard, aiPlayer) - getPureScore(currentBoard, huPlayer) * 1.5;
            let currPlayer = isMaximizing ? aiPlayer : huPlayer;
            let oppoPlayer = isMaximizing ? huPlayer : aiPlayer;
            let cands = getTopCandidates(currentBoard, currPlayer, oppoPlayer, searchWidth);
            if (cands.length === 0) return 0;
            if (isMaximizing) {
                let maxEval = -Infinity;
                for (let i = 0; i < cands.length; i++) {
                    let cand = cands[i];
                    currentBoard[cand.r][cand.c] = aiPlayer;
                    if (checkWin(cand.r, cand.c, aiPlayer)) { currentBoard[cand.r][cand.c] = 0; return 20000000 + depth; }
                    let ev = alphaBeta(currentBoard, depth - 1, alpha, beta, false, aiPlayer, huPlayer, searchWidth);
                    currentBoard[cand.r][cand.c] = 0;
                    maxEval = Math.max(maxEval, ev);
                    alpha = Math.max(alpha, ev);
                    if (beta <= alpha) break; 
                }
                return maxEval;
            } else {
                let minEval = Infinity;
                for (let i = 0; i < cands.length; i++) {
                    let cand = cands[i];
                    currentBoard[cand.r][cand.c] = huPlayer;
                    if (checkWin(cand.r, cand.c, huPlayer)) { currentBoard[cand.r][cand.c] = 0; return -20000000 - depth; }
                    let ev = alphaBeta(currentBoard, depth - 1, alpha, beta, true, aiPlayer, huPlayer, searchWidth);
                    currentBoard[cand.r][cand.c] = 0;
                    minEval = Math.min(minEval, ev);
                    beta = Math.min(beta, ev);
                    if (beta <= alpha) break; 
                }
                return minEval;
            }
        }

        // 核心修改：局勢估值過濾連珠禁手點（防止 AI 誤將禁手點算入黑棋勝率權重中）
        function getPureScore(currBoard, player) {
            let finalMax = 0;
            for (let r = 0; r < BOARD_SIZE; r++) {
                for (let c = 0; c < BOARD_SIZE; c++) {
                    if (currBoard[r][c] === 0 && hasNeighbor(currBoard, r, c)) {
                        if (player === 1 && renjuRulesActive && checkRenjuForbidden(currBoard, r, c)) {
                            continue; 
                        }
                        let score = evaluateCellWithGaps(currBoard, r, c, player);
                        if (score > finalMax) finalMax = score;
                    }
                }
            }
            return finalMax;
        }

        // 核心修改：候選落子點智慧過濾禁手點，使黑方 AI 絕對不會走禁手，白方 AI 也不去防守黑方禁手點
        function getTopCandidates(currBoard, player, opponent, limit) {
            let moves = [];
            for (let r = 0; r < BOARD_SIZE; r++) {
                for (let c = 0; c < BOARD_SIZE; c++) {
                    if (currBoard[r][c] === 0 && hasNeighbor(currBoard, r, c)) {
                        // 1. 如果落子方是黑方且啟用禁手，直接過濾該點
                        if (player === 1 && renjuRulesActive && checkRenjuForbidden(currBoard, r, c)) {
                            continue; 
                        }

                        let myScore = evaluateCellWithGaps(currBoard, r, c, player);
                        let oppScore = 0;

                        // 2. 如果對手是黑方且啟用禁手，該點是黑棋禁手時，白棋完全不需要花防守分數
                        if (!(opponent === 1 && renjuRulesActive && checkRenjuForbidden(currBoard, r, c))) {
                            oppScore = evaluateCellWithGaps(currBoard, r, c, opponent);
                        }

                        let totalScore = 0;
                        if (myScore >= 10000000) totalScore = 20000000;      
                        else if (oppScore >= 10000000) totalScore = 10000000; 
                        else if (myScore >= 100000) totalScore = 5000000;     
                        else if (oppScore >= 100000) totalScore = 2000000;    
                        else if (myScore >= 10000 && oppScore >= 10000) totalScore = 1000000; 
                        else totalScore = myScore + oppScore * 1.25; 
                        totalScore += (15 - (Math.abs(r - 7) + Math.abs(c - 7))); 
                        moves.push({ r, c, score: totalScore });
                    }
                }
            }
            moves.sort((a, b) => b.score - a.score);
            return moves.slice(0, limit);
        }

        function hasNeighbor(currBoard, r, c) {
            for (let i = -2; i <= 2; i++) {
                for (let j = -2; j <= 2; j++) {
                    if (i === 0 && j === 0) continue;
                    let nr = r + i, nc = c + j;
                    if (nr >= 0 && nr < BOARD_SIZE && nc >= 0 && nc < BOARD_SIZE && currBoard[nr][nc] !== 0) return true;
                }
            }
            return false;
        }

        function evaluateCellWithGaps(currBoard, r, c, player) {
            let score = 0;
            const dirs = [[1, 0], [0, 1], [1, 1], [1, -1]];
            for (let [dr, dc] of dirs) {
                let line = "";
                for (let i = -4; i <= 4; i++) {
                    let nr = r + dr * i, nc = c + dc * i;
                    if (nr < 0 || nr >= BOARD_SIZE || nc < 0 || nr >= BOARD_SIZE) line += "2"; 
                    else {
                        let p = currBoard[nr][nc];
                        if (i === 0) line += "1"; 
                        else if (p === player) line += "1";
                        else if (p === 0) line += "0";
                        else line += "2";
                    }
                }
                score += getPatternScore(line);
            }
            return score;
        }

        // 智慧 Gap 啟發評估表
        function getPatternScore(line) {
            if (line.includes("11111")) return 10000000;
            let score = 0;
            if (line.includes("011110")) score += 100000;
            if (line.includes("011112") || line.includes("211110") || line.includes("10111") || line.includes("11011") || line.includes("11101")) score += 10000;
            if (line.includes("011100") || line.includes("001110") || line.includes("010110") || line.includes("011010")) score += 10000;
            if (line.includes("001112") || line.includes("211100") || line.includes("011012") || line.includes("210110") || line.includes("010112") || line.includes("211010") || line.includes("10011") || line.includes("11001") || line.includes("10101") || line.includes("2011102")) score += 1000;
            if (line.includes("001100") || line.includes("010100") || line.includes("001010") || line.includes("010010")) score += 1000;
            if (line.includes("000112") || line.includes("211000") || line.includes("001012") || line.includes("210100") || line.includes("010012") || line.includes("210010") || line.includes("10001") || line.includes("201102") || line.includes("2010102")) score += 100;
            return score;
        }

        // --- 啟動與初始化 ---
        setupAutoDarkMode(); 
        stopGame(); 
    </script>
</body>
</html>
