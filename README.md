# Add-records-CRM-to-Creator-Picklist-Map-for-related-info
A Creator script that loads related records from Zoho CRM and adds as values to a picklist (radio) field on Zoho Creator, then gets related info by creating maps and storing them in multi-line fields in Creator.

## Core Idea
Zoho Creator has a nifty little feature that enables you to add values to a picklist (radio) field via Deluge. With the function, you can fetch records from Zoho CRM and dynamically populate them as field values in the Creator picklist field. Simultaneously, the function creates map strings (key-value pairs) containing relevant info based on the picklist, and adds to multi-line fields. This map strings will then be used by the function to execute certain actions upon form submission.

## Example Case
Staffs of Company A wants a Creator Form, accessible from the Contacts module in CRM. In that Form, they would like to be able to select one of the contacts's Course Enrollment (another module), then reassign that course's Program Coordinator (another module) for the student - there are 3 modules at play here.

## Configuration - CRM
* Create a Custom Button on CRM in the desired module
  * For the Form to be accessible via Contacts in CRM, a custom button can be created in the Contacts module.
  *   Create Button > Where would you like to place the button? (Select: View Page) > What action would you like the button to perform? (Writing Function)
  *  Insert the function below (this will open up the Creator form with the prefilled Contact ID)
      ```javascript  
      url = "<insert_creator_form_link_here>?Contact_ID=" + contactid;
      openUrl(url,"new window");
      return "";  
      ```
## Configuration - Creator Form Fields
* Contact ID 
  * Number field
  * Stores the main ID of the record
  * The function will get the Contact's related Course Enrollments
  * Set field this as **admin only**.
* Which Course Enrollment would you like to Update?
  * Picklist (radio) field
  * Field values will be dynamically populated with the Contact's related Course Enrollment names
* Course Enrollment Name and ID Map
  * Multi-line field
  * Stores the key-value pair (name & ID) of the selected Course Enrollment 
  * Set field this as **admin only**.
* Who should become the new Program Coordinator?
  * Picklist (radio) field
  * Field values will be populated with the all Program Coordinator names from CRM
* Program Coordinator Name and ID Map
  * Multi-line field
  * Loads the key-value pair (name & ID) of all Program Coordinators
  * Set field this as **admin only**.

## Tutorial
### On Load Creator Script
The following script needs to be written **on load** of the Creator form.
* Record Event (Created) > Form Event (Load of the Form)

### Get the Related Course Enrollments and Add to the Radio & Map Fields
Get the related Course Enrollments from the Contact ID, then use a `for loop` to iterate though each Course Enrollments to get the name and ID. In the `loop`, use the `ui.add` function in Creator to populate the Course Enrollment names to the **"Which Course Enrollment would you like to Update?"** radio field. String manipulation is applied to create the name and ID map to add to the **Course Enrollment Name and ID Map** multi-line field.

```javascript
courseenrollments = zoho.crm.getRelatedRecords("Student_Enrolled","Contacts",input.Contact_ID);
string = "";
for each  c in courseenrollments
{
	input.Which_Course_Enrollment_would_you_like_to_Update:ui.add(c.get("Name"));
	string = string + "\"" + c.get("Name") + "\"" + ":" + c.get("id") + ",";
}
mapstring = "{" + string + "}";
mapstring = mapstring.removeLastOccurence(",");
input.Course_Enrollment_Name_and_ID_Map = mapstring;
```

### Get all Program Coordinators to Add to Radio and Map Fields
Get all Program Coordinators from CRM, then use a `for loop` to iterate though each Program Coordinator to get the name and ID. In the `loop`, use the `ui.add` function in Creator to populate the Program Coordinator names to the **"Who should become the new Program Coordinator?"** radio field. String manipulation is applied to create the name and ID map to add to the **Program Coordinator Name and ID Map** multi-line field.

```javascript
progs = zoho.crm.getRecords("Program_Coordinators");
string = "";
for each  p in progs
{
	input.Who_should_become_the_new_Program_Coordinator:ui.add(p.get("Name"));
	string = string + "\"" + p.get("Name") + "\"" + ":" + p.get("id") + ",";
}
mapstring = "{" + string + "}";
mapstring = mapstring.removeLastOccurence(",");
input.Program_Coordinator_Name_and_ID_Map = mapstring;
```

### Submission Actions
An **on submission** Creator script is to be written to execute the actions (the submission actions script is arbitrary and will not be elaborated here). Instead, here are tips on how to use the map string field that you have created in the **on load** script above.

To convert the map string field values to map, use the `toMap()` function.
```javascript
enidmap = input.Course_Enrollment_Name_and_ID_Map.toMap();
progidmap = input.Program_Coordinator_Name_and_ID_Map.toMap();
```
Now that they are maps, you can easily use a `.get()` function to get the information to execute the actions that you need.
```javascript
enrollmentid = enidmap.get(input.Which_Course_Enrollment_would_you_like_to_Update).toLong();
progid = progidmap.get(input.Who_should_become_the_new_Program_Coordinator);
```
