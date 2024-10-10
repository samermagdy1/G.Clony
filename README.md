# G.Clony
<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>إدارة حسابات الموردين</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            margin: 0;
            padding: 0;
        }
        header {
            background: #333;
            color: #fff;
            padding: 10px 0;
            text-align: center;
        }
        .container {
            width: 90%;
            max-width: 800px;
            margin: 20px auto;
            background: #fff;
            padding: 20px;
            border-radius: 5px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        button {
            padding: 10px;
            margin: 5px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            background-color: #5cb85c;
            color: white;
            transition: background-color 0.3s;
        }
        button:hover {
            background-color: #4cae4c;
        }
        input[type="text"], input[type="number"] {
            width: calc(100% - 22px);
            padding: 10px;
            margin: 5px 0;
            border: 1px solid #ccc;
            border-radius: 5px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
            font-size: 14px;
        }
        th, td {
            padding: 10px;
            border: 1px solid #ccc;
            text-align: center;
        }
        th {
            background-color: #f2f2f2;
        }
        .print-btn {
            background-color: #007bff;
        }
        .edit-btn {
            background-color: #f0ad4e;
        }
        .delete-btn {
            background-color: #d9534f;
        }
        .search-container {
            display: flex;
            align-items: center;
        }
        .search-container input {
            flex: 1;
        }
        @media (max-width: 600px) {
            button {
                width: 100%;
                margin: 5px 0;
            }
            button.bs {
                width: 20%;
                margin: 10px 0;
            }
            .search-container button {
                padding: 9px;
                font-size: 12px;
            }
            table {
                font-size: 12px;
            }
            th, td {
                padding: 8px;
            }
        }
        @media print {
            body {
                margin: 0;
            }
            header {
                display: none;
            }
            button {
                display: none;
            }
            table {
                border: 1px solid #000;
                margin: 20px 0;
            }
            th, td {
                border: 1px solid #000;
            }
        }
    </style>
</head>
<body>

<header>
    <h1>إدارة حسابات الموردين - جورج كلوني</h1>
</header>

<div class="container" id="content">
    <h2>الصفحة الرئيسية</h2>
    <button onclick="showAddInvoice()">إضافة فاتورة</button>
    <button onclick="showAddPayment()">دفع</button>
    <button onclick="showAddSupplier()">إضافة مورد جديد</button>
    <button onclick="showSupplierSummary()">عرض ملخص الموردين</button>

    <div id="searchSupplier" style="margin-top: 20px;">
        <h3>البحث عن مورد</h3>
        <div class="search-container">
            <input type="text" id="supplierSearch" placeholder="ادخل اسم المورد">
            <button class="bs" onclick="searchSupplier()">🔍</button>
        </div>
        <div id="supplierAccountTable"></div>
    </div>
    
    <div id="supplierSummary" style="margin-top: 20px;"></div>
</div>

<script>
    let suppliers = JSON.parse(localStorage.getItem('suppliers')) || [];

    function updateLocalStorage() {
        localStorage.setItem('suppliers', JSON.stringify(suppliers));
    }

    function showAddSupplier() {
        const name = prompt("ادخل اسم المورد:");
        const openingBalance = parseFloat(prompt("ادخل الرصيد الافتتاحي:"));
        if (name && !isNaN(openingBalance)) {
            suppliers.push({ name, openingBalance, currentBalance: openingBalance, invoices: [], payments: [] });
            updateLocalStorage();
            alert("تمت إضافة المورد بنجاح");
        }
    }

    function showAddInvoice() {
        const supplierName = prompt("ادخل اسم المورد لإضافة فاتورة:");
        const amount = parseFloat(prompt("ادخل قيمة الفاتورة:"));
        const supplier = suppliers.find(s => s.name === supplierName);
        if (supplier && !isNaN(amount)) {
            const dateTime = new Date().toLocaleString();
            supplier.invoices.push({ amount, date: dateTime });
            supplier.currentBalance += amount;
            updateLocalStorage();
            alert("تمت إضافة الفاتورة بنجاح");
        } else {
            alert("لم يتم العثور على المورد أو قيمة غير صحيحة");
        }
    }

    function showAddPayment() {
        const supplierName = prompt("ادخل اسم المورد لدفع:");
        const amount = parseFloat(prompt("ادخل قيمة الدفع:"));
        const supplier = suppliers.find(s => s.name === supplierName);
        if (supplier && !isNaN(amount)) {
            const dateTime = new Date().toLocaleString();
            supplier.payments.push({ amountPaid: amount, date: dateTime });
            supplier.currentBalance -= amount;
            updateLocalStorage();
            alert("تمت إضافة الدفع بنجاح");
        } else {
            alert("لم يتم العثور على المورد أو قيمة غير صحيحة");
        }
    }

    function searchSupplier() {
        const name = document.getElementById('supplierSearch').value;
        const supplier = suppliers.find(s => s.name === name);
        if (supplier) {
            let table = `
                <table>
                    <thead>
                        <tr>
                            <th>التاريخ</th>
                            <th>الفاتورة</th>
                            <th>الدفع</th>
                            <th>الإجراءات</th>
                        </tr>
                    </thead>
                    <tbody>
            `;
            supplier.invoices.forEach((inv, index) => {
                table += `
                    <tr>
                        <td>${inv.date}</td>
                        <td>${inv.amount}</td>
                        <td>-</td>
                        <td>
                            <button class="edit-btn" onclick="editInvoice('${supplier.name}', ${index})">تعديل</button>
                            <button class="delete-btn" onclick="deleteInvoice('${supplier.name}', ${index})">حذف</button>
                        </td>
                    </tr>
                `;
            });

            supplier.payments.forEach((pay, index) => {
                table += `
                    <tr>
                        <td>${pay.date}</td>
                        <td>-</td>
                        <td>${pay.amountPaid}</td>
                        <td>
                            <button class="edit-btn" onclick="editPayment('${supplier.name}', ${index})">تعديل</button>
                            <button class="delete-btn" onclick="deletePayment('${supplier.name}', ${index})">حذف</button>
                        </td>
                    </tr>
                `;
            });

            table += `
                <tr>
                    <td colspan="2"><strong>الرصيد الحالي</strong></td>
                    <td colspan="1"><strong>${supplier.currentBalance}</strong></td>
                    <td></td>
                </tr>
            `;
            
            table += `
                </tbody>
                </table>
                <button class="print-btn" onclick="printReport('${name}')">طباعة الحساب</button>
                <button class="edit-btn" onclick="editSupplier('${name}')">تعديل المورد</button>
                <button class="delete-btn" onclick="deleteSupplier('${name}')">حذف المورد</button>
            `;
            document.getElementById('supplierAccountTable').innerHTML = table;
        } else {
            document.getElementById('supplierAccountTable').innerHTML = '<p>لم يتم العثور على مورد بهذا الاسم.</p>';
        }
    }

    function editInvoice(supplierName, invoiceIndex) {
        const supplier = suppliers.find(s => s.name === supplierName);
        if (supplier) {
            const newAmount = parseFloat(prompt("ادخل القيمة الجديدة للفاتورة:", supplier.invoices[invoiceIndex].amount));
            if (!isNaN(newAmount)) {
                supplier.currentBalance -= supplier.invoices[invoiceIndex].amount;
                supplier.invoices[invoiceIndex].amount = newAmount;
                supplier.currentBalance += newAmount;
                updateLocalStorage();
                alert("تم تعديل الفاتورة بنجاح");
                searchSupplier();
            } else {
                alert("قيمة غير صحيحة");
            }
        }
    }

    function editPayment(supplierName, paymentIndex) {
        const supplier = suppliers.find(s => s.name === supplierName);
        if (supplier) {
            const newAmount = parseFloat(prompt("ادخل القيمة الجديدة للدفع:", supplier.payments[paymentIndex].amountPaid));
            if (!isNaN(newAmount)) {
                supplier.currentBalance += supplier.payments[paymentIndex].amountPaid;
                supplier.payments[paymentIndex].amountPaid = newAmount;
                supplier.currentBalance -= newAmount;
                updateLocalStorage();
                alert("تم تعديل الدفع بنجاح");
                searchSupplier();
            } else {
                alert("قيمة غير صحيحة");
            }
        }
    }

    function editSupplier(supplierName) {
        const supplier = suppliers.find(s => s.name === supplierName);
        if (supplier) {
            const newName = prompt("ادخل الاسم الجديد للمورد:", supplier.name);
            const newOpeningBalance = parseFloat(prompt("ادخل الرصيد الافتتاحي الجديد:", supplier.openingBalance));
            if (newName && !isNaN(newOpeningBalance)) {
                supplier.name = newName;
                supplier.openingBalance = newOpeningBalance;
                supplier.currentBalance = supplier.invoices.reduce((sum, inv) => sum + inv.amount, 0) - supplier.payments.reduce((sum, pay) => sum + pay.amountPaid, 0) + newOpeningBalance;
                updateLocalStorage();
                alert("تم تعديل المورد بنجاح");
                searchSupplier();
            } else {
                alert("بيانات غير صحيحة");
            }
        }
    }

    function deleteInvoice(supplierName, invoiceIndex) {
        const supplier = suppliers.find(s => s.name === supplierName);
        if (supplier) {
            if (confirm("هل أنت متأكد من حذف هذه الفاتورة؟")) {
                supplier.currentBalance -= supplier.invoices[invoiceIndex].amount;
                supplier.invoices.splice(invoiceIndex, 1);
                updateLocalStorage();
                alert("تم حذف الفاتورة بنجاح");
                searchSupplier();
            }
        }
    }

    function deletePayment(supplierName, paymentIndex) {
        const supplier = suppliers.find(s => s.name === supplierName);
        if (supplier) {
            if (confirm("هل أنت متأكد من حذف هذا الدفع؟")) {
                supplier.currentBalance += supplier.payments[paymentIndex].amountPaid;
                supplier.payments.splice(paymentIndex, 1);
                updateLocalStorage();
                alert("تم حذف الدفع بنجاح");
                searchSupplier();
            }
        }
    }

    function deleteSupplier(supplierName) {
        if (confirm("هل أنت متأكد من حذف هذا المورد؟")) {
            suppliers = suppliers.filter(s => s.name !== supplierName);
            updateLocalStorage();
            alert("تم حذف المورد بنجاح");
            searchSupplier();
        }
    }

    function showSupplierSummary() {
        let summaryTable = `
            <table>
                <thead>
                    <tr>
                        <th>اسم المورد</th>
                        <th>الرصيد الافتتاحي</th>
                        <th>الرصيد الحالي</th>
                    </tr>
                </thead>
                <tbody>
        `;
        suppliers.forEach(supplier => {
            summaryTable += `
                <tr>
                    <td>${supplier.name}</td>
                    <td>${supplier.openingBalance}</td>
                    <td>${supplier.currentBalance}</td>
                </tr>
            `;
        });
        summaryTable += `</tbody></table>`;
        document.getElementById('supplierSummary').innerHTML = summaryTable;
    }

    function printReport(supplierName) {
        const supplier = suppliers.find(s => s.name === supplierName);
        if (supplier) {
            let reportContent = `
                <html>
                <head>
                    <style>
                        body {
                            font-family: Arial, sans-serif;
                        }
                        table {
                            width: 100%;
                            border-collapse: collapse;
                        }
                        th, td {
                            border: 1px solid #000;
                            padding: 10px;
                            text-align: center;
                        }
                        th {
                            background-color: #f2f2f2;
                        }
                    </style>
                </head>
                <body>
                    <h2>تقرير حساب المورد: ${supplierName}</h2>
                    <h4>التاريخ: ${new Date().toLocaleDateString()} الوقت: ${new Date().toLocaleTimeString()}</h4>
                    <table>
                        <thead>
                            <tr>
                                <th>التاريخ</th>
                                <th>الفاتورة</th>
                                <th>الدفع</th>
                            </tr>
                        </thead>
                        <tbody>
            `;
            supplier.invoices.forEach(inv => {
                reportContent += `
                    <tr>
                        <td>${inv.date}</td>
                        <td>${inv.amount}</td>
                        <td>-</td>
                    </tr>
                `;
            });
            supplier.payments.forEach(pay => {
                reportContent += `
                    <tr>
                        <td>${pay.date}</td>
                        <td>-</td>
                        <td>${pay.amountPaid}</td>
                    </tr>
                `;
            });
            reportContent += `
                <tr>
                    <td colspan="2"><strong>الرصيد الحالي</strong></td>
                    <td><strong>${supplier.currentBalance}</strong></td>
                </tr>
            `;
            reportContent += `
                </tbody>
                </table>
            `;
            const printWindow = window.open('', '_blank');
            printWindow.document.write(reportContent);
            printWindow.document.close();
            printWindow.print();
        }
    }
</script>

</body>
</html>
