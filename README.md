## Payroll Module

### Salary Component

- In Salary Component Section we can define Salary Component like Basic Pay, Medical Allowance etc also define that component is Earning and Deduction component.
- Go to Salary Component list, click on New.
- Enter its Name and Abbreviation.
- Enter Description of the Salary Component (optional).
- Enter the Company name and the Default Account of the Salary Component in the Accounts table.
- Under Condition and Formula we can add Amount for which allowance amount is fixed.

### Salary Structure

- Salary Structure is the details of the salary being offered to an Employee, in terms of the breakup of the different components constituting the compensation.
- Go to the Salary Structure list, click on New.
- Under the Earning and Deduction Section add Earning Components also we can add formula like if base salary is greater than 1 only then formula is apply on Salary Structure.
- Select the Company Name and Payroll Frequency.

### Salary Structure Assignment

- After creating Salary Structure then Assign this Salary Structure whom Salary structure is same.
- Go to Salary Structure Assignment list and click on New.
- Select the Employee and Salary Structure.
- Select the From Date from which this particular Salary Structure will be applicable.
- Select preferred Income Tax Slab for the employee.
- Enter Base and Variable amount as per requirement. Base amount refers to the Base Salary of the Employee, which is fixed and paid out, regardless of employees meeting their goals. Variable pay, on the other hand, is the portion of sales compensation determined by employee performance.

### Salary Slip 

- A salary slip is a document issued to an employee. It contains a detailed description of the employee’s salary components and amounts.
- Go to Salary Slip, Click on New.
- Select Employee. On selecting Employee all details of the Employee will be fetched from Salary Structure which is assigned to that Employee. This includes details such as Payroll Frequency, Earnings, Deductions, etc.
- Select Start Date and End Date.
- Here We can also add and remove

### Payroll Entry 

- Payroll is the sum total of all compensation a business must pay to its employees for a set period of time or on a given date.
- Here We can create salary of multiple employees in bulk also we can validate their attendance.


### Steps Followed by us for Creating employee Salary are

- Basic Pay of employee as per college pay scale.
- Add 5% Interim Relief in Basic pay.
- Adding Dearness allowance 142% in Basic Pay.
- Medical Allowance,CCA,PF(10%),HRA.
- These all are Earning in salary.
- Then we add Deduction component like PF(20%),Development tax, GI, SML, SMAF.
- By calculating all Earning & Deduction, we get Net Paid Amount. 
- First we create salary component which are fixed these are declared in Salary Component.
- Next Create Salary Structure in which we group all Salary Component which are earning and deduction components.
- Setting up formula on Basic Pay like calculate IR(5%), ADA(142%) etc.
- After Successfully creating salary structure assigning it to employee where we define the Basic pay of Employee.
- At last in Salary Slip we select employee to whom we assign salary then salary structure automatically fetched and calculate base salary. 

### Adding bus Fee in Fee doctype of erpNext

- By default we have Fee component in which all fee categories are fetched but as per our requirement we need to add bus fee which is different for each student.
- For which First we create two doctype one in which Route detail with Fees is stored and second is child doctype called Bus component which is fetched in Fee doctype.
- Then Apply changes in Bus component like fetch value from Route detail doctype.
- For calculating total bus component and fee component we have to write some code in fee.js file.

```js
calculate_total_amount: function(frm) {
	var grand_total0 = 0, grand_total1 = 0, grand_total = 0;

	for(var i=0;i<frm.doc.components.length;i++) {
		if( frm.doc.components[i].amount >= 0){
			grand_total0 += frm.doc.components[i].amount;
		}
		else{
			grand_total0 = 0;
		}
	}

	if(frm.doc.bus_components.length == 0)
	{
		grand_total1 = 0;
	}
	if(frm.doc.bus_components.length >= 1)
	{
	for(var i=0;i<frm.doc.bus_components.length;i++) {
		if( frm.doc.bus_components[i].amount > 0){
			grand_total1 += frm.doc.bus_components[i].amount;
		}
		else{
			grand_total1 = 0;
		}
	}
	}
	grand_total = grand_total0 + grand_total1;
	console.log(grand_total)
	frm.set_value("grand_total", grand_total);
}

frappe.ui.form.on("Bus Component", {
	amount: function(frm) {
		frm.trigger("calculate_total_amount");
	}
});
```

### Make changes for in fee files to save data at backend

- First write code in api.py file create whitelist() where we use condition if bus component available.
- Then use it in Javascript file as function where we call api.py and sending data field data to function.
- Now in fee.py we calculate the bus component in loop and send grand total in database throught which we succesfully generate receipt with correct calculations.

### Noticeboard App

- In First approach we decide to use create doctype and apply workflow on it.
- Where all states are defined like Approve by hod, Draft etc.
- We allote Department to their Respective Hod and Clerk  So that they are able to create notice.
- Every thing Works fine but When we apply some code then workflow states create problem. eg data is saved but changes apply with delay.
- We find workflow also make task for app difficult because its file is not created inside the app that why it portability is difficult.
- So we decide to work without workflow also learn new things from mistakes.
- Assigning department to Hod & Clerk to one department in User Permission List.
- We use condition like cse = department_name.
- Department_name is fetched from doctype use function self.fieldname.
- After apply such condition we use make_autoname function.
- self.name = make_autoname('NOTICE-'+'CSE'+'/'+'.YYYY.'+'/'+'.#####')
- This function return series 'NOTICE-CSE/2022/00001'

**Fetching HOD by using variables in query**
```py
department = self.department
requiredRole = "Hod" 
		
		self.hod = frappe.db.sql(f""" select full_name 
			from `tabUser` 
			where `email` IN (select user 
			from `tabUser Permission` 
			where `for_value`="{department}" AND `user` IN (select parent 
			from `tabHas Role` 
			where `role`="{requiredRole}" )) """)
```

- By using this query with variable we are able to use this for all departments.
