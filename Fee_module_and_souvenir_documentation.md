## Fee Module 

### Fee 

In the Fee section we can create a fee for an Individual student one by one
We have to enter the Student ,due date and we can aslo edit post time and date 
to create a fee we need some of its components which are described below:

- Prerequisites
  - Student 
  - Fee Structure
  - Fee Category


### Fee Structure

In the fee Structure we can define the type of fee like for which class and category of students what kind of fee should be there

In the fee structure we to provide program , student categor, acedamic year, acedamic term .
The important part we have to define the fee category components which includes the different kind of fee. 

### Fee Category

In the Fee Category we simply add the Different Fee components like Tution fee ,Bus fee,Books etc.

Fee Schedule

- Prerequisites
  - Fee Structure
  - Student group


**Fee schedule** is used to add bulk data for fee according to the Student groups and student category.
It will divide the grand_total with each group of students.
Just go to fee schedule and click new fee schedule.
After creating schedule just click Create fee button on top that will create fee for the students.

## NSPS-Requirements Siblings Discount
New requirement was that we have to add the fee discounts that was not currently available in erpnext so we did some code and doctypes in the fee module.
in the api file which is located in apps/ernext/erpnext/doctypes/education/api.py
we create some apis data.
which is given below :
```py
@frappe.whitelist()
def get_sibling_details(student):
    """Returns Student Sibling.
    :param student: Student.
    """
    if student:
        fs = frappe.get_all(
                "Student Sibling",
                fields=["full_name", "gender","program","date_of_birth"],
                filters={"parent": student},
                order_by="idx",
        )
        return fs

@frappe.whitelist()
def get_student_details(student):
    if student:
        dj = frappe.get_all("Student", fields=["full_name", "gender", "date_of_birth"], )
        return dj

@frappe.whitelist()
def get_details(user = None):
    p = frappe.db.get_value("Student",user,["date_of_birth","gender","student_email_id"])
    return p	

@frappe.whitelist()
def get_p_details(user = None):
    r = frappe.db.get_value("Program Enrollment",user,["program","academic_year","academic_term"])
    return r	

@frappe.whitelist()
def get_bus_component(bus_component):
    """Returns Fee Components.
    :param fee_structure: Fee Structure.
    """
    if bus_component:
        fs = frappe.get_all(
                "Bus Component",
                fields=["bus_route", "description","amount"],
                filters={"parent": bus_component},
                order_by="idx",
        )
        return fs
@frappe.whitelist()
def get_bus_detail(user):
    r = frappe.db.get_value("Program Enrollment",user,["mode_of_transportation","vehicle_no",])
    return r

# @frappe.whitelist()
# def get_bus_details(user = ""):
#     bus = frappe.db.sql(f""" SELECT  bus_route,description,amount FROM `tabBus Component` where parenttype = '{user}' """,as_dict = True)
#     return bus

@frappe.whitelist()
def get_bus_route(user):
    r = frappe.db.get_value("Bus Route",user,["category_name","description","amount"])
    return r

@frappe.whitelist()
def get_programEnrollment_details(user):
    bus = frappe.db.sql(f""" SELECT name,program,academic_year,academic_term FROM `tabProgram Enrollment` where student = '{user}' """,as_dict = True)
    return bus

@frappe.whitelist()
def get_fee_list():
    d = frappe.get_list('Fees', filters={'docstatus': "Draft",}, fields=['student','student_name', 'due_date', 'outstanding_amount'], order_by='outstanding_amount')   
    return d

@frappe.whitelist()
def get_over_due():
    due = frappe.db.sql(f""" SELECT name,student,student_name,academic_year,due_date,outstanding_amount  FROM `tabFees` where due_date <= '{today()}' and outstanding_amount > 0 """,as_dict = True)
    return due

```
### Changes some doctypes like in the fee components 
- added one sibling discount
- two siblings discount 
- amount after discount
Which are used in coding to trigger the fucntions and also we can then edit it through the fornt end in the fee component table to add the required fields,
##### NOTE : The width of the table should not be more than 10 so set the value accordingly 

### Also added the field in the fees doctype 
Simple set the field name to managemnt discount and type % age and its coding will be done through backend
 apps/ernext/erpnext/doctypes/education/doctypes/fee.js
 ```js
   fee_structure: function (frm) {
    frm.set_value("components", "");
    if (frm.doc.fee_structure) {
      frappe.call({
        method: "erpnext.education.api.get_fee_components",
        args: {
          fee_structure: frm.doc.fee_structure,
        },
        callback: function (r) {
          if (r.message) {
            $.each(r.message, function (i, d) {
              var row = frappe.model.add_child(
                frm.doc,
                "Fee Component",
                "components"
              );
              row.fees_category = d.fees_category;
              row.description = d.description;
              row.amount = d.amount;
              row.one_sibling_discount = d.one_sibling_discount;
              row.two_sibling_discount = d.two_sibling_discount;
              row.amount_after_discount = d.amount_after_discount;
            });
          }
          refresh_field("components");
          frm.trigger("calculate_total_amount");
        },
      });
    }
  },

  // Siblings
  student: function (frm) {
    frm.set_value("siblings", "");
    frm.set_value("bus_component", "");
    if (frm.doc.student) {
      frappe.call({
        method: "erpnext.education.api.get_sibling_details",
        args: {
          student: frm.doc.student,
        },
        callback: function (r) {
          if (r.message) {
            $.each(r.message, function (i, d) {
              var row = frappe.model.add_child(
                frm.doc,
                "Student Sibling",
                "siblings"
              );
              row.full_name = d.full_name;
              row.gender = d.gender;
              row.program = d.program;
              row.date_of_birth = d.date_of_birth;
            });
          }
          cur_frm.refresh_field("siblings");
          frm.trigger("calculate_no_of_siblings");
        },
      });

      //   frappe.call({
      //     method: "erpnext.education.api.get_current_enrollment",
      //     args: {
      //       student: frm.doc.student,
      //       academic_year: frm.doc.academic_year,
      //     },
      //     callback: function (r) {
      //       if (r) {
      //         console.log(r);
      //         $.each(r.message, function (i, d) {
      //           frm.set_value(i, d);
      //         });
      //       }
      //     },
      //   });

      frappe.call({
        method: "erpnext.education.api.get_programEnrollment_details",
        args: { user: frm.doc.student },
        callback: function (r) {
          if (r) {
            $.each(r.message, function (i, d) {
              frm.set_value("program_enrollment", d.name);
              frm.set_value("program", d.program);
              frm.set_value("academic_year", d.academic_year);
              frm.set_value("academic_term", d.academic_term);
            });
          }
        },
      });
    }

    // FETCHING STUDENT DETAILS
  },

  program_enrollment: function (frm) {
    if (frm.doc.program_enrollment) {
      frappe.call({
        method: "erpnext.education.api.get_bus_detail",
        args: { user: frm.doc.program_enrollment },
        callback: function (r) {
          if (r.message) {
            var route = r.message[1];
            console.log(route);
            if (r.message[0] == "Institute's Bus") {
              frappe.msgprint(
                `The mode of Transportation is ${r.message[0]} and the bus is ${r.message[1]}  `
              );
            } else {
              frappe.msgprint(`The mode of Transportation is ${r.message[0]} `);
            }
            frappe
              .call({
                method: "erpnext.education.api.get_bus_route",
                args: { user: route },
              })
              .done((r) => {
                let entry = frm.add_child("bus_component");
                console.log(r.message);
                entry.bus_route = r.message[0];
                entry.description = r.message[1];
                entry.amount = r.message[2];
                refresh_field("bus_component");
              });
          }
        },
      });
    }
  },

  // transportation_detail: function (frm) {

  // },

  // Sibings end
  calculate_no_of_siblings: function (frm) {
    frm.set_value("no_of_siblings", 0);
    if (frm.doc.siblings) {
      if (frm.doc.siblings.length > 0) {
        frm.set_value("no_of_siblings", frm.doc.siblings.length);
      }
    }
    cur_frm.refresh_field("no_of_siblings");

    frappe.call({
      method: "erpnext.education.api.get_details",
      args: { user: frm.doc.student },
      callback: (data) => {
        var s_date_of_birth = data.message[0];
        frm.set_value("student_email", data.message[2]);

        if (frm.doc.no_of_siblings == 1) {
          for (var i = 0; i < frm.doc.siblings.length; i++) {
            if (s_date_of_birth > frm.doc.siblings[i].date_of_birth) {
              frappe.msgprint("He is eligible for Discount 50% ");
            }
          }
        } else if (frm.doc.no_of_siblings == 2) {
          for (var i = 0; i < frm.doc.siblings.length; i++) {
            var sibling1 = frm.doc.siblings[0].date_of_birth;
            var sibling2 = frm.doc.siblings[1].date_of_birth;
          }

          if (s_date_of_birth > sibling1 && s_date_of_birth > sibling2) {
            frappe.msgprint("Eligible for 50% ");
          } else if (
            (s_date_of_birth > sibling1 && s_date_of_birth < sibling2) ||
            (s_date_of_birth < sibling1 && s_date_of_birth > sibling2)
          ) {
            frappe.msgprint("He is eligible for 25% discount");
          } else if (s_date_of_birth < sibling1 && s_date_of_birth < sibling2) {
            frappe.msgprint("He is Not Eligible for Sibling discount");
          }
        } else if (frm.doc.no_of_siblings > 2 || !frm.doc.siblings) {
          frappe.msgprint(" Not Eligible for Sibling discount");
        }
      },
    });
  },

  // Discount calculations.

  one_discount: function (frm) {
    var after_discount = 0;

    for (var i = 0; i < frm.doc.components.length; i++) {
      after_discount =
        frm.doc.components[i].amount -
        (frm.doc.components[i].amount *
          frm.doc.components[i].one_sibling_discount) /
          100;
      console.log(after_discount);
      frm.doc.components[i].amount_after_discount = after_discount;

      refresh_field("components");
    }
  },

  two_discount: function (frm) {
    var after_discount = 0;
    for (var i = 0; i < frm.doc.components.length; i++) {
      after_discount =
        frm.doc.components[i].amount -
        (frm.doc.components[i].amount *
          frm.doc.components[i].two_sibling_discount) /
          100;
      console.log(after_discount);
      frm.doc.components[i].amount_after_discount = after_discount;

      refresh_field("components");
    }
  },

  calculate_total_amount: function (frm) {
    var grand_total = 0,
      grand_total0 = 0,
      grand_total1 = 0;
    var mang_discount = 0;
    var bus_grand = 0;

    for (var i = 0; i < frm.doc.components.length; i++) {
      grand_total0 += frm.doc.components[i].amount_after_discount;
    }

    console.log(grand_total0);

    if (frm.doc.bus_component) {
      for (var i = 0; i < frm.doc.bus_component.length; i++) {
        grand_total1 += frm.doc.bus_component[i].amount;
      }

      console.log(grand_total1);
    }

    if (grand_total0 && grand_total1) {
      grand_total = grand_total0 + grand_total1;
    } else {
      grand_total = grand_total0;
    }
    if (frm.doc.management_discount) {
      mang_discount =
        frm.doc.grand_total -
        (frm.doc.grand_total * frm.doc.management_discount) / 100;

      grand_total = mang_discount;
    }

    frm.set_value("grand_total", grand_total);
  },
  // Managemetal Discount

  management_discount: function (frm) {
    frm.trigger("calculate_total_amount");
    frm.refresh_field("grand_total");
  },
  // MD END

  // changes
  bus_component: function (frm) {
    frm.set_value("bus_component", "");
    if (frm.doc.bus_components) {
      frappe.call({
        method: "erpnext.education.api.get_bus_component",
        args: {
          bus_components: frm.doc.bus_components,
        },
        callback: function (r) {
          if (r.message) {
            $.each(r.message, function (i, d) {
              var row = frappe.model.add_child(
                frm.doc,
                "Bus Component",
                "bus_component"
              );
              row.bus_route = d.bus_route;
              row.description = d.description;
              row.amount = d.amount;
            });
          }
          refresh_field("bus_component");
          frm.trigger("calculate_total_amount");
        },
      });
    }
  },
});

frappe.ui.form.on("Fee Component", {
  // amount_after_discount: function (frm, cdt, cdn) {
  // },
  amount: function (frm) {
    frm.trigger("calculate_total_amount");
  },
  one_sibling_discount: function (frm) {
    frm.trigger("one_discount");
    frm.trigger("calculate_total_amount");
  },
  two_sibling_discount: function (frm) {
    frm.trigger("two_discount");
    frm.trigger("calculate_total_amount");
  },
});

frappe.ui.form.on("Bus Component", {
  // amount_after_discount: function (frm, cdt, cdn) {
  // },
  bus_route: function (frm) {
    frm.trigger("calculate_total_amount");
  },
  amount: function (frm) {
    frm.trigger("calculate_total_amount");
  },
});
 
 ```
Also for the backend changes we have to change the pyhton code also
apps/ernext/erpnext/doctypes/education/doctypes/fee.py
the changes I made are as follows:

```py
 def calculate_total(self):
        """Calculates total amount."""
        self.grand_total = 0
        for d in self.components:
            self.grand_total += d.amount_after_discount
        for d in self.bus_component:
            self.grand_total += d.amount    

        if(self.management_discount):
            mang_disc = self.grand_total - (self.grand_total * self.management_discount)/100 
            self.grand_total = mang_disc
        elif(self.management_discount == 0):
            for d in self.components:
                self.grand_total += d.amount_after_discount
```

## To Disable Student whose Fee is Unpaid 

- First we have to fetch the details of the students whose fee is overdue we can do this with the following code
- Paste this code in the api.py file in the education directory of erpnext

```py
@frappe.whitelist()
def get_over_due():
    due = frappe.db.sql(f""" SELECT name,student,student_name,academic_year,due_date,outstanding_amount  FROM `tabFees` where due_date <= '{today()}' and outstanding_amount > 0 """,as_dict = True)
    return due

```

- Then we need to use this info to disable the students  and update their enable disable state .
- In the Studnet doctype there is file student_list.js ,paste the below code to operate.


```js
  get_indicator(doc, frm) {
    // customize indicator color
    frappe.call({
      method: "erpnext.education.api.get_over_due",
      callback: function (r) {
        if (r) {
          $.each(r.message, function (i, d) {
            if ((i, d.student === doc.name)) {
              frappe.call({
                method: "frappe.client.set_value",
                args: {
                  doctype: "Student",
                  name: doc.name,
                  fieldname: "enabled",
                  value: 0,
                },
                freeze: true,
                callback: function (r) {
                  frappe.msgprint(__("Student has been Disable from Access"));
                },
              });
            }
          });
        }
      },
    });

    // if (doc.enabled) {
    //   return [__("Enabled"), "blue", "enabled,=,1"];
    //   // } else if (doc.unpaid) {
    //   //   return [__("unpaid"), "red", "unpaid,=,1"];
    // } else {
    //   return [__("Disabled"), "darkgray", "disabled,=,1"];
    // }
  },
```
## Displaying Fees different head-wise like tution fee, Development fee

 Goto Fee list under Education domain.
- Under list view option select report view, Select Add group option here select Fee Category under Fee Component
- Then Add sum Filter and in third field Select Amount or grand total or outstanding.
- Now you are able to see all fee collected head wise like Tution Fee, Development Fee, Bus Fee.

##school leaving certificate

- Go to the Print Format List, click on New.
- Enter a name and select a DocType for which the Print Format is to be used.
- For school leaving certificate we will choose doctype student and name of the print as school leaving certificate.
- The module for which it should apply will be selected automatically.
- To customize the certificate click on edit format.
- Here we can drag and drop fields or custom html from the sidebar to the page and vice versa. 
- Then added the custom 


## Souvenir App

### Validation Of Form
For the form validation  we have to made changes to the following file 

```js
frappe.ready(function () {
  let data = frappe.web_form.get_values();

  frappe.web_form.after_load = () => {
    frappe.msgprint(`Please Fill the Form carefully `);

    frappe.web_form.on("image", (field, value) => {
      console.log(field, value);

      var img = new Image();
      img.src = value;
      img.onload = function () {
        var height = this.height;
        var width = this.width;
        console.log(height, width);
        frappe.web_form.validate = () => {
          if (height > 1200 || width > 930 || height < 774 || width < 600) {
            frappe.throw(
              "Height must be between 774px to 1200px and Width must be Between 600px and 930px"
            );
            return false;
          }
          return true;
        };
      };
    });

  
    frappe.web_form.validate = () => {
      let data = frappe.web_form.get_values();

      var re = /^[A-Za-z]+$/;

      if (
        data.university_roll_no.toString().length > 7 ||
        data.university_roll_no.toString().length < 7
      ) {
        frappe.throw(
          `Please Enter a valid ${data.name1}'s  University Roll NO.`
        );
        return false;
      }

      if (data.contact_no) {
        if (
          data.contact_no.toString().length > 10 ||
          data.contact_no.toString().length < 10 ||
          data.contact_no == " "
        ) {
          frappe.throw("Please Enter a Valid phone number");
          return false;
        }
      }

      if (
        data.college_roll_no.toString().length > 7 ||
        data.college_roll_no.toString().length < 7
      ) {
        frappe.throw(`Please Enter a valid ${data.name1}'s College Roll NO.`);
        return false;
      }

      if (data.friend_1s_crn) {
        if (
          data.friend_1s_crn.toString().length > 7 ||
          data.friend_1s_crn.toString().length < 7
        ) {
          frappe.throw(
            `Please Enter a valid ${data.friend_1s_name}'s College_Roll NO.`
          );
          return false;
        }
      }

      if (data.friend_1s_urn) {
        if (
          data.friend_1s_urn.toString().length > 7 ||
          data.friend_1s_urn.toString().length < 7
        ) {
          frappe.throw(
            `Please Enter a Valid ${data.friend_1s_name}'s University_Roll No.`
          );
          return false;
        }
      }

      if (data.friend_2s_urn) {
        if (
          data.friend_2s_urn.toString().length > 7 ||
          data.friend_2s_urn.toString().length < 7
        ) {
          frappe.throw(
            `Please Enter a valid ${data.friend_2s__name}'s University_Roll NO.`
          );
          return false;
        }
      }

      if (data.friend_2s_crn) {
        if (
          data.friend_2s_crn.toString().length > 7 ||
          data.friend_2s_crn.toString().length < 7
        ) {
          frappe.throw(
            `Please Enter a valid ${data.friend_2s__name}'s College_Roll NO.`
          );
          return false;
        }
      }

      return true;
    };
  };
});
```
### Fetching data form one doctype to another

For fetching data we use both python and js calls
Python Code 
in the file /souvenir_form/souvenir_form/web_form/souvenir/souvenir.py 
```py
@frappe.whitelist()
def get_detail(user = None):
	p = frappe.db.get_value("User", user, ["email","user_image","mobile_no","birth_date","gender","location","full_name"])
	return p	
        
@frappe.whitelist()
def get_details(user = None):
	p = frappe.db.get_value("Souvenir", user, ["email","image","contact_no","date_of_birth","gender","address","name1"])
	return p	
        
```

Javascript code
In the file /souvenir_form/souvenir_form/web_form/souvenir/souvenir.js 
```js
  let data = frappe.web_form.get_values();
    if (!data) {
      frappe.call({
        method:
          "souvenir_form.souvenir_form.web_form.souvenir.souvenir.get_detail",
        args: { user: frappe.session.user },
        callback: (data) => {
          if (data.message) {
            frappe.web_form.set_value("email", data.message[0]);
            frappe.web_form.set_value("image", data.message[1]);
            frappe.web_form.set_value("contact_no", data.message[2]);
            frappe.web_form.set_value("date_of_birth", data.message[3]);
            frappe.web_form.set_value("gender", data.message[4]);
            frappe.web_form.set_value("address", data.message[5]);
            frappe.web_form.set_value("name1", data.message[6]);
          }
        },
      });
    }

```

To install the app run the commands

- bech get-app app https://github.com/Pawandeep16/NewSov.git
- bench --site sitename install-app appname
