<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Expense Tracker</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
        }

        header {
            background-color: #333;
            color: #fff;
            text-align: center;
            padding: 1em;
        }

        #expense-form {
            max-width: 400px;
            margin: 2em auto;
            padding: 1em;
            background-color: #fff;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }

        label {
            display: block;
            margin-bottom: 0.5em;
        }

        input, select, button {
            width: 100%;
            padding: 0.5em;
            margin-bottom: 1em;
            box-sizing: border-box;
        }

        button {
            background-color: #007bff;
            color: #fff;
            border: none;
            padding: 0.5em;
            cursor: pointer;
        }

        button:hover {
            background-color: #0056b3;
        }

        #expense-list,
        #expense-table,
        #total-expense {
            margin-top: 1em;
        }

        #expense-list li,
        #expense-table tbody tr {
            background-color: #e9e9e9;
            border-radius: 4px;
            margin-bottom: 8px;
            padding: 0.5em;
        }

        .delete-button {
            background-color: #dc3545;
            color: #fff;
            border: none;
            padding: 4px 8px;
            cursor: pointer;
        }

        .delete-button:hover {
            background-color: #bd2130;
        }

        #charts {
            max-width: 800px;
            margin: 2em auto;
            padding: 1em;
            background-color: #fff;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }

        canvas {
            display: block;
            margin: 1em 0;
            border: 1px solid #ddd;
            border-radius: 8px;
        }
    </style>
</head>
<body>
    <header>
        <h1>Expense Tracker</h1>
    </header>
    <div id="expense-form">
        <label for="expense-description">Expense Description:</label>
        <input type="text" id="expense-description" placeholder="E.g., Grocery">

        <label for="expense-category">Expense Category:</label>
        <select id="expense-category">
            <option value="food">Food</option>
            <option value="junk">Junk</option>
            <option value="traveling">Traveling</option>
            <option value="household">Household</option>
            <option value="essentials">Essentials</option>
            <option value="useless">Useless Expense</option>
        </select>

        <label for="expense-amount">Expense Amount:</label>
        <input type="number" id="expense-amount" placeholder="E.g., 50">

        <button onclick="addExpense()">Add Expense</button>
        <button onclick="deleteAllExpenses()">Delete All Expenses</button>
        <button onclick="updateCharts()">Update Charts</button>

        <div id="total-expense"></div>

        <ul id="expense-list"></ul>

        <table id="expense-table">
            <thead>
                <tr>
                    <th>Description</th>
                    <th>Category</th>
                    <th>Amount (₹)</th>
                    <th>Date</th>
                    <th>Action</th>
                </tr>
            </thead>
            <tbody id="expense-table-body"></tbody>
        </table>
    </div>

    <div id="charts">
        <h2>Expense Distribution</h2>
        <canvas id="expense-pie-chart" width="400" height="400"></canvas>

        <h2>Monthly and Weekly Expenses</h2>
        <canvas id="expense-line-chart" width="400" height="200"></canvas>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script>
        // Load expenses from localStorage
        const expenses = JSON.parse(localStorage.getItem('expenses')) || [];

        function addExpense() {
            const description = document.getElementById('expense-description').value;
            const category = document.getElementById('expense-category').value;
            const amount = document.getElementById('expense-amount').value;
            const date = new Date();

            if (description && category && amount) {
                const expense = {
                    description: description,
                    category: category,
                    amount: parseFloat(amount),
                    date: date
                };

                expenses.push(expense);

                // Update the UI
                updateExpenseList();
                updateExpenseTable();
                updateTotalExpense();
                updateCharts();

                // Save expenses to localStorage
                localStorage.setItem('expenses', JSON.stringify(expenses));
            }
        }

        function deleteExpense(index) {
            expenses.splice(index, 1);

            // Update the UI
            updateExpenseList();
            updateExpenseTable();
            updateTotalExpense();
            updateCharts();

            // Save expenses to localStorage
            localStorage.setItem('expenses', JSON.stringify(expenses));
        }

        function deleteAllExpenses() {
            expenses.length = 0;

            // Update the UI
            updateExpenseList();
            updateExpenseTable();
            updateTotalExpense();
            updateCharts();

            // Save expenses to localStorage
            localStorage.setItem('expenses', JSON.stringify(expenses));
        }

        function updateExpenseList() {
            const expenseList = document.getElementById('expense-list');
            expenseList.innerHTML = "";

            // expenses.forEach((expense, index) => {
            //     const listItem = document.createElement('li');
            //     listItem.innerHTML = `
            //         <span>${expense.description} (${expense.category}): ₹${expense.amount.toFixed(2)} - ${expense.date.toLocaleString()}</span>
            //         <button class="delete-button" onclick="deleteExpense(${index})">Delete</button>
            //     `;
            //     expenseList.appendChild(listItem);
            // });
        }

        function updateExpenseTable() {
            const expenseTableBody = document.getElementById('expense-table-body');
            expenseTableBody.innerHTML = "";

            expenses.forEach((expense, index) => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${expense.description}</td>
                    <td>${expense.category}</td>
                    <td>₹${expense.amount.toFixed(2)}</td>
                    <td>${expense.date.toLocaleString()}</td>
                    <td><button class="delete-button" onclick="deleteExpense(${index})">Delete</button></td>
                `;
                expenseTableBody.appendChild(row);
            });
        }

        function updateTotalExpense() {
            const totalExpense = expenses.reduce((total, expense) => total + expense.amount, 0).toFixed(2);
            document.getElementById('total-expense').innerHTML = `<strong>Total Expense: ₹${totalExpense}</strong>`;
        }

        function updateCharts() {
            updateExpenseDistributionChart();
            updateMonthlyWeeklyExpenseChart();
        }

        function updateExpenseDistributionChart() {
            const categories = Array.from(new Set(expenses.map(expense => expense.category)));
            const categoryExpenses = categories.map(category =>
                expenses.reduce((total, expense) => expense.category === category ? total + expense.amount : total, 0)
            );

            const pieChartCanvas = document.getElementById('expense-pie-chart');
            const pieChartContext = pieChartCanvas.getContext('2d');

            new Chart(pieChartContext, {
                type: 'pie',
                data: {
                    labels: categories,
                    datasets: [{
                        data: categoryExpenses,
                        backgroundColor: ['#ff9999', '#66b3ff', '#99ff99', '#ffcc99', '#c2c2f0', '#ffb3e6']
                    }]
                }
            });
        }

        function updateMonthlyWeeklyExpenseChart() {
            // Add your logic to calculate monthly and weekly expenses

            const monthlyExpenses = [100, 150, 200, 120];  // Sample data
            const weeklyExpenses = [30, 50, 40, 20, 60];    // Sample data

            const lineChartCanvas = document.getElementById('expense-line-chart');
            const lineChartContext = lineChartCanvas.getContext('2d');

            new Chart(lineChartContext, {
                type: 'line',
                data: {
                    labels: ['Week 1', 'Week 2', 'Week 3', 'Week 4'],
                    datasets: [{
                        label: 'Monthly Expenses',
                        data: monthlyExpenses,
                        borderColor: 'rgba(255, 99, 132, 1)',
                        borderWidth: 2,
                        fill: false
                    }, {
                        label: 'Weekly Expenses',
                        data: weeklyExpenses,
                        borderColor: 'rgba(54, 162, 235, 1)',
                        borderWidth: 2,
                        fill: false
                    }]
                }
            });
        }

        // Initial rendering
        updateExpenseList();
        updateExpenseTable();
        updateTotalExpense();
        updateCharts();
    </script>
</body>
</html>

