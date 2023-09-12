# Use Cases
The view of the use-cases - see the "use case" diagram in `diagrams.drawio` file.

The following is element catalog for it:


 N. | Name | Description 
--- | --- | ---
ACT-1 | EndUser | A user of the system, general public
ACT-2 | Admin | Company employee, who has elevated permissions
ACT-3 | Accountant | Company employee, who can change system's configuration 
UC-1 | EndUser registers an account | The user registers a new account in a system using an email and a phone number
UC-2 | EndUser accesses list of existing tax calculations | After login, the EndUser is able to see the list of existing (previous) tax calculations and can preview the results or resume the entry process.
UC-3 | EndUser enters personal information | The user navigates to the first page of tax entry process and enters personal information (Name, Surname, Tax ID, Single or married, are there any dependents)
UC-4 | EndUser enters basic tax info | The user proceeds through the tax entry process and enters basic tax info on separate page (Tax Year,  answers the question “Did you use software to complete last year’s taxes?”)
UC-5 | EndUser enters income data | The user proceeds through the tax entry process and enters income data on a separate page (Taxable Wages, Spouse's taxable wages, Federal taxes withheld and estimated payments, Interest income, Unemployment income)
UC-6 | EndUser answers additional questions | The user proceeds through the tax entry process and answers additional questions that system asks
UC-7 | EndUser enters deductions info | The user proceeds through the tax entry process and enters deductions info on a dedicated page
UC-8 | EndUser sends taxes to the regulator | As the final stage of tax entry process, the user is presented with possibility and can send taxes to the regulator
UC-9 | EndUser or Admin or Accountant logs in | EndUser, Admin or Accountant enters login and password and confirmation code and is able to access the system configuration console
UC-10 | Admin manages user access | After login, Admin enters the user management page and is able to list/create/edit/delete and assign roles (Admin/Accountant) to the system users
UC-11 | Accountant changes additional questions configuration | After login, Accountant enters the additional questions configuration page and can edit questions order, create new, edit and delete existing questions. Questions can be of different types (free text, number, dropdown) and can be shown based on condition (previous questions answers)
UC-12 | Accountant changes tax rules configuration | After login, Accountant enters the tax rules configuration page and can edit the rules. The rules take into account the basic information enterd by user and answers to the additional questions.