---
layout: post
title: "RSVP Form with Google Apps Script"
date: 2023-03-18
tags: [javascript]
splash_img_source: /assets/images/blogPosts/googleRSVP_guestList.jpg
splash_img_caption: configuring fans
permalink: "blog/rsvp-form"
---

Here are some advantages to doing it this way:

* Convenience: Your guest's RSVP responses automatically get written to a Google Sheet and emailed to you
* Control: Fully integrates into your website and styled by you. You control every detail. 
* Simplicity: No "backend" to maintain. Just one Javascript file that can be hosted anywhere (e.g. Github Pages)
* Price: Free to use, don't need to pay for hosting.
* Privacy: you're collecting the data yourself.

### Overview
Overview goes here.

### Step 1: Guest Lookup Function
_NOTE: For my wedding, I created a separate Gmail account, and used that account to make these Google sheets and code. You may also want to do that before proceeding._

In Google Sheets, create a spreadsheet for your guest list. It should have 3 columns: (name, inviteID, hasRsvpd), and the data should start in the second row, like so:

![guest_list Google Sheet](/assets/images/blogPosts/googleRSVP_guestList.jpg){:style="display:block; margin-left:auto; margin-right:auto"}
<br/>

While we're here, let's create the Apps Script that will power the server side of things. In Google sheets, go to `Extensions` >> `Apps Script`. Replace the default "myFunction" with this:

```javascript
// validation
function doGet(e) {
    var inputName = e.parameter.name;
    var sheet = SpreadsheetApp.getActiveSheet();
    var lastRow = sheet.getLastRow();
    var nameFound = false;
    var id;
  
    inputName = inputName.toLowerCase();
    isRSVPd = "0"
  
    // loop through list of all names to check if it matches input name
    for (var i = 2; i <= lastRow; i++) {
      var thisName = sheet.getRange(i, 1).getValue().toString().toLowerCase();
      if (thisName === inputName) {
        nameFound = true;
        id = sheet.getRange(i, 2).getValue();
        isRSVPd = sheet.getRange(i, 3).getValue();
        break;
      }
    }
  
    // already RSVPd?
    if (isRSVPd=="1"){
      return ContentService.createTextOutput("Already RSVPd");
    }
  
    // loop through the names again to find other names with that ID
    // turn them into a comma separated list
    var outNames = ""
    for (var i = 2; i <= lastRow; i++) {
      var thisID = sheet.getRange(i, 2).getValue()
      if (thisID == id) {
        outNames = outNames + "," + sheet.getRange(i, 1).getValue().toString()
      }
    }
  
    // output the results back to the form. will be comma delimited, like: id, name1, name2
    var out;
    if (nameFound) {
      out = id.toString() + outNames
    } else {
      out = 'Name not found'
    }
  
    return ContentService.createTextOutput(out);
  }
```

When the Apps Script receives a GET request, it will run this code. The input is a guest's name, and the output is a comma delimited string containing the inviteID for that guest's party, and the name of each guest in that party.

If it can't find the guest's name in your list, then the output will be "Name not found", and if the guest has already RSVP'd, the output will be "Already RSVPd".

Next, you'll need to deploy the web app. Hit the big "Deploy" button at the top, then "New deployment". There, `Select Type` >> `Web app`. Enter a description. For "Execute as", leave the default (your gmail account), and importantly, switch "Who has access" to "Anyone".

![deploy Apps Script](/assets/images/blogPosts/googleRSVP_deploy.jpg){:style="display:block; margin-left:auto; margin-right:auto"}

Since you're allowing anyone on the internet to run this script, it's going to give you a bunch of warnings now. First, press "Authorize access". That will create a pop up where you will need to sign into your account. Then, there will be a screen telling you that "Google hasn't verified this app". You will need to press `Advanced` >> `Go to your_script_name (unsafe)`. Finally, you will need to press "Allow".

Your script is now live! Google will give you your deployment ID and a URL with which your app can be called.


### Step 2: Make the HTML Form
Now, you'll need an HTML form with just a guest lookup field and a submit button. You should also have a spot where errors will show up.

INSERT PICTURE OF STYLED FORM

In it's simplest form:
```html
<form id="form">
	<label>Look up your invitation:</label>
	<div>
		<input name="name" type="text" placeholder="Firstname Lastname">
	</div>
	<p>Enter your first and last name exactly as it appears on your invitation. If there is more than one name, enter just one of them.</p>
	<button type="submit">Continue</button>
	<div id="output"></div>
</form>
```

Now, you'll need the client-side Javascript function that will run when the user hits "Continue". That function will send the guest's name to the Apps Script, and will process the output. In its simplest form, that javascript function should look like this:

```javascript
document.getElementById('form').addEventListener('submit', function(event) {
    event.preventDefault();
    var name = this.elements.name.value;
    var xhr = new XMLHttpRequest();
    xhr.open('GET', 'https://script.google.com/macros/s/YOUR_GOOGLE_APPS_SCRIPT_ID/exec?name=' + name, true);
    xhr.onreadystatechange = function() {
      if (xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
        var idAndNames = xhr.responseText;
        if (idAndNames === "Name not found") {
          document.getElementById('output').innerHTML = "<div>Error: Couldn't find guest name.</div>";
        } else if (idAndNames === "Already RSVPd"){
          document.getElementById('output').innerHTML = `<div>Error: Already RSVP'd.</div>`;
        } else {
            document.getElementById('output').innerHTML = "<div>Name found!</div>";
        }
      }
    };
    xhr.send();
  });
```

At this point, we have a working setup. If you open your form locally, type in a name and press "Continue", you should get a result.

![Name Check Example](/assets/images/blogPosts/googleRSVP_nameCheck.jpg){:style="display:block; margin-left:auto; margin-right:auto"}

### Step 2: Collect responses
Next, we collect whatever information we need. The following code just collects whether each guest is coming to the event, but it's intentionally flexible. It can handle an arbitrary number of guests on each "invite ID":
```javascript
function collectResponses(idAndNames){
  const data = [];
  const inputArray = idAndNames.split(',');
  const inviteID = inputArray.shift() || "NA";

  // Iterate through the remaining names and create an object for each person
  inputArray.forEach((name, index) => {
    if (name) {
      data.push({
        inviteID: inviteID,
        name: name,
        attending: 0,
        id: `person${index}` // checkbox id, so submit button can find it
      });
    }
  });

  // Write HTML and checkboxes for each person
  document.getElementById('entireForm').innerHTML = `
    <div>
      We found your RSVP!<br/><br/>
	${data.map(
          person => `${person.name}: <input type="checkbox" id="${person.id}" /> Attending?<br/><br/>`
        ).join('')}
      <button id="submit">Submit</button>
    </div>
  `;

  // Submit button code
  document.getElementById('submit').addEventListener('click', () => {
    // write each person's attending value to "data"
    data.forEach(person => {
      person.attending = document.getElementById(person.id).checked ? 1 : 0;
    });

    console.log(data);
  });
}
```
This code will take an input we got back from Google (like `5,Bob Johnson,Alice Johson`), and turn it into an array called "data" (that looks like `[[inviteID: "5", name: "Bob Johnson", attending: 0],[inviteID: "5", name: "Alice Johnson", attending: 0]`). Furthermore, it will generate an HTML form like this, with a "Submit" button which will populate the "attending" value of "data" for each guest:

![Collect Responses Example](/assets/images/blogPosts/googleRSVP_collectResponses.jpg){:style="display:block; margin-left:auto; margin-right:auto"}

### Step 3: Submit Form, Write Results and Email
Now we need a function to so send the contents of "data" back to our Google script:


### Step 4: Styling the Form
