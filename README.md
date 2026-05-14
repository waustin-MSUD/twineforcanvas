# Twine to SCORM  
(now with scoring)  
This guide walks you through dropping your Twine simulation into the prebuilt SCORM template folder so it can report results (either completion or a score) to Canvas. Creating a Twine story isn’t covered here; just the SCORM preparation.  
# What You Need  
- A finished twine story  
- The SCORM template zip file  
- A text editor  
- An app to create zip files  
# Assumptions Made
- I'm only familiar with Sugarcube. The "Step 1" scripts are made for that format. 
- If graded, your simulation uses a variable called $score throughout.  
- The published Twine file is called index.html. All lowercase. No spaces.  
- Any media is “hard coded” in Twine to point to folders called images, videos, and sounds.  
# Process  
## Step 1: Add the SCORM call to the Twine simulation  
In the final passage (or where the learner gets their final score), add one script.  
### For a Scoring Activity  
This can even be attached to a button inside the passage. It gives the SCORM three pieces of information: the actual score earned by the user ($score), the maximum possible score for the activity, and what’s to be considered a passing score for the activity.  
```
<<script>>
    if (typeof window.setScormScore === "function") {
        window.setScormScore($score, 100, 70);
    }
<</script>>
```
### For a Completion Activity  
This can even be attached to a button inside the passage. It tells the SCORM to report “complete” status to the LMS.  
``` 
<<script>>
    if (typeof window.markComplete === "function") {
        window.markComplete();
    }
<</script>>
```
## Step 2: Publish the story  
Save the Twine to a file named index.html.  
## Step 3: Add the SCORM “glue” to the HTML file  
Open the HTML file with your text editor and insert the following code block just before </head>. Be sure to save the file afterward.  
### For a Scoring Activity  
```
<script src="SCORM_API_wrapper.js"></script>

<script>
  // SCORM 2004 glue
  var scorm = pipwerks.SCORM;
  scorm.version = "2004";
  var scormInitialized = false;
 
  function initSCORM() {
    scormInitialized = scorm.init();
    if (!scormInitialized) {
      console.warn("SCORM init failed — running outside an LMS?");
    }
    // Note: the wrapper auto-sets completion_status to "incomplete"
    // on first launch, so no manual call is needed here.
  }
 
  // Call from Twine to report a final score (0 to 100)
  // and mark the attempt complete + passed/failed.
  window.setScormScore = function (rawScore, maxScore, passingScore) {
    if (!scormInitialized) return;
    maxScore = maxScore || 100;
    passingScore = (passingScore === undefined) ? 70 : passingScore;
 
    var scaled = (rawScore / maxScore);          // -1.0 to 1.0
    if (scaled > 1) scaled = 1;
    if (scaled < -1) scaled = -1;
 
    scorm.set("cmi.score.raw", rawScore);
    scorm.set("cmi.score.min", 0);
    scorm.set("cmi.score.max", maxScore);
    scorm.set("cmi.score.scaled", scaled);
    scorm.set("cmi.completion_status", "completed");
    scorm.set("cmi.success_status",
              rawScore >= passingScore ? "passed" : "failed");
    scorm.save();
  };
 
  // Call when the user finishes (or when the window closes)
  function terminateSCORM() {
    if (!scormInitialized) return;
    scorm.save();
    scorm.quit();
    scormInitialized = false;
  }
 
  // Run as soon as the page is ready
  if (document.readyState === "loading") {
    document.addEventListener("DOMContentLoaded", initSCORM);
  } else {
    initSCORM();
  }
 
  // posts the score to Canvas
  window.addEventListener("beforeunload", terminateSCORM);
  window.addEventListener("pagehide", terminateSCORM);
</script>

``` 
### For a Completion Activity  
```
<script>
  // SCORM 2004 glue (completion)
  var scorm = pipwerks.SCORM;
  scorm.version = "2004";
  var scormInitialized = false;
 
  function initSCORM() {
    scormInitialized = scorm.init();
    if (!scormInitialized) {
      console.warn("SCORM init failed — running outside an LMS?");
    }
    // The wrapper auto-sets completion_status to "incomplete"
    // on first launch, so no manual call is needed here.
  }
 
  // Call from Twine when the learner has finished the activity
  window.markComplete = function () {
    if (!scormInitialized) return;
    scorm.set("cmi.completion_status", "completed");
    scorm.save();
  };
 
  // Call when the user finishes (or when the window closes)
  function terminateSCORM() {
    if (!scormInitialized) return;
    scorm.save();
    scorm.quit();
    scormInitialized = false;
  }
 
  // Run init as soon as the page is ready
  if (document.readyState === "loading") {
    document.addEventListener("DOMContentLoaded", initSCORM);
  } else {
    initSCORM();
  }
 
  // Always terminate on unload — this is what commits the status
  window.addEventListener("beforeunload", terminateSCORM);
  window.addEventListener("pagehide", terminateSCORM);
</script>
```   
## Step 4: Add Files  
Drop your HTML file into the template folder. Any media should be dropped into the appropriate media folder (images, videos, sounds).  
## Step 5: Edit the Manifest File  
- Open the imsmanifest.xml file in your text editor.  
- Edit the course title and activity title lines.  
- You can optionally add lines following the template for each file in your SCORM, including media. This is required for a “compliant” SCORM file, but Canvas doesn’t care.  
- Save the file.  
```
<?xml version="1.0" encoding="UTF-8"?>
<manifest identifier="MANIFEST-twine-001"
          version="1.0"
          xmlns="http://www.imsglobal.org/xsd/imscp_v1p1"
          xmlns:adlcp="http://www.adlnet.org/xsd/adlcp_v1p3"
          xmlns:adlseq="http://www.adlnet.org/xsd/adlseq_v1p3"
          xmlns:adlnav="http://www.adlnet.org/xsd/adlnav_v1p3"
          xmlns:imsss="http://www.imsglobal.org/xsd/imsss">
 
  <metadata>
    <schema>ADL SCORM</schema>
    <schemaversion>2004 4th Edition</schemaversion>
  </metadata>
 
  <organizations default="ORG-1">
    <organization identifier="ORG-1">
      <title>EDIT ME — Course Title</title>        
      <item identifier="ITEM-1" identifierref="RES-1">
        <title>EDIT ME — Activity Title</title>      
        <adlcp:timeLimitAction>continue,no message</adlcp:timeLimitAction>
        <imsss:sequencing>
          <imsss:deliveryControls completionSetByContent="true"/>
        </imsss:sequencing>
      </item>
      <imsss:sequencing>
        <imsss:controlMode choice="true" flow="true"/>
      </imsss:sequencing>
    </organization>
  </organizations>
 
  <resources>
    <resource identifier="RES-1"
              type="webcontent"
              adlcp:scormType="sco"
              href="index.html">                     
      <file href="index.html"/>
      <file href="SCORM_API_wrapper.js"/>           
      <!-- Add a <file href="..."/> line for every other asset -->
      <!-- Add a <file href="images/..."/> line for every image -->
      <!-- Add a <file href="videos/..."/> line for every video -->
      <!-- Add a <file href="sounds/..."/> line for every sound file -->
    </resource>
  </resources>
</manifest>
   
```
## Step 6: Zip the files  
- Inside the scorm-template folder, select all items and create a zip file.  
- Be sure to just create a zip of the contents, not the containing folder.  
- Now you have a SCORM file you can upload to Canvas.  
## Optional: Canvas Configuration  
For scoring activities, the assignment settings should display grade as points. Completion activities should be complete/incomplete. Ungraded assignments, of course, should be set to the “Not Graded” submission type.  
   
