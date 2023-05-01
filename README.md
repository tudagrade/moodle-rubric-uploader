# Moodle Rubric Uploader
When [Java AutoGrader](https://github.com/sourcegrade/jagr) is used to grade submissions, it produces a collection of grading results that contains the total points and feedback for each student.
These grading results are called "Rubrics" and saved as JSON files.

Moodle Rubric Uploader can be used to import rubrics into Moodle so that students can view their results and feedback in the assignment activity next to their submission.

## Setup
**Moodle Rubric Uploader requires a one-time setup for your Moodle course in order to work properly.**

### 1. Find your Moodle Course ID
1. Open a web browser and go to the overview page of your Moodle course.
1. Find the Moodle course ID of your course. This is the value of the `id` parameter of the URL. For example, if the URL of your course is `https://moodle.informatik.tu-darmstadt.de/course/view.php?id=1234`, then your Moodle course ID is `1234`.
1. Clone this repository and open the file `rubric-uploader.html` in a text editor.
1. Look for the definition of the constant `MOODLE_COURSE_ID` and update it with your Moodle course ID. For example:
    ```js
    const MOODLE_COURSE_ID = 1234;
    ```

### 2. Create a Mapping Student ID (TU-ID) -> Moodle ID
During the upload process Moodle Rubric Uploader needs to translate student IDs (TU-IDs) into Moodle user IDs. This requires a specially crafted Moodle report, which can be created as follows:
1. Open a web browser and go to `https://<MOODLE_URL>/blocks/configurable_reports/managereport.php?courseid=<COURSE_ID>`, where `<MOODLE_URL>` is the base URL of your Moodle instance and `<COURSE_ID>` is the Moodle course ID you identified in the previous section. For example, the URL for course 1234 in Lernportal Informatik is `https://moodle.informatik.tu-darmstadt.de/blocks/configurable_reports/managereport.php?courseid=1234`.
1. Click "Add report".
1. Type a name and optionally a summary. The values that you enter here do not have to follow a specific pattern, but you may want to choose a descriptive name which helps you and your colleagues recognize the report and its purpose later.
1. Set "Type of report" to "Users report".
1. Make sure that "Pagination" is set to 0 (denoting no pagination), which should be the default.
1. Click "Add".
1. You should be redirected to the settings of your report with the "Columns" tab selected. We will now add two columns to the report - one for the TU ID (username) and one for the Moodle user ID (id). **The display names of these columns do not matter, but their order does!** Hence, please make sure to add the column for the username first and then the Moodle user ID as a second column, as described in the following two steps.
1. Next to "Add", click "Choose" and select "User profile field", which causes the page to reload. Once this is completed, set "Column" to `username` and enter a descriptive name, such as "tu_id". The name that you choose here does not matter, as explained above. Click "Add" to add the column.
1. Next to "Add", click "Choose" and select "User profile field", which causes the page to reload. Once this is completed, set "Column" to `id` and enter a descriptive name, such as "moodle_user_id". The name that you choose here does not matter, as explained above. Click "Add" to add the column.
1. Switch to the "Conditions" tab.
1. Next to "Add", click "Choose" and select "Users in current report course", which causes the page to reload. Once this is completed, select `student` under "Roles" and click "Add".
    
    **Note: This condition is important. If you do not add it to your report, Moodle may fail with an out-of-memory error.**
1. Switch to the "View report" tab and verify that your report is displayed correctly. You should see a data table with the two columns you just set up.
1. Determine the Moodle report ID of your report. This is the value of the `id` parameter of the URL (NOT the `courseid` parameter). For example, if your URL is `https://moodle.informatik.tu-darmstadt.de/blocks/configurable_reports/viewreport.php?id=203&courseid=1234`, then the Moodle report ID is `203`.
1. Go back to your text editor where `rubric-uploader.html` is opened.
1. Look for the definition of the constant `MOODLE_USER_ID_REPORT_ID` and update it with the Moodle report ID that you just determined. For example:
    ```js
    const MOODLE_USER_ID_REPORT_ID = 203;
    ```
1. Save the updated file `rubric-uploader.html`.

### 3. Upload the Rubric Uploader to Moodle
1. Open a web browser and go to the overview page of your Moodle course.
1. Turn editing on for your course.
1. Click "Add an activity or resource" in a section of your choice.
1. Select "File" and give it a descriptive name, e.g. "Moodle Rubric Uploader".
1. Drag and drop the file `rubric-uploader.html` that you edited in the previous sections into the drop area next to "Select files".
1. Restrict access such that students and tutors cannot view the file. Alternatively, you can set the visibility of this file to "hidden".

    (This is just to reduce visual clutter for your students and tutors. Even if students could access the file, security would not be impacted. This is because our script makes ordinary AJAX calls to the same grading endpoints that Moodle's official grading UI uses. Since Moodle implements access control for these endpoints, if students were to use the Rubric Uploader, they could interact with the UI but would get "Access denied" errors as they are not authorized to modify grades.)
1. Click "Save and display" and verify that the Moodle Rubric Uploader loads correctly.

## Usage
1. Open your Moodle course in a **Chromium-based** web browser and go to the assignment activity for which you want to upload the rubrics, e.g. `https://moodle.informatik.tu-darmstadt.de/mod/assign/view.php?id=54321`.
1. Copy the full URL of this page.
1. Select "View all submissions", go to the very bottom of the page and make sure that no filters are selected.
1. Open the Rubric Uploader and paste the URL into the field "Moodle Assign URL".
1. Choose whether you want want students to receive email notifications about the uploaded grades.
1. Drag and drop the rubric files you want to upload into the grey area on the page. You can drop one or multiple individual JSON rubric files here or a directory that contains the JSON files. If you drop a directory, it is recursively enumerated and all JSON files are extracted.
1. Check that the number of detected rubrics is correct.
1. Click "Upload Rubrics" to initiate the upload. Depending on the number of rubrics, this may take several minutes.
1. Once the upload is finished, you should see a success message or a list of errors. If there are errors, the JavaScript console in your browser's dev tools may provide additional information.
