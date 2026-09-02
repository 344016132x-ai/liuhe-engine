<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>V34 走势分析系统 (完整版)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { font-family: system-ui, -apple-system, sans-serif; }
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 3px; }
        .toast-enter { opacity: 0; transform: translateY(100%); }
        .toast-enter-active { opacity: 1; transform: translateY(0); transition: all 0.3s ease-out; }
        .toast-exit { opacity: 1; }
        .toast-exit-active { opacity: 0; transition: all 0.3s ease-in; }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 flex flex-col h-screen overflow-x-hidden">

    <!-- 顶部导航条 -->
    <header class="bg-indigo-700 text-white shadow-md p-4 sticky top-0 z-10">
        <div class="max-w-4xl mx-auto flex justify-between items-center">
            <h1 class="text-xl md:text-2xl font-bold tracking-wide">V34 走势分析引擎</h1>
            <span class="bg-indigo-800 px-2 py-1 rounded text-xs font-mono border border-indigo-600">Phase 3B Stable</span>
        </div>
    </header>

    <!-- 主体容器 -->
    <main class="flex-1 overflow-y-auto p-3 md:p-6">
        <div class="max-w-4xl mx-auto space-y-4 md:space-y-6">

            <!-- 数据录入区 -->
            <section class="bg-white p-4 md:p-6 rounded-xl shadow-sm border border-slate-200">
                <h2 class="text-lg font-bold mb-3 text-slate-700 flex items-center">
                    <svg class="w-5 h-5 mr-2 text-indigo-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path></svg>
                    历史数据录入
                </h2>
                <textarea id="importDataInput" class="w-full border border-slate-300 p-3 h-28 md:h-32 rounded-lg text-sm focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 outline-none transition-all resize-none" placeholder="请粘贴历史数据。格式示例：&#10;121,龙&#10;122,虎&#10;123,兔&#10;（支持中文逗号或英文逗号分隔）"></textarea>
                <div class="flex gap-3 mt-3">
                    <button id="importDataBtn" class="flex-1 md:flex-none bg-indigo-600 text-white px-5 py-2.5 rounded-lg font-semibold hover:bg-indigo-700 transition-colors shadow-sm text-sm">导入并解析</button>
                    <button id="clearDataBtn" class="flex-1 md:flex-none bg-rose-50 text-rose-600 border border-rose-200 px-5 py-2.5 rounded-lg font-semibold hover:bg-rose-100 transition-colors text-sm">清空历史</button>
                </div>
            </section>

            <!-- 线索微调区 -->
            <section class="bg-white p-4 md:p-6 rounded-xl shadow-sm border border-slate-200">
                <h2 class="text-lg font-bold mb-3 text-slate-700 flex items-center">
                    <svg class="w-5 h-5 mr-2 text-indigo-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"></path></svg>
                    形态线索微调
                </h2>
                <div id="clueContainer" class="flex flex-wrap gap-2"></div>
            </section>

            <!-- 分析结果区 -->
            <section class="bg-white p-4 md:p-6 rounded-xl shadow-sm border border-slate-200">
                <div class="flex flex-col md:flex-row justify-between items-start md:items-center mb-4 gap-3">
                    <div>
                        <h2 class="text-lg font-bold text-slate-700 flex items-center">
                            <svg class="w-5 h-5 mr-2 text-indigo-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"></path></svg>
                            综合分析面板
                        </h2>
                        <div id="historyStats" class="text-xs md:text-sm text-slate-500 mt-1">当前已录入 0 期历史数据</div>
                    </div>
                    <button id="runAnalysisBtn" class="w-full md:w-auto bg-emerald-600 text-white px-6 py-2.5 rounded-lg font-bold hover:bg-emerald-700 transition-colors shadow-md text-sm">执行全盘分析</button>
                </div>
                
                <!-- 核心 Analytics 结果 -->
                <div id="analysisResults" class="min-h-[100px] transition-all">
                    <div class="text-slate-400 text-center py-8 text-sm bg-slate-50 rounded-lg border border-dashed border-slate-300">暂无分析结果，请录入数据并执行分析。</div>
                </div>

                <!-- 走势偏离信号容器 (动态挂载) -->
                <div id="v34-pattern-signals-container" class="mt-6 bg-white rounded-lg shadow-sm border border-slate-200 overflow-hidden hidden">
                    <!-- JS动态填充 -->
                </div>
            </section>
        </div>
    </main>

    <!-- Toast 通知容器 -->
    <div id="toastContainer" class="fixed bottom-4 right-4 z-50 flex flex-col gap-2"></div>

    <script>
        /**
         * ==========================================
         * V34 ALL-IN-ONE MODULE SYSTEM
         * 为了遵循 Single-File Mandate，将所有 ES Module 合并于此。
         * 严格保持作用域隔离与原有架构设计。
         * ==========================================
         */

        // --- 1. CONFIG MODULE ---
        const ZodiacConfig = {
            ALL_ZODIACS: ['鼠', '牛', '虎', '兔', '龙', '蛇', '马', '羊', '猴', '鸡', '狗', '猪']
        };

        // --- 2. STORAGE MODULE ---
        const StorageEngine = {
            KEY: 'v34_history_data',
            save: function(data) {
                try {
                    localStorage.setItem(this.KEY, JSON.stringify(data));
                    return true;
                } catch(e) { console.error('Storage Save Error:', e); return false; }
            },
            load: function() {
                try {
                    const raw = localStorage.getItem(this.KEY);
                    return raw ? JSON.parse(raw) : [];
                } catch(e) { console.error('Storage Load Error:', e); return []; }
            },
            clear: function() {
                localStorage.removeItem(this.KEY);
            }
        };

        // --- 3. ANALYTICS MODULE (V34 Core) ---
        class AnalyticsEngine {
            constructor(historyData, clueSelections) {
                this.historyData = historyData; // Array of zodiac strings e.g. ['龙', '虎', '兔', ...] (Index 0 is latest)
                this.clueSelections = clueSelections; // Set of zodiac strings
                this.zodiacStats = {};
                ZodiacConfig.ALL_ZODIACS.forEach(z => {
                    this.zodiacStats[z] = { freq: 0, omission: 0, score: 0 };
                });
            }

            analyze() {
                if (!this.historyData || this.historyData.length === 0) return null;

                // 1. Calculate Frequencies & Omissions
                ZodiacConfig.ALL_ZODIACS.forEach(z => {
                    let found = false;
                    for (let i = 0; i < this.historyData.length; i++) {
                        if (this.historyData[i] === z) {
                            this.zodiacStats[z].freq++;
                            if (!found) {
                                this.zodiacStats[z].omission = i;
                                found = true;
                            }
                        }
                    }
                    if (!found) this.zodiacStats[z].omission = this.historyData.length;
                });

                // 2. Identify Consecutive & Extreme Cold
                let consecutiveCount = 1;
                let consecutiveZodiac = this.historyData[0];
                for (let i = 1; i < this.historyData.length; i++) {
                    if (this.historyData[i] === consecutiveZodiac) consecutiveCount++;
                    else break;
                }
                const consecutiveStr = consecutiveCount > 1 ? `${consecutiveZodiac} (${consecutiveCount}连)` : '无明显连出';

                let extremeColdZodiac = ZodiacConfig.ALL_ZODIACS[0];
                let maxOmission = -1;
                ZodiacConfig.ALL_ZODIACS.forEach(z => {
                    if (this.zodiacStats[z].omission > maxOmission) {
                        maxOmission = this.zodiacStats[z].omission;
                        extremeColdZodiac = z;
                    }
                });
                const extremeColdStr = `${extremeColdZodiac} (遗漏 ${maxOmission} 期)`;

                // 3. Scoring (V34 Tie-Break Logic)
                // Score = Freq * 2 - Omission + (Clue Bonus)
                ZodiacConfig.ALL_ZODIACS.forEach(z => {
                    let score = (this.zodiacStats[z].freq * 2) - this.zodiacStats[z].omission;
                    if (this.clueSelections.has(z)) score += 10;
                    this.zodiacStats[z].score = score;
                });

                // 4. Sort Top5 and Kill3
                const sortedZodiacs = [...ZodiacConfig.ALL_ZODIACS].sort((a, b) => {
                    if (this.zodiacStats[b].score !== this.zodiacStats[a].score) {
                        return this.zodiacStats[b].score - this.zodiacStats[a].score;
                    }
                    // Tie break: lower omission wins for top
                    return this.zodiacStats[a].omission - this.zodiacStats[b].omission; 
                });

                return {
                    top5: sortedZodiacs.slice(0, 5),
                    kill3: sortedZodiacs.slice(-3).reverse(), // Worst 3
                    consecutive: consecutiveStr,
                    extremeCold: extremeColdStr,
                    stats: this.zodiacStats
                };
            }
        }

        // --- 4. PREDICTION MODULE (Phase 3A/3B Pattern Signals) ---
        class PredictionEngine {
            constructor(rawZodiacs) {
                this.rawZodiacs = rawZodiacs || [];
                // Configuration parameters safely isolated
                this.SHORT_WINDOW = 10;
                this.REVERSION_BASELINE = 24;
            }

            generatePrediction() {
                const totalSamples = this.rawZodiacs.length;
                
                // State Machine Enforcer
                if (totalSamples < 10) {
                    return { modelStatus: 'COLD_START', predictionTop3: [], predictionAvoid: [], predictionDetails: [] };
                }

                let modelStatus = totalSamples < 30 ? 'LOW_CONFIDENCE' : 'ACTIVE';
                const details = [];

                ZodiacConfig.ALL_ZODIACS.forEach(z => {
                    // Momentum (Recent frequency in Short Window)
                    const recentSlice = this.rawZodiacs.slice(0, this.SHORT_WINDOW);
                    const recentFreq = recentSlice.filter(x => x === z).length;
                    
                    // Current Omission
                    let currentOmission = this.rawZodiacs.indexOf(z);
                    if (currentOmission === -1) currentOmission = totalSamples;

                    // Baseline (Frequency in Reversion Window)
                    const baselineSlice = this.rawZodiacs.slice(0, this.REVERSION_BASELINE);
                    const baselineFreq = baselineSlice.filter(x => x === z).length;

                    // Math.clamp function equivalent mapping to [0, 100]
                    // Momentum vs Reversion index calculation
                    let rawIndex = 50 + (recentFreq * 8) - (currentOmission * 1.5) + (baselineFreq * 2);
                    rawIndex = Math.max(0, Math.min(100, rawIndex)); // Clamp 0-100 to ensure Finite Number

                    const signals = [];
                    if (recentFreq >= 3) signals.push('短期极热');
                    if (currentOmission > 15) signals.push('深度沉睡');
                    if (rawIndex > 80) signals.push('强动量上行');
                    if (rawIndex < 20) signals.push('均值回归受阻');

                    details.push({
                        zodiac: z,
                        predictionIndex: rawIndex,
                        recentFreq: recentFreq,
                        currentOmission: currentOmission,
                        signals: signals
                    });
                });

                // Sort by prediction index descending
                details.sort((a, b) => b.predictionIndex - a.predictionIndex);

                return {
                    modelStatus,
                    predictionTop3: details.slice(0, 3).map(d => d.zodiac),
                    predictionAvoid: details.slice(-3).map(d => d.zodiac),
                    predictionDetails: details
                };
            }
        }

        // --- 5. UI CONTROLLER MODULE ---
        const UIController = {
            clueSelections: new Set(),
            appContext: null,

            init: function(appContext) {
                this.appContext = appContext;
                this.bindEvents();
                this.renderClueSelector();
            },

            bindEvents: function() {
                document.getElementById('importDataBtn').addEventListener('click', () => this.appContext.importData());
                document.getElementById('clearDataBtn').addEventListener('click', () => this.appContext.clearData());
                document.getElementById('runAnalysisBtn').addEventListener('click', () => this.appContext.runAnalysis());
            },

            showToast: function(message, type = 'success') {
                const container = document.getElementById('toastContainer');
                const toast = document.createElement('div');
                const bgColor = type === 'error' ? 'bg-rose-500' : (type === 'warn' ? 'bg-amber-500' : 'bg-emerald-500');
                
                toast.className = `${bgColor} text-white px-4 py-2.5 rounded shadow-lg text-sm font-medium toast-enter`;
                toast.textContent = message;
                
                container.appendChild(toast);
                
                // Trigger reflow to start animation
                toast.offsetHeight;
                toast.classList.remove('toast-enter');
                toast.classList.add('toast-enter-active');

                setTimeout(() => {
                    toast.classList.remove('toast-enter-active');
                    toast.classList.add('toast-exit');
                    setTimeout(() => {
                        toast.classList.add('toast-exit-active');
                        setTimeout(() => toast.remove(), 300);
                    }, 50);
                }, 3000);
            },

            updateDashboardStatus: function(totalCount) {
                const statsEl = document.getElementById('historyStats');
                if (statsEl) {
                    statsEl.textContent = `当前已录入 ${totalCount} 期历史数据`;
                }
            },

            renderClueSelector: function() {
                const container = document.getElementById('clueContainer');
                if (!container) return;
                
                container.innerHTML = '';
                ZodiacConfig.ALL_ZODIACS.forEach(z => {
                    const btn = document.createElement('button');
                    const isSelected = this.clueSelections.has(z);
                    btn.className = `px-4 py-2 text-sm rounded-md border transition-all duration-200 ${isSelected ? 'bg-indigo-600 text-white border-indigo-600 shadow-md font-bold transform scale-105' : 'bg-slate-50 text-slate-700 border-slate-200 hover:bg-slate-100 hover:border-slate-300'}`;
                    btn.textContent = z;
                    btn.onclick = () => {
                        if (isSelected) {
                            this.clueSelections.delete(z);
                        } else {
                            this.clueSelections.add(z);
                        }
                        this.renderClueSelector();
                        // 触发实时刷新
                        if (this.appContext && this.appContext.historyData.length > 0) {
                            this.appContext.runAnalysis(true); // Silent run
                        }
                    };
                    container.appendChild(btn);
                });
            },

            updateResults: function(analyticsResults, predictionContext = null) {
                this._renderV34Analytics(analyticsResults);
                this._renderPatternSignals(predictionContext);
            },

            _renderV34Analytics: function(results) {
                const resultsEl = document.getElementById('analysisResults');
                if (!resultsEl) return;
                
                if (!results) {
                    resultsEl.innerHTML = '<div class="text-slate-400 text-center py-8 text-sm bg-slate-50 rounded-lg border border-dashed border-slate-300">暂无分析结果，请录入数据并执行分析。</div>';
                    return;
                }

                let html = `<div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">`;
                
                html += `<div class="bg-indigo-50/50 p-4 rounded-lg border border-indigo-100">`;
                html += `<h3 class="font-bold text-indigo-800 mb-3 text-sm flex items-center"><svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"></path></svg>综合推荐 Top 5</h3>`;
                html += `<div class="flex flex-wrap gap-2">`;
                results.top5.forEach(z => html += `<span class="bg-indigo-600 text-white px-3 py-1.5 rounded-md font-bold text-sm shadow-sm">${z}</span>`);
                html += `</div></div>`;

                html += `<div class="bg-rose-50/50 p-4 rounded-lg border border-rose-100">`;
                html += `<h3 class="font-bold text-rose-800 mb-3 text-sm flex items-center"><svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18.364 18.364A9 9 0 005.636 5.636m12.728 12.728A9 9 0 015.636 5.636m12.728 12.728L5.636 5.636"></path></svg>低置信排除 Kill 3</h3>`;
                html += `<div class="flex flex-wrap gap-2">`;
                results.kill3.forEach(z => html += `<span class="bg-rose-500 text-white px-3 py-1.5 rounded-md font-bold text-sm shadow-sm">${z}</span>`);
                html += `</div></div>`;
                html += `</div>`;

                html += `<div class="flex flex-col md:flex-row gap-4 p-4 bg-slate-50/80 rounded-lg text-sm text-slate-700 border border-slate-100">`;
                html += `<div class="flex-1"><span class="text-slate-500 mr-2">近期高频连出:</span> <strong class="text-indigo-700">${results.consecutive}</strong></div>`;
                html += `<div class="flex-1"><span class="text-slate-500 mr-2">历史极冷沉睡:</span> <strong class="text-rose-700">${results.extremeCold}</strong></div>`;
                html += `</div>`;

                resultsEl.innerHTML = html;
            },

            _renderPatternSignals: function(context) {
                const container = document.getElementById('v34-pattern-signals-container');
                if (!container) return;

                if (!context) {
                    container.style.display = 'none';
                    return;
                }
                
                container.style.display = 'block';
                
                if (context.error) {
                    container.innerHTML = `<div class="p-4 text-rose-600 font-bold bg-rose-50 border-l-4 border-rose-500 text-sm">${context.message}</div>`;
                    return;
                }

                // 强制合规免责声明
                const disclaimer = `
                    <div class="p-3 bg-amber-50/80 text-amber-800 text-xs border-b border-amber-200 flex items-start gap-2 leading-relaxed">
                        <span class="mt-0.5 text-amber-500"><svg class="w-4 h-4" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M8.257 3.099c.765-1.36 2.722-1.36 3.486 0l5.58 9.92c.75 1.334-.213 2.98-1.742 2.98H4.42c-1.53 0-2.493-1.646-1.743-2.98l5.58-9.92zM11 13a1 1 0 11-2 0 1 1 0 012 0zm-1-8a1 1 0 00-1 1v3a1 1 0 002 0V6a1 1 0 00-1-1z" clip-rule="evenodd"></path></svg></span>
                        <span><b>安全免责声明：</b> 本模块仅对历史遗漏等形态进行数学特征映射。历史形态不代表未来开奖结果，指数也不是概率或胜率，切勿作为购买依据。</span>
                    </div>
                `;

                let html = `${disclaimer}<div class="p-4 md:p-5">`;
                html += `<h2 class="text-base font-bold text-slate-800 mb-4 border-l-3 border-indigo-400 pl-2 flex items-center">走势偏离信号 (Phase 3B)</h2>`;

                if (context.modelStatus === 'COLD_START') {
                    html += `<div class="py-8 text-center text-slate-400 text-sm bg-slate-50 rounded-lg border border-slate-100">❄️ 冷启动：当前历史样本不足 (需 ≥10期)，暂不生成走势信号。</div>`;
                } else {
                    if (context.modelStatus === 'LOW_CONFIDENCE') {
                        html += `<div class="mb-4 text-xs text-amber-700 font-semibold bg-amber-50/50 border border-amber-100 p-2.5 rounded-md flex items-center"><svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>低置信状态：样本量较小 (<30期)，仅供基础形态观察。</div>`;
                    }

                    html += `<div class="flex flex-col md:flex-row gap-4 mb-6">`;
                    
                    // Top 3
                    html += `<div class="flex-1 p-4 bg-slate-50 rounded-lg border border-slate-200 shadow-sm">`;
                    html += `<h3 class="font-bold text-slate-700 mb-1 text-sm flex items-center"><svg class="w-4 h-4 mr-1 text-emerald-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 7h8m0 0v8m0-8l-8 8-4-4-6 6"></path></svg>高偏离形态 Top 3</h3>`;
                    html += `<p class="text-[11px] text-slate-400 mb-3">排名仅代表偏离指数，不构成推荐。</p>`;
                    html += `<div class="flex gap-2">`;
                    (context.predictionTop3 || []).forEach(z => html += `<span class="bg-slate-700 text-white px-3 py-1.5 rounded-md font-medium text-sm shadow-sm">${z}</span>`);
                    html += `</div></div>`;

                    // Avoid
                    html += `<div class="flex-1 p-4 bg-slate-50 rounded-lg border border-slate-200 shadow-sm">`;
                    html += `<h3 class="font-bold text-slate-600 mb-1 text-sm flex items-center"><svg class="w-4 h-4 mr-1 text-rose-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 17h8m0 0V9m0 8l-8-8-4 4-6-6"></path></svg>低偏离形态 Top 3</h3>`;
                    html += `<p class="text-[11px] text-slate-400 mb-3">处于均值回归末端，动量较弱。</p>`;
                    html += `<div class="flex gap-2">`;
                    (context.predictionAvoid || []).forEach(z => html += `<span class="bg-slate-200 text-slate-700 px-3 py-1.5 rounded-md font-medium text-sm border border-slate-300 shadow-sm">${z}</span>`);
                    html += `</div></div>`;
                    
                    html += `</div>`;

                    // Details Matrix
                    if (context.predictionDetails && context.predictionDetails.length > 0) {
                        html += `<h3 class="font-bold text-slate-700 mb-3 text-sm">形态偏离参数矩阵</h3>`;
                        html += `<div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-6 gap-3">`;
                        
                        context.predictionDetails.forEach(d => {
                            const signalText = (d.signals && d.signals.length > 0) ? d.signals.join(', ') : '形态平稳';
                            const signalClass = d.signals.length > 0 ? 'text-indigo-600 font-bold' : 'text-slate-400';
                            
                            html += `
                                <div class="p-3 border border-slate-100 rounded-lg shadow-sm bg-white hover:border-indigo-200 transition-colors">
                                    <div class="flex justify-between items-center mb-2 border-b border-slate-50 pb-2">
                                        <span class="font-bold text-base text-slate-800">${d.zodiac}</span>
                                        <span class="text-[10px] font-mono bg-indigo-50 text-indigo-700 px-1.5 py-0.5 rounded border border-indigo-100">IDX ${d.predictionIndex.toFixed(1)}</span>
                                    </div>
                                    <div class="text-[11px] text-slate-500 space-y-1.5">
                                        <div class="flex justify-between"><span>近期出现</span> <span class="font-semibold text-slate-700">${d.recentFreq}次</span></div>
                                        <div class="flex justify-between"><span>当前遗漏</span> <span class="font-semibold text-slate-700">${d.currentOmission}期</span></div>
                                        <div class="mt-2 text-[10px] truncate ${signalClass}" title="${signalText}">${signalText}</div>
                                    </div>
                                </div>
                            `;
                        });
                        html += `</div>`;
                    }
                }

                html += `</div>`;
                container.innerHTML = html;
            }
        };

        // --- 6. APP ORCHESTRATOR MODULE ---
        const App = {
            historyData: [], // Format: [{period: '121', zodiac: '龙'}, ...]

            init: function() {
                UIController.init(this);
                this.loadData();
                
                // Initialize default empty UI state
                UIController.updateDashboardStatus(this.historyData.length);
                if (this.historyData.length > 0) {
                    this.runAnalysis(true); // silent run on load
                }
            },

            loadData: function() {
                this.historyData = StorageEngine.load();
            },

            importData: function() {
                const input = document.getElementById('importDataInput').value.trim();
                if (!input) {
                    UIController.showToast('请输入历史数据', 'warn');
                    return;
                }

                const lines = input.split('\n');
                let newRecords = [];
                let parseError = false;

                lines.forEach(line => {
                    if (!line.trim()) return;
                    // Support both english and chinese comma parsing
                    const parts = line.split(/[,，]/).map(p => p.trim()); 
                    if (parts.length === 2 && ZodiacConfig.ALL_ZODIACS.includes(parts[1])) {
                        newRecords.push({ period: parts[0], zodiac: parts[1] });
                    } else {
                        parseError = true;
                    }
                });

                if (newRecords.length === 0) {
                    UIController.showToast('未能解析出有效数据，请检查格式。', 'error');
                    return;
                }

                // Add to beginning (latest first) to simulate real app history flow
                // Or handle based on V34 logic. Assuming user imports chronological or reverse.
                // We'll just replace state for simplicity in this artifact, or append.
                // V34 standard: Latest at index 0.
                this.historyData = [...newRecords.reverse(), ...this.historyData];
                
                // Deduplicate by period
                const seen = new Set();
                this.historyData = this.historyData.filter(item => {
                    const duplicate = seen.has(item.period);
                    seen.add(item.period);
                    return !duplicate;
                });

                StorageEngine.save(this.historyData);
                document.getElementById('importDataInput').value = '';
                
                UIController.updateDashboardStatus(this.historyData.length);
                if (parseError) {
                    UIController.showToast(`部分数据解析失败。成功导入 ${newRecords.length} 条。`, 'warn');
                } else {
                    UIController.showToast(`成功导入 ${newRecords.length} 条数据`);
                }
            },

            clearData: function() {
                if (confirm('确定要清空所有历史数据吗？此操作不可恢复。')) {
                    StorageEngine.clear();
                    this.historyData = [];
                    UIController.updateDashboardStatus(0);
                    UIController.clueSelections.clear();
                    UIController.renderClueSelector();
                    UIController.updateResults(null, null); // Reset UI to Empty State
                    UIController.showToast('数据已清空');
                }
            },

            runAnalysis: function(silent = false) {
                if (this.historyData.length === 0) {
                    if(!silent) UIController.showToast('暂无历史数据，无法执行分析', 'warn');
                    return;
                }

                // 1. Run Analytics Engine (Core V34)
                // Extract just the zodiac array for the engine e.g., ['龙', '虎', ...]
                const rawZodiacs = this.historyData.map(r => r.zodiac);
                
                const analytics = new AnalyticsEngine(rawZodiacs, UIController.clueSelections);
                const analyticsResults = analytics.analyze();

                // 2. Run Prediction Engine (Phase 3B Signal Integration isolated via try-catch)
                let predictionContext = null;
                try {
                    const predictionEngine = new PredictionEngine(rawZodiacs);
                    predictionContext = predictionEngine.generatePrediction();
                } catch (err) {
                    console.error("Prediction Engine crashed:", err);
                    predictionContext = { error: true, message: "走势信号暂不可用，内部计算异常。" };
                }

                // 3. Dispatch to UI
                UIController.updateResults(analyticsResults, predictionContext);
                
                if(!silent) UIController.showToast('分析完成！');
            }
        };

        // Bootstrap the application on load
        document.addEventListener('DOMContentLoaded', () => {
            App.init();
        });

    </script>
</body>
</html>

