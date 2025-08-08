<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Payroll Management System</title>
  <style>
    body { font-family: Arial, sans-serif; margin:0; padding:0; background:#f8fafd;}
    .container { max-width:500px; margin:40px auto; background:#fff; padding:40px 30px 30px 30px; border-radius:12px; box-shadow:0 2px 10px #ccc;}
    .heading { font-size:2rem; text-align:center; margin-bottom:30px; color:#283048;}
    h2 { text-align:center; }
    .back { position:absolute; left:20px; top:20px; padding:6px 18px; background:#efefef; border:none; border-radius:18px; cursor:pointer; }
    .nav-btn { margin-top:20px; display:block; width:100%; padding:12px; background:#2e86c1; color:white; border:none; border-radius:6px; font-size:1rem; cursor:pointer;}
    .company-selector { margin:25px 0;}
    .dot-menu { display:inline-block; width:30px; cursor:pointer;}
    .dot{height:7px;width:7px;background:#283048;border-radius:50%;display:inline-block;margin:0 2px;animation: moveDots 1.5s infinite alternate;}
    @keyframes moveDots{ 0%{transform:translateY(0);} 100%{transform:translateY(-6px);} }
    label{display:block;margin:14px 0 5px;}
    input[type="text"], input[type="number"], input[type="date"] { width:100%; padding:7px; margin-bottom:10px; }
    .employee-list { margin:16px 0;}
    .employee { display:flex; align-items:center; justify-content:space-between; margin-bottom:7px; }
    .add-btn { margin-top:10px; background:#e67e22; color:white; padding:6px 14px; border:none; border-radius:12px; cursor:pointer;}
    .edit-btn { background:#8cc152; color:white; border:none; border-radius:6px; padding:3px 8px; margin-left:7px; cursor:pointer;}
    .del-btn { background:#df4545; color:white; border:none; border-radius:6px; padding:3px 8px; cursor:pointer;}
    .butterfly { position:fixed; width:40px; left:calc(35vw + 30px * var(--i)); top:calc(10vh + 50px * var(--i)); animation:fly 3s infinite alternate;}
    @keyframes fly { 0%{transform:rotate(-8deg) translateY(0);} 100%{transform:rotate(16deg) translateY(-25px);}
    }
    .hide { display:none;}
    .output-files{margin-top:20px;}
  </style>
</head>
<body>
  <!-- Butterflies -->
  <div class="butterfly" style="--i:1;z-index:99;font-size:2rem;">🦋</div>
  <div class="butterfly" style="--i:2;z-index:99;font-size:2rem;">🦋</div>

  <div class="container" id="page1">
    <div class="heading">Payroll Management Portal</div>
    <h2>Select Company</h2>
    <div class="company-selector">
      <select id="companyList">
        <option value="NRSR">NRSR & Co</option>
        <option value="Google">Google</option>
      </select>
      <span class="dot-menu" onclick="showAddCompany()">
        <span class="dot"></span><span class="dot"></span><span class="dot"></span>
      </span>
      <div id="addCompany" class="hide">
        <input type="text" id="newCompany" placeholder="Enter new company name"/>
        <button onclick="addCompany()">Add</button>
      </div>
    </div>
    <button class="nav-btn" onclick="gotoPage(2)">Next</button>
  </div>

  <div class="container hide" id="page2">
    <button class="back" onclick="gotoPage(1)">Back</button>
    <h2>Manage Employees of <span id="activeCompany"></span></h2>
    <div class="employee-list" id="employeeList"></div>
    <input type="text" id="employeeName" placeholder="Add new employee name"/>
    <button class="add-btn" onclick="addEmployee()">Add Employee</button>
    <button class="nav-btn" onclick="gotoPage(3)">Next</button>
  </div>

  <div class="container hide" id="page3">
    <button class="back" onclick="gotoPage(2)">Back</button>
    <h2>Employee Details</h2>
    <form id="empDetailsForm"></form>
    <button class="nav-btn" onclick="saveDetails()">Save & Next</button>
  </div>

  <div class="container hide" id="page4">
    <button class="back" onclick="gotoPage(3)">Back</button>
    <h2>Payroll Calculation</h2>
    <div id="payrollCalcs"></div>
    <button class="nav-btn" onclick="gotoPage(5)">Next</button>
  </div>

  <div class="container hide" id="page5">
    <button class="back" onclick="gotoPage(4)">Back</button>
    <h2>Download Payroll</h2>
    <div class="output-files">
      <button onclick="downloadFile('excel')">Download Excel</button>
      <button onclick="downloadFile('pdf')">Download PDF</button>
    </div>
  </div>

<script>
  // Sample data
  let companyData = {
    "NRSR": ["Niranjan", "Ujwal", "Abhishek", "Krishna"],
    "Google": ["A", "B", "C"]
  };
  let empDetails = {};
  let payrollData = {};
  let currentCompany = "NRSR";

  function showAddCompany() {
    document.getElementById('addCompany').classList.toggle('hide');
  }
  function addCompany() {
    let name = document.getElementById('newCompany').value.trim();
    if(!name) return alert('Enter company name');
    if(companyData[name]) return alert('Company exists');
    companyData[name] = [];
    let sel = document.getElementById('companyList');
    let opt = document.createElement('option');
    opt.value = name; opt.textContent = name;
    sel.append(opt);
    sel.value = name;
    showAddCompany();
    document.getElementById('newCompany').value = '';
  }

  function gotoPage(n) {
    for(let i=1;i<=5;i++) document.getElementById('page'+i).classList.add('hide');
    document.getElementById('page'+n).classList.remove('hide');
    if(n == 2) setupEmployeePage();
    if(n == 3) setupDetailsPage();
    if(n == 4) setupPayrollPage();
    if(n == 5) setupDownloadPage();
  }

  // Page 2: Employee management
  function setupEmployeePage() {
    let company = document.getElementById('companyList').value;
    currentCompany = company;
    document.getElementById('activeCompany').textContent = company;
    let listDiv = document.getElementById('employeeList');
    listDiv.innerHTML = '';
    (companyData[company] || []).forEach((e,i) => {
      let div = document.createElement('div');
      div.className = 'employee';
      div.innerHTML = `${e} 
      <button class='edit-btn' onclick='editEmp(${i})'>Edit</button>
      <button class='del-btn' onclick='deleteEmp(${i})'>Del</button>`;
      listDiv.appendChild(div);
    });
  }
  function addEmployee() {
    let val = document.getElementById('employeeName').value.trim();
    if(val) {
      companyData[currentCompany].push(val);
      setupEmployeePage();
      document.getElementById('employeeName').value = '';
    }
  }
  function editEmp(idx) {
    let newName = prompt('Edit employee name:', companyData[currentCompany][idx]);
    if(newName) { companyData[currentCompany][idx] = newName.trim(); setupEmployeePage(); }
  }
  function deleteEmp(idx) {
    if(confirm('Delete employee?')) {
      companyData[currentCompany].splice(idx,1);
      setupEmployeePage();
    }
  }

  // Page 3: Employee details
  function setupDetailsPage() {
    let form = document.getElementById('empDetailsForm');
    form.innerHTML = '';
    (companyData[currentCompany] || []).forEach((e,i) => {
      let details = empDetails[e] || {};
      form.innerHTML += `
        <label>${e}</label>
        <input type="date" placeholder="Date of joining" value="${details.doj||''}" id="doj_${i}">
        <input type="text" placeholder="PAN" value="${details.pan||''}" id="pan_${i}">
        <input type="number" placeholder="CTC" value="${details.ctc||''}" id="ctc_${i}">
        <input type="date" placeholder="Leaving date" value="${details.leave||''}" id="leave_${i}">
        <hr>
      `;
    });
  }
  function saveDetails() {
    (companyData[currentCompany] || []).forEach((e,i) => {
      empDetails[e] = {
        doj: document.getElementById('doj_'+i).value,
        pan: document.getElementById('pan_'+i).value,
        ctc: document.getElementById('ctc_'+i).value,
        leave: document.getElementById('leave_'+i).value,
      };
    });
    gotoPage(4);
  }

  // Page 4: Payroll calculation
  function setupPayrollPage() {
    let div = document.getElementById('payrollCalcs');
    div.innerHTML = '';
    (companyData[currentCompany] || []).forEach((e,i) => {
      let d = empDetails[e] || {};
      payrollData[e] = payrollData[e] || {};
      div.innerHTML += `
        <h3>${e}</h3>
        <label>ESI (%): </label><input type="number" value="${payrollData[e].esi||''}" id="esi_${i}" min="0" max="100"><br>
        <label>PF (%): </label><input type="number" value="${payrollData[e].pf||''}" id="pf_${i}" min="0" max="100"><br>
        <label>TDS (%): </label><input type="number" value="${payrollData[e].tds||''}" id="tds_${i}" min="0" max="100"><br>
        <label>HRA: </label><input type="number" value="${payrollData[e].hra||''}" id="hra_${i}"><br>
        <label>DA: </label><input type="number" value="${payrollData[e].da||''}" id="da_${i}"><br>
        <label>Other Allowance: </label><input type="number" value="${payrollData[e].other||''}" id="other_${i}"><br>
        <hr>
      `;
    });
  }
  // Save payroll data
  function setupDownloadPage() {
    (companyData[currentCompany] || []).forEach((e,i) => {
      payrollData[e] = {
        esi: document.getElementById('esi_'+i).value,
        pf: document.getElementById('pf_'+i).value,
        tds: document.getElementById('tds_'+i).value,
        hra: document.getElementById('hra_'+i).value,
        da: document.getElementById('da_'+i).value,
        other: document.getElementById('other_'+i).value,
      }
    });
  }

  // Dummy download functionality – real implementation requires backend/export libraries
  function downloadFile(type) {
    alert("Payroll data saved as "+type.toUpperCase()+" (Mockup)");
  }

  // Init page
  document.getElementById('companyList').addEventListener('change', setupEmployeePage);
</script>
</body>
</html>
# Payroll-website
