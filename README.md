<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>상부3조 가계부 웹사이트</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.0.1/chart.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.0.1/chart.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 800px;
            margin: auto;
        }
        .hidden {
            display: none;
        }
        .chart-container {
            margin-top: 20px;
        }
        .date-control {
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>상부3조 가계부</h1>

        <!-- 날짜 이동 -->
        <div class="date-control">
            <button onclick="prevDay()">이전 날</button>
            <span id="current-date"></span>
            <button onclick="nextDay()">다음 날</button>
        </div>

        <!-- 초기 계좌 잔액 입력 -->
        <div id="initial-balance-section">
            <label for="initial-balance">초기 계좌 잔액 입력:</label>
            <input type="number" id="initial-balance" placeholder="0" />
            <button onclick="setInitialBalance()">초기 입력</button>
        </div>

        <!-- 일일 수입/지출 입력 -->
        <div id="daily-income-expense">
            <h2>일일 추가 수입/지출</h2>
            <label for="daily-income">일일 추가 수입:</label>
            <input type="number" id="daily-income" placeholder="0" />
            <label for="daily-expense">일일 추가 지출:</label>
            <input type="number" id="daily-expense" placeholder="0" />
            <button onclick="addDailyTransaction()">저장</button>
        </div>

        <!-- 현재 남은 금액 표시 -->
        <h2>현재 남은 금액: <span id="current-balance">0</span>원</h2>


        <!-- 월별 통계와 그래프 -->
        <h2>월별 통계</h2>
        <canvas id="monthly-chart" class="chart-container"></canvas>

        <!-- 가계부 종료 버튼 -->
        <button onclick="endBudgetTracking()">가계부 작성 종료</button>

        <!-- 결과 표 표시 -->
        <div id="result-summary" class="hidden">
            <h2>1년간의 가계부 요약</h2>
            <table border="1">
                <thead>
                    <tr>
                        <th>월</th>
                        <th>총 수입</th>
                        <th>총 지출</th>
                        <th>잔액</th>
                    </tr>
                </thead>
                <tbody id="result-table-body"></tbody>
            </table>
        </div>
    </div>

    <script>
        let balance = 0;
        let currentDate = new Date();
        let dailyData = {}; // 날짜별 데이터 저장
        let monthlyData = Array.from({ length: 12 }, () => ({ income: 0, expense: 0, startOfMonth: true })); // 월별 수입/지출 데이터 초기화

        function formatDate(date) {
            return date.toISOString().split('T')[0];
        }

        function updateDateDisplay() {
            document.getElementById('current-date').textContent = formatDate(currentDate);
            const dateKey = formatDate(currentDate);
            const daily = dailyData[dateKey] || { income: 0, expense: 0 };

            document.getElementById('daily-income-display').textContent = daily.income;
            document.getElementById('daily-expense-display').textContent = daily.expense;
        }

        function prevDay() {
            currentDate.setDate(currentDate.getDate() - 1);
            updateDateDisplay();
        }

        function nextDay() {
            currentDate.setDate(currentDate.getDate() + 1);
            updateDateDisplay();
        }

        function setInitialBalance() {
            const input = document.getElementById('initial-balance');
            balance = parseFloat(input.value);
            if (isNaN(balance) || balance < 0) {
                alert('유효한 금액을 입력하세요.');
                return;
            }
            document.getElementById('current-balance').textContent = balance;
            document.getElementById('initial-balance-section').classList.add('hidden');
        }

        function addDailyTransaction() {
            const income = parseFloat(document.getElementById('daily-income').value) || 0;
            const expense = parseFloat(document.getElementById('daily-expense').value) || 0;
            const dateKey = formatDate(currentDate);

            // 오늘의 데이터를 업데이트 (누적되는 방식으로)
            if (!dailyData[dateKey]) {
                dailyData[dateKey] = { income: 0, expense: 0 };
            }

            // 누적값을 더해줌
            dailyData[dateKey].income += income;
            dailyData[dateKey].expense += expense;

            // 월별 데이터 업데이트
            const month = currentDate.getMonth();
            monthlyData[month].income += income;
            monthlyData[month].expense += expense;

            // 잔액 업데이트
            updateBalance(income - expense);
            updateDateDisplay();
        }

        function updateBalance(amount) {
            balance += amount;
            document.getElementById('current-balance').textContent = balance;
            updateMonthlyChart();
        }

        function updateMonthlyChart() {
            const ctx = document.getElementById('monthly-chart').getContext('2d');
            const incomeData = monthlyData.map((data) => data.income);
            const expenseData = monthlyData.map((data) => data.expense);

            new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: Array.from({ length: 12 }, (_, i) => `${i + 1}월`),
                    datasets: [
                        {
                            label: '총 수입',
                            data: incomeData,
                            backgroundColor: 'green',
                        },
                        {
                            label: '총 지출',
                            data: expenseData,
                            backgroundColor: 'red',
                        },
                    ],
                },
            });
        }

        function endBudgetTracking() {
            const tableBody = document.getElementById('result-table-body');
            tableBody.innerHTML = '';

            // 각 월별로 총 수입과 총 지출을 계산하여 표에 표시
            monthlyData.forEach((data, index) => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${index + 1}월</td>
                    <td>${data.income}원</td>
                    <td>${data.expense}원</td>
                    <td>${data.income - data.expense}원</td>
                `;
                tableBody.appendChild(row);

                // 가계수지 지표 계산 (지출 / 수입)
                const expenseRatio = data.income !== 0 ? (data.expense / data.income) * 100 : 0;
                const guideMessage = expenseRatio <= 70 ? "소비가 적절합니다." : "과소비하였습니다.";

                // 각 월에 대한 소비 상태 메시지 추가
                const messageRow = document.createElement('tr');
                messageRow.innerHTML = `
                    <td colspan="4">${index + 1}월: ${guideMessage}</td>
                `;
                tableBody.appendChild(messageRow);
            });

            document.getElementById('result-summary').classList.remove('hidden');
        }

function endBudgetTracking() {
    const tableBody = document.getElementById('result-table-body');
    tableBody.innerHTML = '';

    // 각 월별로 총 수입과 총 지출을 계산하여 표에 표시
    monthlyData.forEach((data, index) => {
        const row = document.createElement('tr');
        const prevIncome = index > 0 ? monthlyData[index - 1].income : 0;
        const prevExpense = index > 0 ? monthlyData[index - 1].expense : 0;

        // 변화율 계산
        const incomeChangeRate = prevIncome !== 0 ? ((data.income - prevIncome) / prevIncome) * 100 : 0;
        const expenseChangeRate = prevExpense !== 0 ? ((data.expense - prevExpense) / prevExpense) * 100 : 0;

        row.innerHTML = `
            <td>${index + 1}월</td>
            <td>${data.income}원</td>
            <td>${data.expense}원</td>
            <td>${data.income - data.expense}원</td>
            <td>수입 변화율: ${incomeChangeRate.toFixed(2)}%</td>
            <td>지출 변화율: ${expenseChangeRate.toFixed(2)}%</td>
        `;
        tableBody.appendChild(row);

        // 가계수지 지표 계산 (총 수입 / 총 지출)
        const expenseRatio = data.income !== 0 ? (data.expense / data.income) * 100 : 0;
        const guideMessage = expenseRatio <= 70 ? "소비가 적절합니다." : "과소비하였습니다.";

        // 각 월에 대한 소비 상태 메시지 추가
        const messageRow = document.createElement('tr');
        messageRow.innerHTML = `
            <td colspan="6">${index + 1}월: 가계수지 지표 = ${expenseRatio.toFixed(2)}%. ${guideMessage}</td>
        `;
        tableBody.appendChild(messageRow);
    });

    // 결과 표 표시
    document.getElementById('result-summary').classList.remove('hidden');
}
        // 초기 화면 업데이트
        updateDateDisplay();
    </script>
</body>
</html>

