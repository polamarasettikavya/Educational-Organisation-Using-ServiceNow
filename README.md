# Educational-Organisation-Using-ServiceNow
### **Problem Statement**

Managing educational operations such as admissions, student progress tracking, and administrative workflows can be cumbersome and prone to errors when handled manually. To address this, a **ServiceNow-based Educational Management System** was developed.
This system enables educational institutions to manage student records, admission details, and performance progress efficiently within a unified platform.

The solution provides:

* Streamlined data management for students and admissions.
* Automated client-side scripts to calculate totals, percentages, and results.
* Dynamic forms for user-friendly data entry and process flow visualization.
* Centralized tracking through **update sets** for deployment across instances.

---

## **Implementation Steps**

### **1. Setting Up ServiceNow Instance**

**Navigation:**
Go to [ServiceNow Developer Site](https://developer.servicenow.com)

**Steps:**

1. Sign up for a **Developer Account**.
2. Navigate to **Personal Developer Instance** → Click **Request Instance**.
3. Fill in the required details and submit the request.
4. Once approved, you’ll receive credentials via email.
5. Log in to your instance and start configuration.

---

### **2. Creating a Local Update Set**

**Navigation:**
ServiceNow → All → *Local Update Sets*

**Steps:**

1. Click **New**.
2. Enter:

   * **Name:** Educational Organisation
3. Click **Submit** → **Make Current**.

🟢 *Note:* All subsequent configurations will be captured under this update set.

---

### **3. Creating Tables**

#### **a. Salesforce Table**

**Navigation:**
ServiceNow → *System Definition* → *Tables* → *New*

**Steps:**

1. **Label:** Salesforce
   (API Name auto-generates as `u_salesforce`)
2. Create columns for:

   * Admin Number (Display = True)
   * Student Name
   * Grade
   * Father Name
   * Mother Name
   * Contact Numbers, etc.
3. Enable **Extensible** under Controls.
4. For *Admin Number* → Advanced View → Set **Dynamic Default Value** to *Get Next Padded Number*.
5. For *Grade* → Add Choices (e.g., Grade 1, Grade 2, Grade 3, etc.).

---

#### **b. Admission Table**

**Navigation:**
ServiceNow → *System Definition* → *Tables* → *New*

**Steps:**

1. **Label:** Admission
2. **Extends Table:** Salesforce
3. **Add Module to Menu:** Checked
4. Add fields such as:

   * Admission Number
   * Purpose of Join
   * Admin Status
   * School Name
   * Pincode
   * Fee, etc.
5. Add Choices for:

   * **Admin Status:** New, In Progress, Joined, Rejected, Rejoined, Closed, Cancelled
   * **Pincode**, **Purpose of Join**, **School**, and **School Area**

---

#### **c. Student Progress Table**

**Navigation:**
ServiceNow → *System Definition* → *Tables* → *New*

**Steps:**

1. **Label:** Student Progress
2. **Add Module to Menu:** Salesforce
3. Add fields such as:

   * Admission Number
   * Telugu, Hindi, English, Maths, Science, Social
   * Total, Percentage, Result

---

### **4. Form Layout Configuration**

**Navigation:**
Student Progress Table → *Form Layout*

**Steps:**

1. Move the required fields (like Admission Number and its related details) from *Available* to *Selected* side.
2. Click **Save**.

---

### **5. Form Design**

#### **a. Salesforce Table**

**Navigation:**
System Definition → Tables → Open *Salesforce*

**Steps:**

1. Right-click the header → *Configure → Form Design*
2. Arrange and organize fields logically.
3. Click **Save**.

#### **b. Admission Table**

Repeat the same steps as above and customize fields relevant to the admission process.

#### **c. Student Progress Table**

Repeat and arrange student marks and computed fields (Total, Percentage, Result).

---

### **6. Number Maintenance**

**Navigation:**
All → *Number Maintenance* → *New*

**Steps:**

1. Create number maintenance for **Admin Number**.
2. Fill in details and click **Submit**.

---

### **7. Process Flow**

**Navigation:**
All → *Process Flow* → *New*

**Steps:**

1. Create process flow for **Admission Table**.
2. Define states and order:

   * **New → In Progress → Joined → Rejected → Rejoined → Closed → Cancelled**
3. Click **Insert and Stay** for each state.

---

### **8. Client Scripts**

#### **a. Auto Populate Script (Admission Table)**

Automatically fills student details based on Admission Number.

```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;

    var a = g_form.getReference('u_admission_number');
    g_form.setValue('u_admin_date', a.u_admin_date);
    g_form.setValue('u_grade', a.u_grade);
    g_form.setValue('u_student_name', a.u_student_name);
    g_form.setValue('u_father_name', a.u_father_name);
    g_form.setValue('u_mother_name', a.u_mother_name);
    g_form.setValue('u_father_cell', a.u_father_cell);
    g_form.setValue('u_mother_cell', a.u_mother_cell);

    g_form.setDisabled('u_admin_date', true);
    g_form.setDisabled('u_grade', true);
    g_form.setDisabled('u_student_name', true);
    g_form.setDisabled('u_father_name', true);
    g_form.setDisabled('u_mother_name', true);
    g_form.setDisabled('u_father_cell', true);
    g_form.setDisabled('u_mother_cell', true);
}
```

---

#### **b. Pincode Update Script (Admission Table)**

Auto-fills Mandal, City, and District based on Pincode.

```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;

    var a = g_form.getValue('u_pincode');

    if (a == '509358') {
        g_form.setValue('u_mandal', 'Kadthal');
        g_form.setValue('u_city', 'Kadthal');
        g_form.setValue('u_district', 'RangaReddy');
    } else if (a == '500081') {
        g_form.setValue('u_mandal', 'Karmanghat');
        g_form.setValue('u_city', 'Karmanghat');
        g_form.setValue('u_district', 'RangaReddy');
    } else if (a == '500079') {
        g_form.setValue('u_mandal', 'Abids');
        g_form.setValue('u_city', 'AsifNagar');
        g_form.setValue('u_district', 'Hyderabad');
    }
}
```

---

#### **c. Disable Fields Script (Student Progress Table)**

Prevents editing of calculated fields.

```javascript
function onLoad() {
    g_form.setDisabled('u_total', true);
    g_form.setDisabled('u_percentage', true);
    g_form.setDisabled('u_result', true);
}
```

---

#### **d. Total Calculation Script (Student Progress Table)**

```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;

    var a = parseInt(g_form.getValue('u_telugu'));
    var b = parseInt(g_form.getValue('u_hindi'));
    var c = parseInt(g_form.getValue('u_english'));
    var d = parseInt(g_form.getValue('u_maths'));
    var e = parseInt(g_form.getValue('u_science'));
    var f = parseInt(g_form.getValue('u_social'));
    var total = a + b + c + d + e + f;

    g_form.setValue('u_total', total);
}
```

---

#### **e. Percentage Script (Student Progress Table)**

```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;

    var total = g_form.getValue('u_total');
    var percentage = (total / 600) * 100;
    g_form.setValue('u_percentage', percentage + '%');
}
```

---

#### **f. Result Script (Student Progress Table)**

```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') return;

    var a = parseInt(g_form.getValue('u_percentage'));

    if (a >= 0 && a <= 59) {
        g_form.setValue('u_result', 'Fail');
    } else if (a >= 60 && a <= 100) {
        g_form.setValue('u_result', 'Pass');
    } else {
        g_form.addErrorMessage('Percentage should be between 0 and 100.');
        g_form.clearValue('u_result');
    }
}
```

---

### **9. Result**

**Final Forms Created:**

1. **Salesforce Table Form:** Contains student and parent details.
2. **Admission Table Form:** Displays admission workflow and pincode-based auto-population.
3. **Student Progress Table Form:** Calculates total, percentage, and displays result automatically.

*(Refer to screenshots in your attached images for visual output.)*

---

## **Project Structure**

```
Educational-Organisation-Using-ServiceNow/
│
├── Update Set/
│  └── Educational Organisation (Local Update Set)
│
├── Tables/
│  ├── Salesforce Table
│  ├── Admission Table (Extends Salesforce)
│  └── Student Progress Table
│
├── Form Configuration/
│  ├── Form Layout – Student Progress Table
│  ├── Form Design – Salesforce Table
│  ├── Form Design – Admission Table
│  └── Form Design – Student Progress Table
│
├── Number Maintenance/
│  └── Admin Number Auto-Generation
│
├── Process Flow/
│  └── Admission Table Workflow
│    → New → In Progress → Joined → Rejected → Rejoined → Closed → Cancelled
│
├── Client Scripts/
│  ├── Auto Populate Script – Admission Table
│  ├── Pincode Update Script – Admission Table
│  ├── Disable Fields Script – Student Progress Table
│  ├── Total Calculation Script – Student Progress Table
│  ├── Percentage Calculation Script – Student Progress Table
│  └── Result Script – Student Progress Table
│
└── Result/
  └── Configured forms displaying automation of Admission & Student Progress management
```

---

## **Outcome**

The **Educational Organisation Using ServiceNow** project successfully automates key processes for managing academic operations within an educational institution.

### **Key Achievements:**

* ✅ **Automated Data Handling:** Student and admission data auto-populate for consistency.
* ✅ **Dynamic Calculations:** Marks, totals, percentages, and results are computed automatically.
* ✅ **Process Visualization:** Admission process flow provides clear state tracking.
* ✅ **Reduced Manual Work:** Automations minimize human errors and data redundancy.
* ✅ **Centralized Platform:** Unified management of student information, admissions, and progress.

---

## **Overall Result:**

This project demonstrates the capability of **ServiceNow** as an educational management tool — providing automation, efficiency, and visibility across institutional processes, thus modernizing administrative workflows for educational organizations.

---
