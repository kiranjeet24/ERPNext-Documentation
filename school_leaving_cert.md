##school leaving certificate

Go to the Print Format List, click on New.
Enter a name and select a DocType for which the Print Format is to be used.
For school leaving certificate we will choose doctype student and name of the print as school leaving certificate.
The module for which it should apply will be selected automatically.
To customize the certificate click on edit format.
Here we can drag and drop fields or custom html from the sidebar to the page and vice versa. 
Then added the custom html for certificate. 
Select education module select format as standard format or not we use 'NO' option because this format is only used when we want to generate school leaving certificate.
Use custom css for designing logo of company for fetching student name we use doc.first_name.
For fetching class name as it was in another doctype we use frappe function, frappe.db.get_value. 
Then goto student list open student then click on print.
After opening the print on left their will be written standard format.
Click on standard a dropdown will appear then click on the school leaving certificate option from dropdown.
Then the school leaving certificate of a particular student will appear.

