<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>개선대책 유효성 점검 대시보드</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* GitHub Pages 마크다운 기본 스타일 간섭 방지 */
        body .markdown-body h1, body > h1:first-child { display: none !important; }
        header h1 { border-bottom: none !important; margin: 0 !important; padding: 0 !important; }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen">

    <!-- Header (줄바꿈 방지 및 슬림 레이아웃) -->
    <header class="bg-indigo-900 text-white shadow-md">
        <div class="max-w-7xl mx-auto px-5 py-3 flex flex-row justify-between items-center gap-4">
            <h1 class="text-base sm:text-lg font-bold tracking-tight whitespace-nowrap text-white">
                개선대책 유효성 점검 대시보드
            </h1>
            <div class="flex items-center space-x-2.5 shrink-0">
                <a href="https://docs.google.com/spreadsheets/d/1CAAK11qrimDmHXvii7zRcUaWFTY6ENMT-LB7q5nrJz4/edit?gid=359651245#gid=359651245" 
                   target="_blank" 
                   class="inline-flex items-center px-3 py-1.5 rounded-lg text-xs font-semibold bg-indigo-800 hover:bg-indigo-700 text-indigo-100 transition-colors border border-indigo-700 shadow-sm whitespace-nowrap">
                    📊 상세 원본 시트 열기
                </a>
                <span id="sync-status" class="inline-flex items-center px-3 py-1.5 rounded-full text-xs font-medium bg-emerald-100 text-emerald-800 whitespace-nowrap">
                    <span class="w-2 h-2 mr-1.5 bg-emerald-500 rounded-full animate-pulse"></span> 연동 중
                </span>
            </div>
        </div>
    </header>

    <!-- Main Container -->
    <main class="max-w-7xl mx-auto px-4 py-5 space-y-5">

        <!-- KPI 현황판: 좌측 3개(위->아래) / 우측 3개(위->아래) 배치 -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            
            <!-- 좌측 컬럼: 전체 관리 항목 / 전체 점검 완료율 / 대상 업체 수 -->
            <div class="flex flex-col space-y-3">
                <!-- 1. 전체 관리 항목 -->
                <div class="bg-white px-5 py-4 rounded-xl shadow-sm border border-slate-200/80 flex items-center justify-between">
                    <div class="flex items-center space-x-3.5">
                        <div class="p-2.5 bg-indigo-50 text-indigo-600 rounded-lg text-lg font-bold shrink-0">📋</div>
                        <div>
                            <p class="text-xs font-bold text-slate-500 whitespace-nowrap">전체 관리 항목</p>
                            <p id="kpi-total-sub" class="text-[11px] text-slate-400 mt-0.5 whitespace-nowrap">총 점검 주기 0회</p>
                        </div>
                    </div>
                    <h3 id="kpi-total" class="text-2xl font-bold text-slate-900 whitespace-nowrap">0 건</h3>
                </div>

                <!-- 2. 전체 점검 완료율 -->
                <div class="bg-white px-5 py-4 rounded-xl shadow-sm border border-slate-200/80 flex items-center justify-between">
                    <div class="flex items-center space-x-3.5">
                        <div class="p-2.5 bg-blue-50 text-blue-600 rounded-lg text-lg font-bold shrink-0">📈</div>
                        <div>
                            <p class="text-xs font-bold text-slate-500 whitespace-nowrap">전체 점검 완료율</p>
                            <p id="kpi-rate-sub" class="text-[11px] text-slate-400 mt-0.5 whitespace-nowrap">0 / 0 주기 완료</p>
                        </div>
                    </div>
                    <h3 id="kpi-rate" class="text-2xl font-bold text-indigo-600 whitespace-nowrap">0%</h3>
                </div>

                <!-- 3. 대상 업체 수 -->
                <div class="bg-white px-5 py-4 rounded-xl shadow-sm border border-slate-200/80 flex items-center justify-between">
                    <div class="flex items-center space-x-3.5">
                        <div class="p-2.5 bg-slate-100 text-slate-700 rounded-lg text-lg font-bold shrink-0">🏢</div>
                        <div>
                            <p class="text-xs font-bold text-slate-500 whitespace-nowrap">대상 업체 수</p>
                            <p id="kpi-partners-sub" class="text-[11px] text-slate-400 mt-0.5 whitespace-nowrap">참여 협력사 현황</p>
                        </div>
                    </div>
                    <h3 id="kpi-partners" class="text-2xl font-bold text-slate-800 whitespace-nowrap">0 개사</h3>
                </div>
            </div>

            <!-- 우측 컬럼: 정상 / 지연 / 경과 -->
            <div class="flex flex-col space-y-3">
                <!-- 1. 정상 (초록색) -->
                <div class="bg-white px-5 py-4 rounded-xl shadow-sm border border-emerald-200 flex items-center justify-between">
                    <div class="flex items-center space-x-3.5">
                        <div class="p-2.5 bg-emerald-50 text-emerald-600 rounded-lg text-lg font-bold shrink-0">✅</div>
                        <div>
                            <div class="flex items-center space-x-1.5">
                                <span class="w-2.5 h-2.5 rounded-full bg-emerald-500 shrink-0"></span>
                                <p class="text-xs font-bold text-emerald-700 whitespace-nowrap">정상 (기준일 대비 2일 이내)</p>
                            </div>
                            <p id="kpi-normal-sub" class="text-[11px] text-slate-400 mt-0.5 whitespace-nowrap">전 주기 정상 품목 0건</p>
                        </div>
                    </div>
                    <h3 id="kpi-normal" class="text-2xl font-bold text-emerald-600 whitespace-nowrap">0 건</h3>
                </div>

                <!-- 2. 지연 (주황색) -->
                <div class="bg-white px-5 py-4 rounded-xl shadow-sm border border-amber-200 flex items-center justify-between">
                    <div class="flex items-center space-x-3.5">
                        <div class="p-2.5 bg-amber-50 text-amber-500 rounded-lg text-lg font-bold shrink-0">⚠️</div>
                        <div>
                            <div class="flex items-center space-x-1.5">
                                <span class="w-2.5 h-2.5 rounded-full bg-amber-500 shrink-0"></span>
                                <p class="text-xs font-bold text-amber-700 whitespace-nowrap">지연 (기준일 대비 3~5일 이내)</p>
                            </div>
                            <p id="kpi-delayed-sub" class="text-[11px] text-slate-400 mt-0.5 whitespace-nowrap">지연 발생 품목 0건</p>
                        </div>
                    </div>
                    <h3 id="kpi-delayed" class="text-2xl font-bold text-amber-500 whitespace-nowrap">0 건</h3>
                </div>

                <!-- 3. 경과 (빨간색) -->
                <div class="bg-white px-5 py-4 rounded-xl shadow-sm border border-rose-200 flex items-center justify-between">
                    <div class="flex items-center space-x-3.5">
                        <div class="p-2.5 bg-rose-50 text-rose-600 rounded-lg text-lg font-bold shrink-0">🚨</div>
                        <div>
                            <div class="flex items-center space-x-1.5">
                                <span class="w-2.5 h-2.5 rounded-full bg-rose-500 shrink-0"></span>
                                <p class="text-xs font-bold text-rose-700 whitespace-nowrap">경과 (기준일 대비 5일 초과)</p>
                            </div>
                            <p id="kpi-overdue-sub" class="text-[11px] text-slate-400 mt-0.5 whitespace-nowrap">경과 발생 품목 0건</p>
                        </div>
                    </div>
                    <h3 id="kpi-overdue" class="text-2xl font-bold text-rose-600 whitespace-nowrap">0 건</h3>
                </div>
            </div>

        </div>

        <!-- 업체별 요약 현황 카드 -->
        <div class="bg-white p-4 rounded-xl shadow-sm border border-slate-100">
            <h3 class="text-sm font-bold text-slate-800 mb-3">업체별(OSA) 점검 진행 및 지연 요약</h3>
            <div id="partner-summary" class="grid grid-cols-1 md:grid-cols-3 gap-3">
                <!-- JS로 동적 생성 -->
            </div>
        </div>

        <!-- Detailed Table Section -->
        <div class="bg-white rounded-xl shadow-sm border border-slate-100 overflow-hidden">
            <div class="p-4 border-b border-slate-100 flex flex-col sm:flex-row justify-between items-center gap-3">
                <div>
                    <h3 class="text-base font-bold text-slate-900">개선 대책 상세 진행 목록</h3>
                    <p class="text-xs text-slate-400 mt-0.5">* 관리번호 형식: 업체명_년도_순번 | 색상 기준: 초록(정상, ≤2일), 주황(지연, 3~5일), 빨강(경과, &gt;5일)</p>
                </div>
                <!-- 관리번호 업체명 필터 셀렉트박스 -->
                <div class="flex items-center space-x-2 shrink-0">
                    <label for="partner-filter" class="text-xs font-semibold text-slate-600 whitespace-nowrap">업체명 필터:</label>
                    <select id="partner-filter" onchange="filterTable()" class="px-3 py-1.5 text-xs bg-slate-50 border border-slate-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500 text-slate-700 font-semibold">
                        <option value="ALL">전체 업체 보기</option>
                    </select>
                </div>
            </div>
            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr class="bg-slate-50 text-slate-500 text-xs uppercase font-semibold border-b border-slate-200">
                            <th class="p-3.5 w-36 whitespace-nowrap">관리번호</th>
                            <th class="p-3.5 w-28 whitespace-nowrap">발생일</th>
                            <th class="p-3.5 w-28 whitespace-nowrap">OSA</th>
                            <th class="p-3.5 w-32 whitespace-nowrap">고객사</th>
                            <th class="p-3.5 w-44 whitespace-nowrap">품명</th>
                            <th class="p-3.5 whitespace-nowrap">불량내용</th>
                        </tr>
                    </thead>
                    <tbody id="table-body" class="divide-y divide-slate-100 text-sm">
                        <tr>
                            <td colspan="6" class="p-6 text-center text-slate-400">데이터를 불러오는 중입니다...</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

    </main>

    <!-- JavaScript 로직 -->
    <script>
        const SHEET_CSV_URL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vRCGmQTOQd1DK4cmMzKU618FjIvvcwcSCgX3PBwtNF0i7_Q6aK3Hux-W56QCwAwNY7O2bff6zZ01RZm/pub?gid=359651245&single=true&output=csv";

        let allParsedData = [];

        window.addEventListener('DOMContentLoaded', () => {
            // GitHub Pages README.md 자동 생성 타이틀(QQQ 등) 감지 및 자동 제거
            document.querySelectorAll('h1, p').forEach(el => {
                if (el.textContent.trim() === 'QQQ' || el.textContent.includes('<!DOCTYPE')) {
                    el.style.display = 'none';
                }
            });

            fetchAndRenderData();
            setInterval(fetchAndRenderData, 60000);
        });

        async function fetchAndRenderData() {
            const targetUrl = SHEET_CSV_URL + (SHEET_CSV_URL.includes('?') ? '&' : '?') + 't=' + Date.now();

            try {
                const response = await fetch(targetUrl, { cache: 'no-store' });
                if (!response.ok) throw new Error('네트워크 응답 오류 발생');
                
                const csvText = await response.text();
                parseCSVAndRender(csvText);

                document.getElementById('sync-status').innerHTML = '<span class="w-2 h-2 mr-1.5 bg-emerald-500 rounded-full"></span> 연동 완료 (' + new Date().toLocaleTimeString() + ')';
            } catch (error) {
                console.error(error);
                document.getElementById('sync-status').innerHTML = '<span class="w-2 h-2 mr-1.5 bg-rose-500 rounded-full"></span> 연동 실패';
            }
        }

        function parseDate(str) {
            if (!str) return null;
            const match = str.match(/(\d{4})[\.\-\/\s]+(\d{1,2})[\.\-\/\s]+(\d{1,2})/);
            if (match) {
                return new Date(parseInt(match[1], 10), parseInt(match[2], 10) - 1, parseInt(match[3], 10));
            }
            return null;
        }

        function parseCSVAndRender(csvText) {
            const rows = parseCSVToArray(csvText);
            if (rows.length < 2) return;

            let headerIndex = -1;
            for (let i = 0; i < rows.length; i++) {
                if (rows[i].some(cell => cell && cell.includes("관리번호"))) {
                    headerIndex = i;
                    break;
                }
            }

            if (headerIndex === -1) return;

            const dataRows = rows.slice(headerIndex + 1);

            let totalItems = 0;
            let totalDoneSteps = 0;
            let normalSteps = 0;
            let delayedSteps = 0;
            let overdueSteps = 0;

            let normalItemsCount = 0;
            let delayedItemsCount = 0;
            let overdueItemsCount = 0;

            let partnersStats = {};
            let parsedItems = [];

            const periodLabels = ['1주', '2주', '3주', '4주', '2달', '3달', '4달', '6달'];

            let i = 0;
            while (i < dataRows.length) {
                const row1 = dataRows[i];
                
                if (row1 && row1[0] && row1[0].trim() !== '' && !row1[0].includes('점검') && !row1[0].includes('기준일')) {
                    const id = row1[0].trim();

                    // 숨겨진 행 (L26_001, L26_002, L26_003) 제외
                    if (['L26_001', 'L26_002', 'L26_003'].includes(id)) {
                        i++;
                        continue;
                    }

                    const date = row1[1] || '';
                    const osa = row1[2] || (id.includes('_') ? id.split('_')[0] : '기타');
                    const client = row1[3] || '';
                    const productName = row1[4] || '';
                    const defect = row1[5] || '';
                    const idPrefix = id.includes('_') ? id.split('_')[0] : osa;

                    totalItems++;

                    if (!partnersStats[osa]) {
                        partnersStats[osa] = {
                            prefix: idPrefix,
                            items: 0,
                            doneSteps: 0,
                            normal: 0,
                            delayed: 0,
                            overdue: 0
                        };
                    }
                    partnersStats[osa].items++;

                    let rowStandard = row1;
                    let rowExecution = [];
                    
                    for (let k = 0; k <= 3; k++) {
                        if (i + k < dataRows.length) {
                            let candidate = dataRows[i + k];
                            if (k > 0 && candidate[0] && candidate[0].trim() !== '' && !candidate[0].includes('점검')) {
                                break;
                            }
                            if (candidate.some(cell => cell && cell.includes('점검 기준일'))) {
                                rowStandard = candidate;
                            }
                            if (candidate.some(cell => cell && cell.includes('점검 실시일'))) {
                                rowExecution = candidate;
                            }
                        }
                    }

                    let labelColIdx = rowStandard.findIndex(cell => cell && cell.includes('점검 기준일'));
                    if (labelColIdx === -1 && rowExecution.length > 0) {
                        labelColIdx = rowExecution.findIndex(cell => cell && cell.includes('점검 실시일'));
                    }
                    const startCol = labelColIdx !== -1 ? labelColIdx + 1 : 10;

                    let gaugeHtml = '<div class="flex flex-wrap gap-1.5 items-center">';
                    let itemHasDelayed = false;
                    let itemHasOverdue = false;

                    for (let idx = 0; idx < 8; idx++) {
                        let col = startCol + idx;
                        let stdVal = (rowStandard && rowStandard[col]) ? rowStandard[col].trim() : '';
                        let execVal = (rowExecution && rowExecution[col]) ? rowExecution[col].trim() : '';

                        if (execVal.length > 5) {
                            totalDoneSteps++;
                            partnersStats[osa].doneSteps++;

                            let dtExec = parseDate(execVal);
                            let dtStd = parseDate(stdVal);

                            let badgeColor = 'bg-emerald-500 text-white';
                            let statusText = '정상 완료';

                            if (dtExec && dtStd) {
                                let diffDays = Math.round((dtExec.getTime() - dtStd.getTime()) / (1000 * 60 * 60 * 24));

                                if (diffDays > 5) {
                                    badgeColor = 'bg-rose-500 text-white ring-2 ring-rose-200';
                                    statusText = `경과 (+${diffDays}일 지연)`;
                                    overdueSteps++;
                                    partnersStats[osa].overdue++;
                                    itemHasOverdue = true;
                                } else if (diffDays >= 3) {
                                    badgeColor = 'bg-amber-500 text-white ring-2 ring-amber-200';
                                    statusText = `지연 (+${diffDays}일 지연)`;
                                    delayedSteps++;
                                    partnersStats[osa].delayed++;
                                    itemHasDelayed = true;
                                } else {
                                    normalSteps++;
                                    partnersStats[osa].normal++;
                                    statusText = diffDays < 0 ? `정상 (${Math.abs(diffDays)}일 조기완료)` : `정상 (+${diffDays}일 이내)`;
                                }
                            } else {
                                normalSteps++;
                                partnersStats[osa].normal++;
                            }

                            let titleText = `[${periodLabels[idx]}] 기준일: ${stdVal || '-'} / 실시일: ${execVal} (${statusText})`;
                            gaugeHtml += `<span class="px-2.5 py-1 text-xs font-bold rounded-md ${badgeColor} shadow-sm cursor-help transition-transform hover:scale-105 whitespace-nowrap" title="${titleText}">${periodLabels[idx]}</span>`;
                        } else {
                            let titleText = `[${periodLabels[idx]}] 기준일: ${stdVal || '-'} (미실시)`;
                            gaugeHtml += `<span class="px-2.5 py-1 text-xs font-semibold rounded-md bg-slate-200 text-slate-500 cursor-help whitespace-nowrap" title="${titleText}">${periodLabels[idx]}</span>`;
                        }
                    }
                    gaugeHtml += '</div>';

                    if (itemHasOverdue) {
                        overdueItemsCount++;
                    } else if (itemHasDelayed) {
                        delayedItemsCount++;
                    } else {
                        normalItemsCount++;
                    }

                    parsedItems.push({
                        id, idPrefix, date, osa, client, productName, defect, gaugeHtml
                    });
                }
                i++;
            }

            allParsedData = parsedItems;

            const totalPossibleSteps = totalItems * 8;
            const completionRate = totalPossibleSteps > 0 ? Math.round((totalDoneSteps / totalPossibleSteps) * 100) : 0;
            const partnerNames = Object.keys(partnersStats);

            // 좌측 3개 KPI 업데이트
            document.getElementById('kpi-total').innerText = totalItems + " 건";
            document.getElementById('kpi-total-sub').innerText = `총 점검 주기 ${totalPossibleSteps}회`;

            document.getElementById('kpi-rate').innerText = completionRate + "%";
            document.getElementById('kpi-rate-sub').innerText = `${totalDoneSteps} / ${totalPossibleSteps} 주기 완료`;

            document.getElementById('kpi-partners').innerText = partnerNames.length + " 개사";
            document.getElementById('kpi-partners-sub').innerText = partnerNames.join(', ') || '참여 협력사 현황';

            // 우측 3개 KPI (정상 / 지연 / 경과) 업데이트
            document.getElementById('kpi-normal').innerText = normalSteps + " 건";
            document.getElementById('kpi-normal-sub').innerText = `전 주기 정상 품목 ${normalItemsCount}건`;

            document.getElementById('kpi-delayed').innerText = delayedSteps + " 건";
            document.getElementById('kpi-delayed-sub').innerText = `지연 발생 품목 ${delayedItemsCount}건`;

            document.getElementById('kpi-overdue').innerText = overdueSteps + " 건";
            document.getElementById('kpi-overdue-sub').innerText = `경과 발생 품목 ${overdueItemsCount}건`;

            renderPartnerSummary(partnersStats);
            updateFilterOptions(partnersStats);
            filterTable();
        }

        function renderPartnerSummary(partnersStats) {
            const container = document.getElementById('partner-summary');
            let html = '';

            Object.keys(partnersStats).forEach(osa => {
                const st = partnersStats[osa];
                const maxSteps = st.items * 8;
                const rate = maxSteps > 0 ? Math.round((st.doneSteps / maxSteps) * 100) : 0;

                let statusBadge = `<span class="px-2 py-0.5 text-[11px] font-bold rounded-full bg-emerald-100 text-emerald-700 whitespace-nowrap">정상 진행</span>`;
                if (st.overdue > 0) {
                    statusBadge = `<span class="px-2 py-0.5 text-[11px] font-bold rounded-full bg-rose-100 text-rose-700 whitespace-nowrap">경과 ${st.overdue}건</span>`;
                } else if (st.delayed > 0) {
                    statusBadge = `<span class="px-2 py-0.5 text-[11px] font-bold rounded-full bg-amber-100 text-amber-700 whitespace-nowrap">지연 ${st.delayed}건</span>`;
                }

                html += `
                    <div class="p-3.5 rounded-lg bg-slate-50 border border-slate-200/80 flex flex-col justify-between">
                        <div class="flex justify-between items-center mb-2">
                            <div class="flex items-center space-x-2">
                                <span class="font-bold text-sm text-indigo-950 whitespace-nowrap">${osa}</span>
                                <span class="text-xs text-slate-500 font-medium whitespace-nowrap">(${st.items}건)</span>
                            </div>
                            ${statusBadge}
                        </div>
                        <div class="w-full bg-slate-200 rounded-full h-2 mb-2 overflow-hidden">
                            <div class="bg-indigo-600 h-2 rounded-full" style="width: ${rate}%"></div>
                        </div>
                        <div class="flex justify-between items-center text-xs text-slate-600">
                            <span class="whitespace-nowrap">완료율: <strong class="text-slate-900">${rate}%</strong> (${st.doneSteps}/${maxSteps})</span>
                            <div class="space-x-1.5 text-[11px] whitespace-nowrap">
                                <span class="text-emerald-600 font-semibold">정상 ${st.normal}</span>
                                <span class="text-amber-600 font-semibold">지연 ${st.delayed}</span>
                                <span class="text-rose-600 font-semibold">경과 ${st.overdue}</span>
                            </div>
                        </div>
                    </div>
                `;
            });

            container.innerHTML = html || `<div class="text-xs text-slate-400">집계된 업체 데이터가 없습니다.</div>`;
        }

        function updateFilterOptions(partnersStats) {
            const selectEl = document.getElementById('partner-filter');
            const currentVal = selectEl.value;
            let optionsHtml = '<option value="ALL">전체 업체 보기</option>';
            Object.keys(partnersStats).forEach(osa => {
                const prefix = partnersStats[osa].prefix;
                const label = prefix !== osa ? `${osa} (${prefix}_*)` : osa;
                optionsHtml += `<option value="${osa}">${label}</option>`;
            });
            selectEl.innerHTML = optionsHtml;
            if (Object.keys(partnersStats).includes(currentVal)) {
                selectEl.value = currentVal;
            } else {
                selectEl.value = 'ALL';
            }
        }

        function filterTable() {
            const selectedPartner = document.getElementById('partner-filter').value;
            if (selectedPartner === 'ALL') {
                renderTable(allParsedData);
            } else {
                const filtered = allParsedData.filter(item => item.osa === selectedPartner || item.idPrefix === selectedPartner);
                renderTable(filtered);
            }
        }

        function renderTable(items) {
            let tableHtml = '';
            items.forEach(item => {
                tableHtml += `
                    <tr class="hover:bg-slate-50/50 transition-colors">
                        <td class="p-3.5 font-bold text-slate-900 bg-white whitespace-nowrap" rowspan="2" style="vertical-align: middle;">${item.id}</td>
                        <td class="p-3 text-slate-600 text-xs whitespace-nowrap">${item.date}</td>
                        <td class="p-3 font-semibold text-indigo-900 text-xs whitespace-nowrap">${item.osa}</td>
                        <td class="p-3 text-slate-700 font-medium text-xs whitespace-nowrap">${item.client}</td>
                        <td class="p-3 text-slate-700 text-xs">${item.productName}</td>
                        <td class="p-3 text-slate-600 text-xs">${item.defect}</td>
                    </tr>
                    <tr class="hover:bg-slate-50/50 transition-colors bg-slate-50/70 border-b border-slate-200">
                        <td class="p-3" colspan="5">
                            <div class="flex flex-wrap items-center gap-3">
                                <span class="text-xs font-bold text-slate-500 whitespace-nowrap">기간별 진행 상태:</span>
                                ${item.gaugeHtml}
                            </div>
                        </td>
                    </tr>
                `;
            });

            document.getElementById('table-body').innerHTML = tableHtml || `<tr><td colspan="6" class="p-6 text-center text-slate-400">조건에 해당하는 데이터가 없습니다.</td></tr>`;
        }

        function parseCSVToArray(str) {
            let arr = [];
            let row = [];
            let inQuotes = false;
            let c = '';
            let val = '';

            for (let i = 0; i < str.length; i++) {
                c = str[i];
                let nextC = str[i + 1];

                if (c === '"') {
                    if (inQuotes && nextC === '"') {
                        val += '"';
                        i++;
                    } else {
                        inQuotes = !inQuotes;
                    }
                } else if (c === ',' && !inQuotes) {
                    row.push(val.trim());
                    val = '';
                } else if ((c === '\r' || c === '\n') && !inQuotes) {
                    if (c === '\r' && nextC === '\n') { i++; }
                    row.push(val.trim());
                    arr.push(row);
                    row = [];
                    val = '';
                } else {
                    val += c;
                }
            }
            if (val !== '' || row.length > 0) {
                row.push(val.trim());
                arr.push(row);
            }
            return arr;
        }
    </script>
</body>
</html>
