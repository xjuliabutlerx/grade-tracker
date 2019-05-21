# grade-tracker
Originally an app made on Code.org, this app sorts a student's classes and generates unweighted and weighted GPA. It can manage multiple students' classes and GPA's. It does use permanent data storage.
firstTimeSetup();
var currUser;
//var unweightedGPA;
//var weightedGPA;
var gradesIndex = 0;
var updateIndex = 0;
var editClass;
var editID = 0;
var editType = "";
var editCredits = 0;
var points = 0;
var GPAid = 0;
var lowClass = "";
var low = 4.00;

onEvent("loginBtn", "click", function() {
  var username = getText("usernameInput");
  username = username.trim();
  var password = getText("passwordInput");
  password = password.trim();
  console.log("Username: " + username + "\nPassword: " + password);
  readRecords("loginTable", {username: username, password: password}, function(records) {
    if (records.length === 0) {
      console.log("Length is 0.");
      setProperty("usernameInput", "background-color", "red");
      setProperty("passwordInput", "background-color", "red");
      showElement("noAccountTxt");
      setProperty("noAccountTxt", "text-color", "red");
      setText("noAccountTxt", "Your password and/or username is incorrect. Please try again or make a new account.");
      change("usernameInput", "noAccountTxt");
      change("passwordInput", "noAccountTxt");
    }
    else {
      for (var i = 0; i < records.length; i++) {
        console.log(records[i].username + "\n" + records[i].password);
        if (username == records[i].username && password == records[i].password) {
          setScreen("homePage");
          setText("usernameInput", "");
          setText("passwordInput", "");
          currUser = username;
          setupApp(currUser);
          updateClassList();
          calcGPA();
        }
      }
    }
  });
  hideElement("noAccountTxt");
});

/* Problem #2: When logging out of the account, the screen does
switch back to the log-in screen, but the username and password
entry boxes turn red as if someone typed in something that did
not exist. (2/19/19)
        
RESOLVED on 2/19/19 (I added a return function at the end of each
if-else statement.)
*/

onEvent("signupBtn", "click", function() {
  var newUser = getText("newUserTxt");
  newUser = newUser.trim();
  var newPass = getText("newPassTxt");
  newPass = newPass.trim();
  var realName = getText("realNameTxt");
  realName = realName.trim();
  console.log("Username: " + newUser + "\nPassword: " + newPass + "\nName: " + realName);
  readRecords("loginTable", {username: newUser}, function(records) {
    if (records.length === 0) {
      createRecord("loginTable", {username: newUser, password: newPass, realName: realName}, function(record) {
        console.log(record);
      });
      createRecord("gpaTable", {username: newUser, unweighted: 0, weighted: 0, creditsEarned: 0}, function(record) {
        console.log(record);
      });
      setText("usernameInput", "");
      setProperty("usernameInput", "background-color", "");
      setText("passwordInput", "");
      setProperty("passwordInput", "background-color", "");
      hideElement("noAccountTxt");
      setText("noAccountTxt", "");
      setProperty("noAccountTxt", "text-color", "#333333");
      setScreen("loginScreen");
      showElement("noAccountTxt");
      setText("noAccountTxt", "Log into your new account with your username and password!");
    }
    else {
      setProperty("newUserTxt", "background-color", "red");
      setProperty("alreadyExistsTxt", "text-color", "red");
      showElement("alreadyExistsTxt");
      setText("alreadyExistsTxt", "An account with this username already exists.");
      change("newUserTxt", "alreadyExistsTxt");
    }
  });
});

/* Problem #1: If the username is the same as another one, 
then it will follow the correct code, but since it is on
a loop, then it will check the others and then proceed with
the wrong code. I tried using "return" but it didn't work
because of the loop. I don't know how to fix this right
now. (2/14/19)
          
RESOLVED on 2/19/19 (Put in a specific username for the
readRecords function to find.)

****

Problem #3: The onEvent function wasn't making a new account.
          
RESOLVED on 2/20/19 (Removed the for loop.)
*/

//BUTTON ON-EVENTS (switching screens only)
//Miscellaneous
onEvent("newAccBtn", "click", function() {
  setText("newUserTxt", "");
  setText("newPassTxt", "");
  setText("realNameTxt", "");
  setProperty("newUserTxt", "background-color", "");
  hideElement("alreadyExistsTxt");
  setText("alreadyExistsTxt", "");
  setProperty("alreadyExistsTxt", "text-color", "#333333");
  setScreen("newAccount");
  showElement("newAccountTxt");
  setText("newAccountTxt", "Enter a username and password below and remember to write it down!");
});

onEvent("backLoginBtn", "click", function() {
  setText("usernameInput", "");
  setProperty("usernameInput", "background-color", "");
  setText("passwordInput", "");
  setProperty("passwordInput", "background-color", "");
  hideElement("noAccountTxt");
  setText("noAccountTxt", "");
  setProperty("noAccountTxt", "text-color", "#333333");
  setScreen("loginScreen");
});

onEvent("gradesBtn2", "click", function() {
  updateGrades(0);
  setScreen("gradesScreen");
});

onEvent("gradesBtn3", "click", function() {
  updateGrades(0);
  setScreen("gradesScreen");
});

onEvent("gradesBtn4", "click", function() {
  updateGrades(0);
  setScreen("gradesScreen");
});

//On Home Page
onEvent("settingsBtn", "click", function() {
  setScreen("settingsScreen");
});

onEvent("gradesBtn", "click", function() {
  updateGrades(0);
  setScreen("gradesScreen");
});

onEvent("logOutBtn", "click", function() {
  setProperty("usernameInput", "background-color", "");
  setText("usernameInput", "");
  setProperty("passwordInput", "background-color", "");
  setText("passwordInput", "");
  setText("noAccountTxt", "");
  setText("updateNameTxt", "");
  setText("updateCreditsTxt", "");
  setText("typeOfDD2", "Regular");
  setScreen("loginScreen");
  currUser = "";
  lowClass = "";
  low = 4.00;
});

onEvent("addClassesBtn", "click", function() {
  setScreen("classListScreen");
});

onEvent("gpaBtn", "click", function() {
  calcGPA();
  setScreen("calcGPAScreen");
});

onEvent("manageBtn", "click", function() {
  setEditList(0);
  setScreen("manageScreen");
});

onEvent("statsBtn", "click", function() {
  setupStats();
  setScreen("statsScreen");
});

//On Settings Page
onEvent("updateBtn", "click", function() {
  var currID;
  var changedPass = getText("currPassword");
  changedPass = changedPass.trim();
  var changedName = getText("currName");
  changedName = changedName.trim();
  readRecords("loginTable", {username: currUser}, function(records) {
    for (var i = 0; i < records.length; i++) {
      currID = records[i].id;
      console.log("Update ID: " + currID);
    }
    updateRecord("loginTable", {id: currID, username: currUser, password: changedPass, realName: changedName}, function(record) {
      console.log(record);
      setText("classListTxt", record.realName + "'s Classes");
      setText("welcomeTxt", "Welcome, " + record.realName + "!");
      setText("currUsername", record.username);
      setText("currPassword", record.password);
      setText("currName", record.realName);
    });
  });
  setProperty("updateResTxt", "text-color", "green");
  setText("updateResTxt", "Settings Updated!");
  showElement("updateResTxt");
  //setupApp(currUser);
  setTimeout(function() {
    setProperty("updateResTxt", "text-color", "#333333");
    setText("updateResTxt", "");
    hideElement("updateResTxt");
  }, 1500);
});

onEvent("deleteBtn", "click", function() {
  showElement("warningTxt");
  showElement("deleteAccBtn");
  showElement("keepAccBtn");
});

onEvent("keepAccBtn", "click", function() {
  hideElement("warningTxt");
  hideElement("deleteAccBtn");
  hideElement("keepAccBtn");
});

onEvent("deleteAccBtn", "click", function() {
  readRecords("loginTable", {username: currUser}, function(records) {
    for (var i = 0; i < records.length; i++) {
      console.log("ID number: " + records[i].id);
      deleteRecord("loginTable", {id: records[i].id}, function(success) {
        console.log("Account deletetion: " + success);
      });
    }
  });
  readRecords("classTable", {username: currUser}, function(records) {
    for (var i = 0; i < records.length; i++) {
      console.log("ID number: " + records[i].id);
      deleteRecord("classTable", {id: records[i].id}, function(success) {
        console.log("Class deletion: " + success);
      });
    }
  });
  readRecords("gpaTable", {username: currUser}, function(records) {
    for (var i =0 ; i < records.length; i++) {
      console.log("ID number: " + records[i].id);
      deleteRecord("gpaTable", {id: records[i].id}, function(success) {
        console.log("GPA deletion: " + success);
      });
    }
  });
  
  setScreen("loginScreen");
  currUser = "";
});

//On Grades Screen
onEvent("lastBtn", "click", function() {
  readRecords("classTable", {username: currUser}, function(records) {
    gradesIndex--;
    gradesIndex = wrap(gradesIndex, 0, records.length-1);
    updateGrades(gradesIndex);
    //singleGPA();
  });
});

onEvent("nextBtn", "click", function() {
  readRecords("classTable", {username: currUser}, function(records) {
    gradesIndex++;
    gradesIndex = wrap(gradesIndex, 0, records.length-1);
    updateGrades(gradesIndex);
    //singleGPA();
  });
});

onEvent("gradesScreen", "keydown", function(event) {
  if (event.key == "Left") {
    readRecords("classTable", {username: currUser}, function(records) {
      gradesIndex--;
      gradesIndex = wrap(gradesIndex, 0, records.length-1);
      updateGrades(gradesIndex);
      //singleGPA();
    });
  } else if (event.key == "Right") {
    readRecords("classTable", {username: currUser}, function(records) {
      gradesIndex++;
      gradesIndex = wrap(gradesIndex, 0, records.length-1);
      updateGrades(gradesIndex);
      //singleGPA();
    });
  }
});

onEvent("saveGradesBtn", "click", function() {
  var q1grade = getText("q1gradeDD");
  var q2grade = getText("q2gradeDD");
  var q3grade = getText("q3gradeDD");
  var q4grade = getText("q4gradeDD");
  var className = getText("className");
  var classType = getText("classType");
  var credits = getText("credits");
  //var updateID;
  readRecords("classTable", {username: currUser, className: className}, function(records) {
    console.log("Updating Record: " + records);
    for (var i = 0; i < records.length; i++) {
      updateRecord("classTable", {id: records[i].id, username: currUser, className: className, credits: credits, type: classType, Q1: q1grade, Q2: q2grade, Q3: q3grade, Q4: q4grade}, function(record, success) {
        console.log(record);
        if (success) {
          singleGPA();
          setProperty("gradeUpdateRes", "text-color", "green");
          showElement("gradeUpdateRes");
          setText("gradeUpdateRes", "Grades Saved!");
          setTimeout(function() {
            hideElement("gradeUpdateRes");
            setText("gradeUpdateRes", "");
            setProperty("gradeUpdateRes", "text-color", "#333333");
          }, 1500);
        }
      });
    }
  });
  
});

//On Manage Classes Screen
onEvent("returnBtn", "click", function() {
  setScreen("classListScreen");
});

onEvent("addClassBtn", "click", function() {
  var newClass = "";
  newClass = getText("newClassTxt");
  newClass = newClass.trim();
  newClass = newClass.toUpperCase();
  var newCredits = 0;
  newCredits = getText("newCreditsTxt");
  newCredits = 1.00*newCredits; //Will make it into an integer
  var classType = "";
  classType = getText("typeOfDD");
  classType = classType.toString();
  console.log("Type of Variable: " + typeof(classType));
  //console.log("New Class: " + newClass + "\nCredits: " + newCredits + "\nClass Type: " + classType);
  if (newClass == "" || newCredits == "" || isNaN(newCredits) == true) {
    console.log("One or more fields are invalid.");
    setProperty("newClassTxt", "background-color", "red");
    setProperty("newCreditsTxt", "background-color", "red");
    setProperty("addClassResTxt", "text-color", "red");
    showElement("addClassResTxt");
    setText("addClassResTxt", "One or more fields are invalid.");
    change("newClassTxt", "addClassResTxt"); 
    change("newCreditsTxt", "addClassResTxt");
  }
  else {
    readRecords("classTable", {username: currUser}, function(records) {
      console.log(records);
      for (var i = 0; i < records.length; i++) {
        if (newClass == records[i].className) {
          setProperty("newClassTxt", "background-color", "red");
          setProperty("addClassResTxt", "text-color", "red");
          showElement("addClassResTxt");
          setText("addClassResTxt", "This class already exists!");
          change("newClassTxt", "addClassResTxt");
          return;
        }
      }
      createRecord("classTable", {username: currUser, className: newClass, credits: Number(newCredits), type: classType, Q1: "", Q2: "", Q3: "", Q4: "", unweightedGPA: "", weightedGPA: ""}, function(record) {
        console.log("User: " + record.username + "\nNew Class: " + record.className + "\nCredits: " + newCredits + "\nType: " + classType);
        setText("newClassTxt", "");
        setText("newCreditsTxt", "");
        setText("typeOfDD", "Regular");
        setProperty("addClassResTxt", "text-color", "green");
        showElement("addClassResTxt");
        setText("addClassResTxt", "Congratulations! Class Added!");
        setTimeout(function() {
          setText("addClassResTxt", "");
          setProperty("addClassResTxt", "text-color", "#333333");
          hideElement("addClassResTxt");
        }, 1500);
        updateClassList();
        setEditList(0);
      });
    });
  }
});

onEvent("manageScreen", "keydown", function(event) {
  if (event.key == "Left") {
    readRecords("classTable", {username: currUser}, function(records) {
      updateIndex--;
      updateIndex = wrap(updateIndex, 0, records.length-1);
      setEditList(updateIndex);
    });
  }
  if (event.key == "Right") {
    readRecords("classTable", {username: currUser}, function(records) {
      updateIndex++;
      updateIndex = wrap(updateIndex, 0, records.length-1);
      setEditList(updateIndex);
    });
  }
});

onEvent("leftBtn", "click", function() {
  readRecords("classTable", {username: currUser}, function(records) {
      updateIndex--;
      updateIndex = wrap(updateIndex, 0, records.length-1);
      setEditList(updateIndex);
    });
});

onEvent("rightBtn", "click", function() {
  readRecords("classTable", {username: currUser}, function(records) {
      updateIndex++;
      updateIndex = wrap(updateIndex, 0, records.length-1);
      setEditList(updateIndex);
    });
});

onEvent("saveClassBtn", "click", function() {
  var updateName = getText("updateNameTxt");
  updateName = updateName.trim();
  updateName = updateName.toString();
  updateName = updateName.toUpperCase();
  var updateCredits = getText("updateCreditsTxt");
  updateCredits = Number(updateCredits);
  var updateType = getText("typeOfDD2");
  updateType = updateType.toString();
  console.log("updateName: " + updateName);
  console.log("ID to Update: " + editID);
  readRecords("classTable", {username: currUser, className: updateName}, function(records) {
    console.log(records);
    console.log("Length: " + records.length);  
    
    if (records.length > 0) {
      console.log("THERE ARE CLASSES TO UPDATE.");
      var Q1;
      var Q2;
      var Q3;
      var Q4;
      var unweightedGPA;
      var weightedGPA;
      var className;
      for (var i = 0; i < records.length; i++) {
        className = records[i].className;
        Q1 = records[i].Q1;
        Q2 = records[i].Q2;
        Q3 = records[i].Q3;
        Q4 = records[i].Q4;
        unweightedGPA = records[i].unweightedGPA;
        weightedGPA = records[i].weightedGPA;
      }
      className = className.toString();
      Q1 = Q1.toString();
      Q2 = Q2.toString();
      Q3 = Q3.toString();
      Q4 = Q4.toString();
      unweightedGPA = Number(unweightedGPA);
      weightedGPA = Number(weightedGPA);
      
      updateRecord("classTable", {id: editID, username: currUser, className: className, credits: updateCredits, type: updateType, Q1: Q1, Q2: Q2, Q3: Q3, Q4: Q4, unweightedGPA: unweightedGPA, weightedGPA: weightedGPA}, function(record, success) {
        console.log("Updated Record: " + record);
        if (success) {
          setProperty("editResTxt", "text-color", "green");
          setText("editResTxt", "Class Updated!");
          showElement("editResTxt");
          setTimeout(function() {
            hideElement("editResTxt");
            setProperty("editResTxt", "text-color", "#333333");
            setText("editResTxt", "");
            }, 1500);
          updateClassList();
          setText("updateNameTxt", className);
          setText("updateCreditsTxt", updateCredits);
          setText("typeOfDD2", updateType);  
        }
      });
    } 
    else {
      readRecords("classTable", {username: currUser}, function(records) {
        if (records.length > 0) {
          for (var i = 0; i < records.length; i++) {
            if (editID == records[i].id) {
              var Q1 = records[i].Q1;
              var Q2 = records[i].Q2;
              var Q3 = records[i].Q3;
              var Q4 = records[i].Q4;
              var unweightedGPA = records[i].unweightedGPA;
              var weightedGPA = records[i].weightedGPA;
              updateRecord("classTable", {id: editID, username: currUser, className: updateName, credits: updateCredits, type: updateType, Q1: Q1, Q2: Q2, Q3: Q3, Q4: Q4, unweightedGPA: unweightedGPA, weightedGPA: weightedGPA}, function(record, success) {
                console.log("Updated Record: " + record);
                if (success) {
                  setProperty("editResTxt", "text-color", "green");
                  setText("editResTxt", "Class Updated!");
                  showElement("editResTxt");
                  setTimeout(function() {
                    hideElement("editResTxt");
                    setProperty("editResTxt", "text-color", "#333333");
                    setText("editResTxt", "");
                    }, 1500);
                  updateClassList();  
                  setText("updateNameTxt", updateName);
                  setText("updateCreditsTxt", updateCredits);
                  setText("typeOfDD2", updateType);
                }
              });
            } 
          }
        }
        else {
          console.log("NO CLASSES TO UPDATE");
          setProperty("editResTxt", "text-color", "red");
          setText("editResTxt", "No classes yet!");
          showElement("editResTxt");
          setTimeout(function() {
            hideElement("editResTxt");
            setProperty("editResTxt", "text-color", "#333333");
            setText("editResTxt", "");
          }, 1500);
          setText("updateNameTxt", "");
          setText("updateCreditsTxt", "");
          setText("typeOfDD2", "Regular");
        }
      });
    }  
  });  
});

onEvent("deleteClassBtn", "click", function() {
  var deleteClass = "";
  deleteClass = getText("updateNameTxt");
  deleteClass = deleteClass.trim();
  deleteClass = deleteClass.toUpperCase();
  readRecords("classTable", {username: currUser, className: deleteClass}, function(records) {
    if (records.length === 0) {
      setProperty("editResTxt", "text-color", "red");
      setText("editResTxt", "Class doesn't exist.");
      showElement("editResTxt");
      setTimeout(function() {
        hideElement("editResTxt");
        setProperty("editResTxt", "text-color", "#333333");
        setText("editResTxt", "");
      }, 1500);
      return;
    }
    else {
      var deleteID = 0;
      for (var i = 0; i < records.length; i++) {
        if (records[i].className == deleteClass) {
          deleteID = records[i].id;
        }
      }
      deleteRecord("classTable", {id: deleteID, username: currUser, className: deleteClass}, function(success) {
        console.log(records);
        if (success) {
          setProperty("editResTxt", "text-color", "green");
          setText("editResTxt", "Class deleted!");
          showElement("editResTxt");
          setTimeout(function() {
            hideElement("editResTxt");
            setProperty("editResTxt", "text-color", "#333333");
            setText("editResTxt", "");
          }, 1500);
          updateClassList();
          setEditList(0);
        }
      });
    }
  });
});

//On Calculate GPA Screen
onEvent("statsBtn1", "click", function() {
  setupStats();
  setScreen("statsScreen");
});

//Home Buttons
onEvent("homeBtn1", "click", function() {
  calcGPA();
  setScreen("homePage");
});

onEvent("homeBtn2", "click", function() {
  setScreen("homePage");
});

onEvent("homeBtn3", "click", function() {
  calcGPA();
  setScreen("homePage");
});

onEvent("homeBtn4", "click", function() {
  setScreen("homePage");
});

onEvent("homeBtn5", "click", function() {
  setScreen("homePage");
});

//FUNCTIONS
function change (textbox, result) {
  onEvent(textbox, "change", function() {
    setProperty(textbox, "background-color", "");
    setProperty(textbox, "text-color", "#333333");
    setProperty(result, "text-color", "#333333");
    setText(result, "");
    hideElement(result);
  });
}

function setupApp (currUser) {
  readRecords("loginTable", {username: currUser}, function(records) {
    for (var i = 0; i < records.length; i++) {
      setText("classListTxt", records[i].realName + "'s Classes");
      //ON SETTINGS SCREEN
      setText("welcomeTxt", "Welcome, " + records[i].realName + "!");
      setText("currUsername", records[i].username);
      setText("currPassword", records[i].password);
      setText("currName", records[i].realName);
    }
  });
}

function updateClassList () {
  readRecords("classTable", {username: currUser}, function(records) {
    var classString = "";
    var classStringFull = "";
    var creditsString = "";
    var freq = {};
    freq.regNum = 0;
    freq.honorsNum = 0;
    freq.apNum = 0;
    if (records.length === 0) {
      setText("classTxt", "No classes yet! To add classes, go to 'classes' and then 'manage.'");
      setText("classesTxt", "No classes yet! To add classes, click 'manage.'");
      setText("creditsTxt", "No classes yet!");
    }
    else {
      for (var h = 0; h < records.length; h++) {
        classStringFull += records[h].className + "\n";
      }
      for (var i = 0; i < records.length; i ++) {
        if (records[i].className.length > 13) {
          classString += records[i].className.substring(0, 14) + "...\n";
        }
        else {
          classString += records[i].className + "\n";
        }
        console.log("Shortened Version: \n" + classString);
      }
      for (var j = 0; j < records.length; j++) {
        creditsString += records[j].credits + "\n";
      }
      console.log("Classes: \n" + classString);
      setText("classTxt", classStringFull);
      setText("classesTxt", classString);
      setText("creditsTxt", creditsString);
      for (var c = 0; c < records.length; c++) {
        if (records[c].type == "Regular") {
          freq.regNum++;
        }
        else if (records[c].type == "Honors") {
          freq.honorsNum++;
        }
        else {
          freq.apNum++;
        }
      }
      var data = [{type: "Regular", frequency: freq.regNum}, {type: "Honors", frequency: freq.honorsNum}, {type: "AP", frequency: freq.apNum}];
      var myOptions={};
      myOptions.title = "Types of Classes";  
      myOptions.colors = ["#62EF70", "#629BEF", "#EF9862"];
      myOptions.legend = {position: "none"};
      drawChart("typesClassesChart", "pie", data, myOptions);
    }
  });
}

function firstTimeSetup () {
  setScreen("loginScreen");
  hideElement("noAccountTxt");
  hideElement("newAccountTxt");
  hideElement("alreadyExistsTxt");
  hideElement("warningTxt");
  hideElement("deleteAccBtn");
  hideElement("keepAccBtn");
  hideElement("addClassResTxt");
  hideElement("updateResTxt");
  hideElement("gradeUpdateRes");
  hideElement("editResTxt");
}

function singleGPA () {
  var Q1 = "";
  Q1 = getText("q1gradeDD");
  var Q2 = "";
  Q2 = getText("q2gradeDD");
  var Q3 = "";
  Q3 = getText("q3gradeDD");
  var Q4 = "";
  Q4 = getText("q4gradeDD");
  
  if (Q1 == "") {
    setText("currGPATxt", "N/A");
    return;
  }
  else {
    var GPAclass = getText("className");
    var totalQ = 1;
    var pointArray = [];
    var calcGPA = 0;
    var calcGPAw = 0;
    var GPAcredits = getText("credits");
    var GPAtype = getText("classType");
    
    appendItem(pointArray, pointSystem(getText("q1gradeDD")));
    console.log(pointArray + "\nNumber of Quarters: " + totalQ);
    if (Q2 !== "") {
      appendItem(pointArray, pointSystem(getText("q2gradeDD")));
      totalQ++;
      console.log(pointArray + "\nNumber of Quarters: " + totalQ);
    }
    if (Q3 !== "") {
      appendItem(pointArray, pointSystem(getText("q3gradeDD")));
      totalQ++;
      console.log(pointArray + "\nNumber of Quarters: " + totalQ);
    }
    if (Q4 !== "") {
      appendItem(pointArray, pointSystem(getText("q4gradeDD")));
      totalQ++;
      console.log(pointArray + "\nNumber of Quarters: " + totalQ);
    }
    
    var pointSum = 0;
    for (var i = 0; i < pointArray.length; i++) {
      pointSum += pointArray[i];
    }
    calcGPA = pointSum/totalQ;
    calcGPAw = calcGPA;
    calcGPA = round(calcGPA);
    console.log(GPAclass + "'s unweighted GPA: " + calcGPA);
    console.log(GPAclass + "'s weighted GPA: " + calcGPAw);
    
    if (GPAtype == "Honors") {
      calcGPAw += 0.5;
      calcGPAw = round(calcGPAw);
      console.log("New Weighted GPA: " + calcGPAw);
    }
    if (GPAtype == "AP") {
      calcGPAw += 1.0;
      calcGPAw = round(calcGPAw);
      console.log("New Weighted GPA: " + calcGPAw);
    }
    
    readRecords("classTable", {username: currUser, className: GPAclass}, function(records) {
      if (records.length === 0) {
        //Set up error message?
      }
      for (var i = 0; i < records.length; i++) {
        GPAid = records[i].id;
        GPAid = Number(GPAid);
        console.log("Updating GPA ID: " + GPAid);
      }
      
      updateRecord("classTable", {id: GPAid, username: currUser, className: GPAclass, credits: GPAcredits, type: GPAtype, Q1: Q1, Q2: Q2, Q3: Q3, Q4: Q4, unweightedGPA: calcGPA, weightedGPA: calcGPAw}, function(record, success) {
        console.log(record);
        if (success) {
          setText("currGPATxt", calcGPA);
          setText("gradeUpdateRes", "GPA updated!");
          setProperty("gradeUpdateRes", "text-color", "green");
          showElement("gradeUpdateRes");
          setTimeout(function() {
            hideElement("gradeUpdateRes");
            setText("gradeUpdateRes", "");
            setProperty("gradeUpdateRes", "text-color", "#333333");
          }, 1500);
        }
      });
    });
  }
}

function calcGPA () {
  readRecords("classTable", {username: currUser}, function(records) {
    if (records.length === 0) {
      setText("unweightedGPATxt", "N/A");
      setText("weightedGPATxt", "N/A");
      setText("currentGPATxt", "N/A");
      return;
    }
    for (var i = 0; i < records.length; i++) {
      if (records[i].Q1 == "") {
        setText("unweightedGPATxt", "N/A");
        setText("weightedGPATxt", "N/A");
        setText("currentGPATxt", "N/A");
        return;
      }
    }
    var totalPoints1 = 0;
    var totalPoints2 = 0;
    var totalCredits = 0;
    //var totalClass = records.length;
    var cumGPA = 0;
    var cumGPAw = 0;
    console.log(records);
    for (var j = 0; j < records.length; j++) {
      totalPoints1 += records[j].unweightedGPA * records[j].credits;
      console.log("Points for Unweighted: " + totalPoints1);
      totalPoints2 += records[j].weightedGPA * records[j].credits;
      console.log("Points for Weighted: " + totalPoints2);
      totalCredits += Number(records[j].credits);
      console.log("Credits: " + totalCredits);
    }
    cumGPA = totalPoints1/totalCredits;
    cumGPAw = totalPoints2/totalCredits;
    cumGPA = round(cumGPA);
    cumGPAw = round(cumGPAw);
    console.log("Unweighted GPA: " + cumGPA + "\nWeighted GPA: " + cumGPAw);
    setText("unweightedGPATxt", cumGPA);
    setText("currentGPATxt", cumGPA);
    setText("weightedGPATxt", cumGPAw);
  });
}

function pointSystem (letter) {
  if (letter == "A+" || letter == "A") {
    points = 4.00;
    return points;
  } else if (letter == "A-") {
    points = 3.70;
    return points;
  } else if (letter == "B+") {
    points = 3.30;
    return points;
  } else if (letter == "B") {
    points = 3.00;
    return points;
  } else if (letter == "B-") {
    points = 2.70;
    return points;
  } else if (letter == "C+") {
    points = 2.30;
    return points;
  } else if (letter == "C") {
    points = 2.00;
    return points;
  } else if (letter == "C-") {
    points = 1.70;
    return points;
  } else if (letter == "D+") {
    points = 1.30;
    return points;
  } else if (letter == "D") {
    points = 1.00;
    return points;
  } else {
    points = 0.00;
    return points;
  }
}

function setEditList (index) {
  readRecords("classTable", {username: currUser}, function(records) {
    if (records.length === 0) {
      setText("updateIndexTxt", " 0 of 0");
      return;
    }
    else {
      for (var i = 0; i < records.length; i++) {
        if (i == index) {
          editClass = records[index].className;
          editID = records[index].id;
          editCredits = records[index].credits;
          editType = records[index].type;
          setText("updateIndexTxt", (index + 1) + " of " + records.length);
          setText("updateNameTxt", records[index].className);
          setText("updateCreditsTxt", records[index].credits);
          setText("typeOfDD2", records[index].type);
          if (records[i].unweightedGPA !== "") {
            setText("currGPATxt", records[index].unweightedGPA);
          } else {
            setText("currGPATxt", "N/A");
          }
        }
      }
    }
  });
}

/* Problem #5 (2/26/19): After adding a class, the "update/delete a class would not
refresh after adding a class. 

RESOLVED (2/28/19): I added a for loop that found the index that matched
the variable "updateIndex."
*/

function updateGrades (index) {
  readRecords("classTable", {username: currUser}, function(records) {
    var type = records[index].type;
    type = type.toString();
    setText("classIndexTxt", (index + 1) + " of " + records.length);
    setText("className", records[index].className);
    setText("classType", type);
    setText("credits", records[index].credits);
    if (records[index].unweightedGPA == undefined) {
      setText("currGPATxt", "N/A");
      setText("currentGPATxt", "N/A");
    } else {
      setText("currGPATxt", records[index].unweightedGPA);
      setText("currentGPATxt", records[index].unweightedGPA);
    }
    if (records[index].Q1 !== undefined) {
      if (records[index].Q1 == null) {
        //setText("q1gradeDD", "None");
        
      }
      else {
        setText("q1gradeDD", records[index].Q1);
      }
    }
    if (records[index].Q2 !== undefined) {
      if (records[index].Q2 == null) {
        //setText("q2gradeDD", "None");
        
      }
      else {
        setText("q2gradeDD", records[index].Q2);
      }
    }
    if (records[index].Q3 !== undefined) {
      if (records[index].Q3 == null) {
        //setText("q3gradeDD", "None");
        
      }
      else {
        setText("q3gradeDD", records[index].Q3);
      }
    }
    if (records[index].Q4 !== undefined) {
      if (records[index].Q4 == null) {
        //setText("q4gradeDD", "None");
        
      }
      else {
        setText("q4gradeDD", records[index].Q4);
      }
    }
  });
}
/* Problem #4 (2/23/19): It sets every grade to "A" regardless of if there are
grades present.
    
RESOLVED (2/23/19) - changed the condition in the second "if" statement
to records[index].Q# == null.
*/

function setupStats () {
  readRecords("classTable", {username: currUser}, function(records) {
    for (var i = 0; i < records.length; i++) {
      if (records[i].unweightedGPA < low) {
        low = records[i].unweightedGPA;
        lowClass = records[i].className;
      }
    }
    var high = 0.00;
    var highClass = "";
    for (var j = 0; j < records.length; j++) {
      if (records[j].unweightedGPA > high) {
        high = records[j].unweightedGPA;
        highClass = records[j].className;
      }
    }
    console.log("Highest Class: " + highClass + "\nUnweighted GPA: " + high);
    console.log("Lowest Class: " + lowClass + "\nUnweighted GPA: " + low);
    setText("lowClassTxt", lowClass);
    setText("highClassTxt", highClass);
    adviceGPA();
    improvement();
  });
}

function improvement () {
  readRecords("classTable", {username: currUser}, function(records) {
    console.log(currUser + "'s Classes: " + records + "\n");
    var diff = [];
    for (var h = 0; h < records.length; h++) {
      var firstGPA;
      var finalGPA;
      var range;
      firstGPA = records[h].Q1;
      console.log("Q1: " + firstGPA);
      if (records[h].Q4 == "") {
        if (records[h].Q3 == "") {
          if (records[h].Q2 == "") {
            finalGPA = records[h].Q1;
            console.log("Final Q: " + finalGPA);
          } else {
            finalGPA = records[h].Q2;
            console.log("Final Q: " + finalGPA);
          }
        } else {
          finalGPA = records[h].Q3;
          console.log("Final Q: " + finalGPA);
        }
      } else {
        finalGPA = records[h].Q4;
        console.log("Final Q: " + finalGPA);
      }
      firstGPA = pointSystem(firstGPA);
      firstGPA = Number(firstGPA);
      firstGPA = round(firstGPA);
      finalGPA = pointSystem(finalGPA);
      finalGPA = Number(finalGPA);
      finalGPA = round(finalGPA);
      range = finalGPA - firstGPA;
      range = round(range);
      console.log("Difference: " + range);
      appendItem(diff, range);
      console.log("Difference Array: " + diff + "\n" + "Class Number: " + diff.length + "\n");
      range = 0;
    }
    
    var mostImprvd;
    for (var k = 0; k < diff.length; k++) {
      if (k == 0) {
        mostImprvd = k;
        console.log(mostImprvd);
      } else {
        if (diff[k] < diff[k-1]) {
          mostImprvd = k - 1;
          console.log(mostImprvd);
        }
      }
    }
    console.log("ID of Most Improved: " + mostImprvd);
    
    var mostImprvdClass;
    readRecords("classTable", {username: currUser}, function(records) {
      mostImprvdClass = records[mostImprvd].className;
      console.log("The most improved class is: " + mostImprvdClass);
      setText("imprvdTxt", mostImprvdClass);
    });
    console.log("All of the Differences: " + diff + "\n");
    diff = [];
  });
}

function adviceGPA () {
  var currGPA = 0;
  var currGrade;
  readRecords("gpaTable", {username: currUser}, function(records) {
    if (records.length === 0) {
      setText("lowClassTxt", "No classes yet!");
      setText("highClassTxt", "No classes yet!");
      setText("adviceTxt", "No classes yet!");
      return;
    } else {
      //console.log(records);
      for (var i = 0; i < records.length; i++) {
        currGPA = Number(records[i].unweighted);
      }
    }
  });
  readRecords("classTable", {username: currUser}, function(records) {
    var points = 0;
    var gpa = 0;
    for (var i = 0; i < records.length; i++) {
      if (records[i].className == lowClass) {
        currGrade = records[i].unweightedGPA;
        currGrade = findLetter(currGrade);
        if (low == 0.00) {
          low = 1.00;
          points += low;
        } else {
          low += 0.30;
          points += low;
        }
      } else {
        points += records[i].unweightedGPA;
      }
    } 
    gpa = points/(records.length);
    gpa = 1 * gpa;
    gpa = round(gpa);
    console.log("Lowest Class's Altered Points: " + low);
    console.log("Lowest Class's Original Grade: " + currGrade);
    console.log("Total Points: " + points);
    console.log("Hypothetical GPA: " + gpa); 
    console.log("Current Unweighted GPA: " + currGPA);
    low = findLetter(low);
    console.log("Must Raise To: " + low);
    //Setting the advice box to
    setText("adviceTxt", "If you raise your cumulative grade in " + lowClass + " from a " + currGrade + " to a/an " + low + ", then your cumulative unweighted GPA will be a " + gpa + ".");
  
    currGPA = 0;
    currGrade = "";
    points = 0;
    gpa = 0;
    low = 4.00;
    lowClass = "";
  });
}

function findLetter (unweightedGPA) {
  var letter = "";
  if (unweightedGPA == 0) {
    letter = "F";
    return letter;
  } else if (unweightedGPA > 0 && unweightedGPA <= 1.00) {
    letter = "D";
    return letter;
  } else if (unweightedGPA > 1.00 && unweightedGPA <= 1.3) {
    letter = "D+";
    return letter;
  } else if (unweightedGPA > 1.3 && unweightedGPA <= 1.70) {
    letter = "C-";
    return letter;
  } else if (unweightedGPA > 1.70 && unweightedGPA <= 2.00) {
    letter = "C";
    return letter;
  } else if (unweightedGPA > 2.00 && unweightedGPA <= 2.30) {
    letter = "C+";
    return letter;
  } else if (unweightedGPA > 2.30 && unweightedGPA <= 2.70) {
    letter = "B-";
    return letter;
  } else if (unweightedGPA > 2.70 && unweightedGPA <= 3.00) {
    letter = "B";
    return letter;
  } else if (unweightedGPA > 3.00 && unweightedGPA <= 3.30) {
    letter = "B+";
    return letter;
  } else if (unweightedGPA > 3.30 && unweightedGPA <= 3.70) {
    letter = "A-";
    return letter;
  } else if (unweightedGPA > 3.70 && unweightedGPA <= 4.00) {
    letter = "A";
    return letter;
  } else {
    letter = "A+";
    return letter;
  }
}

function wrap(val, low, high){
  var output;
  if(val < low){
    output = high;
  } else if (val > high){
    output = low;
  } else {
    output = val;
  }
  return output;
}

function round (input) {
  var n = 1.00 * input;
  console.log("INPUT: " + n);
  n = 100 * n;
  n = Math.round(n);
  n = n/100;
  console.log("FINAL: " + n);
  return n;
}
