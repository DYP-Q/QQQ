<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>개선대책 유효성 점검 대시보드</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body .markdown-body h1, body > h1:first-child { display: none !important; }
        
        /* 상단 파란색 박스 헤더 스타일 */
        .dashboard-header {
            background-color: #262272;
            color: #ffffff;
            padding: 24px 36px;
            border-radius: 12px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            gap: 24px;
            margin-bottom: 28px;
            box-sizing: border-box;
        }
        .header-left {
            display: flex;
            align-items: baseline;
            gap: 20px;
            flex-wrap: nowrap;
            overflow: hidden;
        }
        .header-title {
            font-size: 28px;
            font-weight: 800;
            margin: 0;
            white-space: nowrap;
            letter-spacing: -0.5px;
        }
        .header-subtitle {
            font-size: 15px;
            color: #c7d2fe;
            margin: 0;
            white-space: nowrap;
        }
        .header-right {
            display: flex;
            align-items: center;
            gap: 12px;
            flex-shrink: 0;
        }
        .btn-sheet {
            background-color: rgba(255, 255, 255, 0.12);
            color: #ffffff;
            border: 1px solid rgba(255, 255, 255, 0.4);
            padding: 10px 18px;
            border-radius: 8px;
            font-size: 14px;
            font-weight: 600;
            text-decoration: none;
            display: inline-flex;
            align-items: center;
            gap: 8px;
            white-space: nowrap;
            cursor: pointer;
            transition: background-color 0.2s;
        }
        .btn-sheet:hover {
            background-color: rgba(255, 255, 255, 0.22);
        }
        .sync-badge {
            background-color: #dcfce7;
            color: #15803d;
            padding: 10px 18px;
            border-radius: 9999px;
            font-size: 14px;
            font-weight: 700;
            display: inline-flex;
            align-items: center;
            gap: 8px;
            white-space: nowrap;
        }
        .sync-dot {
            width: 8px;
            height: 8px;
            background-color: #16a34a;
            border-radius: 50%;
            display: inline-block;
        }

        /* 현황판 6개 폰트 50% 확대 스타일 */
        .kpi-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 20px;
            margin-bottom: 30px;
        }
        .kpi-card {
            background: #ffffff;
            border: 1px solid #e5e7eb;
            border-radius: 16px;
            padding: 28px 32px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            box-shadow: 0 2px 6px rgba(0, 0, 0, 0.03);
        }
        .kpi-left {
            display: flex;
            align-items: center;
            gap: 20px;
        }
        .kpi-icon-box {
            width: 66px;
            height: 66px;
            border-radius: 14px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 32px;
            flex-shrink: 0;
        }
        .kpi-text-group {
            display: flex;
            flex-direction: column;
            gap: 8px;
        }
        .kpi-title {
            font-size: 21px;
            font-weight: 700;
            margin: 0;
            display: flex;
            align-items: center;
            gap: 8px;
            white-space: nowrap;
        }
        .kpi-desc {
            font-size: 18px;
            color: #6b7280;
            margin: 0;
            white-space: nowrap;
        }
        .kpi-value {
            font-size: 36px;
            font-weight: 800;
            white-space: nowrap;
        }
        .status-dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            display: inline-block;
        }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen">

    <!-- 상단 파란색 박스 헤더 -->
    <header class="dashboard-header max-w-7xl mx-auto my-6">
        <div class="header-left">
            <h1 class="header-title">개선대책 유효성 점검 현황</h1>
            <p class="header-subtitle">주기별 자주 검증 진척도 및 현장 품질 관리 실시간 모니터링</p>
        </div>
        <div class="header-right">
            <a href="https://docs.google.com/spreadsheets/d/1CAAK11qrimDmHXvii7zRcUaWFTY6ENMT-LB7q5nrJz4/edit?gid=359651245#gid=359651245" 
               target="_blank" class="btn-sheet">
                📊 상세 원본 시트 열기
            </a>
            <div class="sync-badge">
                <span class="sync-dot"></span>
                <span id="sync-status">연동 중</span>
            </div>
        </div>
    </header>

    <!-- Main Container -->
    <main class="max-w-7xl mx-auto px-4 space-y-6">

        <!-- 현황판 6개 (좌측 3개 / 우측 3개 수직 배치, 글자 크기 50% 확대) -->
        <div class="kpi-grid">
            
            <!-- 좌측 컬럼 -->
            <div class="flex flex-col space-y-5">
                <!-- 1. 전체 관리 항목 -->
                <div class="kpi-card">
                    <div class="kpi-left">
                        <div class="kpi-icon-box bg-slate-100">📋</div>
                        <div class="kpi-text-group">
                            <h3 class="kpi-title text-slate-800">전체 관리 항목</h3>
                            <p class="kpi-desc" id="kpi-total-sub">총 점검 주기 0회</p>
                        </div>
                    </div>
                    <div class="kpi-value text-slate-900" id="kpi-total">0 건</div>
                </div>

                <!-- 2. 전체 점검 완료율 -->
                <div class="kpi-card">
                    <div class="kpi-left">
                        <div class="kpi-icon-box bg-blue-50">📈</div>
                        <div class="kpi-text-group">
                            <h3 class="kpi-title text-slate-800">전체 점검 완료율</h3>
                            <p class="kpi-desc" id="kpi-rate-sub">0 / 0 주기 완료</p>
                        </div>
                    </div>
                    <div class="kpi-value text-indigo-600" id="kpi-rate">0%</div>
                </div>

                <!-- 3. 대상 업체 수 -->
                <div class="kpi-card">
                    <div class="kpi-left">
                        <div class="kpi-icon-box bg-slate-100">🏢</div>
                        <div class="kpi-text-group">
                            <h3 class="kpi-title text-slate-800">대상 업체 수</h3>
                            <p class="kpi-desc" id="kpi-partners-sub">참여 협력사 현황</p>
                        </div>
                    </div>
                    <div class="kpi-value text-slate-800" id="kpi-partners">0 개사</div>
                </div>
            </div>

            <!-- 우측 컬럼 -->
            <div class="flex flex-col space-y-5">
                <!-- 4. 정상 (초록색) -->
                <div class="kpi-card border-emerald-200">
                    <div class="kpi-left">
                        <div class="kpi-icon-box bg-emerald-50 text-emerald-600">✅</div>
                        <div class="kpi-text-group">
                            <h3 class="kpi-title text-emerald-800">
                                <span class="status-dot" style="background-color: #10b981;"></span>
                                정상 (기준일 대비 2일 이내)
                            </h3>
                            <p class="kpi-desc" id="kpi-normal-sub">전 주기 정상 품목 0건</p>
                        </div>
                    </div>
                    <div class="kpi-value text-emerald-600" id="kpi-normal">0 건</div>
                </div>

                <!-- 5. 지연 (주황색) -->
                <div class="kpi-card border-amber-200">
                    <div class="kpi-left">
                        <div class="kpi-icon-box bg-amber-50 text-amber-500">⚠️</div>
                        <div class="kpi-text-group">
                            <h3 class="kpi-title text-amber-800">
                                <span class="status-dot" style="background-color: #d97706;"></span>
                                지연 (기준일 대비 3~5일 이내)
                            </h3>
                            <p class="kpi-desc" id="kpi-delayed-sub">지연 발생 품목 0건</p>
                        </div>
                    </div>
                    <div class="kpi-value text-amber-500" id="kpi-delayed">0 건</div>
                </div>

                <!-- 6. 경과 (빨간색) -->
                <div class="kpi-card border-rose-200">
                    <div class="kpi-left">
                        <div class="kpi-icon-box bg-rose-50 text-rose-600">🚨</div>
                        <div class="kpi-text-group">
                            <h3 class="kpi-title text-rose-800">
                                <span class="status-dot" style="background-color: #e11d48;"></span>
                                경과 (기준일 대비 5일 초과)
                            </h3>
                            <p class="kpi-desc" id="kpi-overdue-sub">경과 발생 품목 0건</p>
                        </div>
                    </div>
                    <div class="kpi-value text-rose-600" id="kpi-overdue">0 건</div>
                </div>
            </div>

        </div>

        <!-- 업체별 요약 현황 카드 -->
        <div class="bg-white p-5 rounded-xl shadow-sm border border-slate-100">
            <h3 class="text-sm font-bold text-slate-800 mb-3">업체별(OSA) 점검 진행 및 지연 요약</h3>
            <div id="partner-summary" class="grid grid-cols-1 md:grid-cols-3 gap-3">
                <!-- JS로 동적 생성 -->
            </div>
        </div>

        <!-- Detailed Table Section -->
        <div class="bg-white rounded-xl shadow-sm border border-slate-100 overflow-hidden">
            <div class="p-5 border-b border-slate-100 flex flex-col sm:flex-row justify-between items-center gap-3">
                <div>
                    <h3 class="text-base font-bold text-slate-900">개선 대책 상세 진행 목록</h3>
                    <p class="text-xs text-slate-400 mt-0.5">* 색상 기준: 초록(정상, ≤2일), 주황(지연, 3~5일), 빨강(경과, &gt;5일)</p>
                </div>
                <!-- C열 업체명 기준 필터 셀렉트박스 -->
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

                document.getElementById('sync-status').textContent = '연동 완료 (' + new Date().toLocaleTimeString() + ')';
            } catch (error) {
                console.error(error);
                document.getElementById('sync-status').textContent = '연동 실패';
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

                    // 숨겨진 행 제외
                    if (['L26_001', 'L26_002', 'L26_003'].includes(id)) {
                        i++;
                        continue;
                    }

                    const date = row1[1] || '';
                    // C열(인덱스 2)의 값을 OSA 업체명으로 사용
                    const osa = (row1[2] ? row1[2].trim() : '') || (id.includes('_') ? id.split('_')[0] : '기타');
                    const client = row1[3] || '';
                    const productName = row1[4] || '';
                    const defect = row1[5] || '';

                    totalItems++;

                    if (!partnersStats[osa]) {
                        partnersStats[osa] = {
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
                        id, date, osa, client, productName, defect, gaugeHtml
                    });
                }
                i++;
            }

            allParsedData = parsedItems;

            const totalPossibleSteps = totalItems * 8;
            const completionRate = totalPossibleSteps > 0 ? Math.round((totalDoneSteps / totalPossibleSteps) * 100) : 0;
            const partnerNames = Object.keys(partnersStats);

            document.getElementById('kpi-total').textContent = totalItems + " 건";
            document.getElementById('kpi-total-sub').textContent = `총 점검 주기 ${totalPossibleSteps}회`;

            document.getElementById('kpi-rate').textContent = completionRate + "%";
            document.getElementById('kpi-rate-sub').textContent = `${totalDoneSteps} / ${totalPossibleSteps} 주기 완료`;

            document.getElementById('kpi-partners').textContent = partnerNames.length + " 개사";
            document.getElementById('kpi-partners-sub').textContent = partnerNames.join(', ') || '참여 협력사 현황';

            document.getElementById('kpi-normal').textContent = normalSteps + " 건";
            document.getElementById('kpi-normal-sub').textContent = `전 주기 정상 품목 ${normalItemsCount}건`;

            document.getElementById('kpi-delayed').textContent = delayedSteps + " 건";
            document.getElementById('kpi-delayed-sub').textContent = `지연 발생 품목 ${delayedItemsCount}건`;

            document.getElementById('kpi-overdue').textContent = overdueSteps + " 건";
            document.getElementById('kpi-overdue-sub').textContent = `경과 발생 품목 ${overdueItemsCount}건`;

            renderPartnerSummary(partnersStats);
            updateFilterOptions(partnerNames);
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

        function updateFilterOptions(partnerNames) {
            const selectEl = document.getElementById('partner-filter');
            const currentVal = selectEl.value;
            let optionsHtml = '<option value="ALL">전체 업체 보기</option>';
            partnerNames.forEach(name => {
                optionsHtml += `<option value="${name}">${name}</option>`;
            });
            selectEl.innerHTML = optionsHtml;
            if (partnerNames.includes(currentVal)) {
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
                const filtered = allParsedData.filter(item => item.osa === selectedPartner);
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
    </script>
</body>
</html>
