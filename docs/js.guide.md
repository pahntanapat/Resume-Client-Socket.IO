# Guide for Javascript and Node.js Client

- [Guide for Javascript and Node.js Client](#guide-for-javascript-and-nodejs-client)
  - [**_Important!_** Provide JavaScript for HTML <a name="prepare"></a>](#important-provide-javascript-for-html-)
    - [Directly copy &amp; paste to your `./js/` path](#directly-copy--paste-to-your-js-path)
      - [Location of these files](#location-of-these-files)
      - [Resume CDN Files](#resume-cdn-files)
    - [Include from Resume CDN](#include-from-resume-cdn)
    - [Router your HTTP server to JS path or files](#router-your-http-server-to-js-path-or-files)
  - [(Optional) Write essential callbacks  <a name="conv-call"></a>](#optional-write-essential-callbacks--)
    - [getIntermediateUserTranscript Function](#getintermediateusertranscript-function)
    - [onReceiveTranscript Callback](#onreceivetranscript-callback)
  - [Create `Socket.IO` client](#create-socketio-client)
  - [Construct `ResumeOne` object](#construct-resumeone-object)
    - [Example for Conversation Mode](#example-for-conversation-mode)
    - [Example for Dictation Mode](#example-for-dictation-mode)
    - [Example for Combination Mode](#example-for-combination-mode)
  - [Assign user input form to `ResumeOne.tag` before start recoring](#assign-user-input-form-to-resumeonetag-before-start-recoring)
    - [Conversation Mode](#conversation-mode)
    - [Dictation Mode](#dictation-mode)
    - [Combination Mode](#combination-mode)
      - [Start with Conversion transcription](#start-with-conversion-transcription)
    - [Start with Dictation transcription](#start-with-dictation-transcription)
  - [Start new session for recording](#start-new-session-for-recording)
  - [Pause and Resume](#pause-and-resume)
  - [Switch to other Form input (Form tag) in *Dictation Mode* and *Combination Mode*](#switch-to-other-form-input-form-tag-in-dictation-mode-and-combination-mode)
  - [Switch to Conversation transcription in *Combination Mode*](#switch-to-conversation-transcription-in-combination-mode)
  - [Update the Result](#update-the-result)
    - [Stop recording and End session <a name="end-sess"></a>](#stop-recording-and-end-session-)
  - [Differences between Resume Transcription Mode](#differences-between-resume-transcription-mode)
  - [More Information](#more-information)
   

## **_Important!_** Provide JavaScript for HTML <a name="prepare"></a>


`ResumeOne` requires [RecordRTC](https://github.com/muaz-khan/RecordRTC) to control microphone, [Socket.IO](https://socket.io/) to connect with server, and [jQuery](https://jquery.com/) if you want to use [`ResumeOne.loadSectionList()`](public/js/Resume.js) method.  

### Directly copy &amp; paste to your `./js/` path

#### Location of these files
- [NPM package](https://www.npmjs.com/package/resume-client-socket.io): `node_modules/resume-client-socket.io/public/`
- [GitHub](https://github.com/pahntanapat/Resume-Client-Socket.IO): [https://github.com/pahntanapat/Resume-Client-Socket.IO/tree/main/public/js](https://github.com/pahntanapat/Resume-Client-Socket.IO/tree/main/public/js)
- [Resume CDN](#resume-cdn-files): direct download file from CDN and copy to your directory.

#### Resume CDN Files
- [adapter.js](https://cdn.sati.co.th/resume-client-socketio/adapter.js)
- [jquery.min.js](https://cdn.sati.co.th/resume-client-socketio/jquery.min.js)
- [RecordRTC.min.js](https://cdn.sati.co.th/resume-client-socketio/RecordRTC.min.js)
- [Resume.js](https://cdn.sati.co.th/resume-client-socketio/Resume.js)

You can provide them by copy &amp; paste to your front-end HTML.

```HTML
<head>
    <!-- ... -->

    <!-- Socket.IO Client -->
    <script src="/socket.io/socket.io.js"></script>
    <!-- Resume and jQuery -->
    <script src="./js/jquery.min.js"></script>
    <script src="./js/Resume.js"></script>

    <!-- ... -->
</head>
<body>
    <!-- ... -->

    <!-- RecordRTC -->
    <script src="./js/RecordRTC.min.js"></script>
    <!-- ... -->
</body>
```

### Include from Resume CDN 

***Note:*** NOT recommend for local network use, if you are not sure that your clients are able to reach external HTTP(S) files.
  
```HTML
<head>
    <!-- ... -->

    <!-- Socket.IO Client -->
    <script src="/socket.io/socket.io.js"></script>
    <!-- You can use jQuery CDN. (https://code.jquery.com/) -->
    <script src="//cdn.sati.co.th/resume-client-socketio/jquery.min.js"></script>
    <!-- Resume -->
    <script src="//cdn.sati.co.th/resume-client-socketio/Resume.js"></script>

    <!-- ... -->
</head>
<body>
    <!-- ... -->

    <!-- RecordRTC -->
    <script src="//cdn.sati.co.th/resume-client-socketio/RecordRTC.min.js"></script>
    <!-- ... -->
</body>
```

### Router your HTTP server to JS path or files

The `StaticJSDir()` returns directory path of them. And the `StaticJSFiles()` gives array of JavaScript files.  
You can set your server object to these paths.  

Example server in [Express](https://expressjs.com/)
```JS
const express = require('express');
const app = express();
const { StaticJSDir } = require('resume-client-socket.io');
/* ... */

// Route the Resume Client Scripts' directory to /resume path
app.use('/resume', express.static(StaticJSDir()));

```

And include into your front-end HTML.
```HTML
<head>
    <!-- ... -->

    <!-- Socket.IO Client -->
    <script src="/socket.io/socket.io.js"></script>
    <!-- Resume and jQuery -->
    <script src="/resume/jquery.min.js"></script>
    <script src="/resume/Resume.js"></script>

    <!-- ... -->
</head>
<body>
    <!-- ... -->

    <!-- RecordRTC -->
    <script src="/resume/RecordRTC.min.js"></script>
    <!-- ... -->
</body>
```
  <br>


## (Optional) Write essential callbacks  <a name="conv-call"></a>
We recommend you to assign function for [`getIntermediateUserTranscript`](Resume.js.md#getIntermediateUserTranscript) and [`onReceiveTranscript`](Resume.js.md#onReceiveTranscript) callback.  

### getIntermediateUserTranscript Function
Resume.js will automatically calls [`getIntermediateUserTranscript`](Resume.js.md#getIntermediateUserTranscript) when requests for user input to send to Resume API with sound chunk.

The properties of object returned from [`getIntermediateUserTranscript`](Resume.js.md#getIntermediateUserTranscript) should follow [the Property of "C-CDA 1.1.0 on FHIR resource profile"](CCDA.md). 

```JS
/* Function */
function _getUserTranscribe () {
    /* this format follows C-CDA. */
    return {
        chief_complaint_section: this.CC,
        history_of_present_illness_section: this.PI,
        past_medical_history_section: this.PHx,
        problem_section: this.Dx,
        follow_up_section: this.FU
    };
}
```
- The [`getIntermediateUserTranscript`](Resume.js.md#getIntermediateUserTranscript) callback
- The [`GroupTxt`](https://github.com/pahntanapat/Resume-Node-REST-Connector/blob/main/docs/Resume-REST-API-Connect.md#module_Resume-REST-API-Connect..GroupText), return object from [`getIntermediateUserTranscript`](Resume.js.md#getIntermediateUserTranscript)

### onReceiveTranscript Callback
The [`onReceiveTranscript`](Resume.js.md#onReceiveTranscript) callback will be triggered when result arrives. <br>  
The [`MlGroupTxt`](https://github.com/pahntanapat/Resume-Node-REST-Connector/blob/main/docs/Resume-REST-API-Connect.md#module_Resume-REST-API-Connect..GroupText) and [`TagRawTxt`](https://github.com/pahntanapat/Resume-Node-REST-Connector/blob/main/docs/Resume-REST-API-Connect.md#module_Resume-REST-API-Connect..GroupText) in [transcript](Resume.js.md#Transcript) argument contains properties that follow [Terminologies of "C-CDA 1.1.0 on FHIR resource profile"](CCDA.md).
 
Their values are string of sentence or pharse (`["This is sentence one.","Or pharse","It can be pharse or sentence.","..."]`). please read [[Resume-Node-REST-Connector](https://github.com/pahntanapat/Resume-Node-REST-Connector) for more details.

```JS
function _onReceiveTranscript (transcript, isEnd) {
    if (transcript.MlGroupTxt) {
        /* If received Transcription result */
        
        let PI = transcript.MlGroupTxt.history_of_present_illness_section;
        /* The response will have keys like the Terminology of "C-CDA 1.1.0 on FHIR resource profile". */
        /* http://hl7.org/fhir/us/ccda/artifacts.html#structures-resource-profiles */

        /* Others */
        let other = transcript.TagRawTxt.other;
        /* ... */
    }
    

    if (transcript.TagRawTxt) {
        //////////////////////////////////////////////////////////////////////////////////////////////
        /// Note: Resume API will respond transcript.TagRawTxt only Dictation and Combination Mode ///
        //////////////////////////////////////////////////////////////////////////////////////////////
        
        
        tCC = transcript.TagRawTxt.chief_complaint_section;
        tPI = transcript.TagRawTxt.history_of_present_illness_section;
        /* The response will have keys like the Terminology of "C-CDA 1.1.0 on FHIR resource profile". */
        
        /* ... */
    }


    if (isEnd) {
        /* Action when API stop session completely. */
    }
}
```
  
 - ["C-CDA 1.1.0 on FHIR resource profile"](CCDA.md)
 - [README.md](../README.md#freq-doc)
 - [GroupText](https://github.com/pahntanapat/Resume-Node-REST-Connector/blob/main/docs/Resume-REST-API-Connect.md#module_Resume-REST-API-Connect..GroupText)
 - [transcript](Resume.js.md#Transcript)

<br>

## Create `Socket.IO` client

```JS
var socket = io();
```
  
If you use Socket.IO server from different Host or Path, Please follow [Socket.IO Client document](https://socket.io/docs/v4/client-initialization/#From-a-different-domain).

```JS
var socket = io("your-another-host-for-socket.io");
```

## Construct `ResumeOne` object

Assign `getIntermediateUserTranscript`, `onReceiveTranscript`, and `multiSpeaker` to [`resumeOption`](Resume.js.md#new-resumeonesocket-resumeoption).
```JS
var resumeOption = {
    getIntermediateUserTranscript: _getUserTranscribe,
    onReceiveTranscript: _onReceiveTranscript,
    multiSpeaker: /* depend on mode */
};

var resume = new ResumeOne(
    socket, /* socket variable from above*/
    resumeOption
);
```
The [`resumeOption`](Resume.js.md#new-resumeonesocket-resumeoption) argument should follow [`RESUME_DEFAULT_OPTION`](Resume.js.md#RESUME_DEFAULT_OPTION) object.

### Example for Conversation Mode

Set [`resumeOption.multiSpeaker = true`](Resume.js.md#new-resumeonesocket-resumeoption)
```JS
var resume = new ResumeOne(socket, 
    {
        getIntermediateUserTranscript: _getUserTranscribe,
        onReceiveTranscript: _onReceiveTranscript,
        multiSpeaker: true   // true = conversation Mode
    });
```
  <br>


### Example for Dictation Mode

Set [`resumeOption.multiSpeaker = false`](Resume.js.md#new-resumeonesocket-resumeoption)
```JS
var resume = new ResumeOne(socket, 
    {
        getIntermediateUserTranscript: _getUserTranscribe,
        onReceiveTranscript: _onReceiveTranscript,
        multiSpeaker: false   // true = conversation Mode
    });
```
  <br>



### Example for Combination Mode

Set [`resumeOption.multiSpeaker = true`](Resume.js.md#new-resumeonesocket-resumeoption)
```JS
var resume = new ResumeOne(socket, 
    {
        getIntermediateUserTranscript: _getUserTranscribe,
        onReceiveTranscript: _onReceiveTranscript,
        multiSpeaker: true   // true = combination Mode
    });
```
  <br>


## Assign user input form to `ResumeOne.tag` before start recoring
In Dictation Mode and Combination Mode.  <br>
For good practice, [`ResumeOne.tag`](Resume.js.md#ResumeChild) should follow [section (property) name of C-CDA 1.1.0](CCDA.md).

### Conversation Mode
*Not have this function*

### Dictation Mode
```JS
resume.tag = "chief_complaint_section";
```

### Combination Mode

#### Start with Conversion transcription
Set to `undefined` or `null`.
```JS
resume.tag = null;
```

### Start with Dictation transcription
Same to [Dictation Mode](#dictation-mode)
```JS
resume.tag = "history_of_present_illness_section";
```

## Start new session for recording
When user trigers start event, the [`ResumeOne.newSession`](Resume.js.md#resumeonenewsessionhint-identifier-sectionid-docformat-langsuggest) will obtain new Session ID from `Resume API`, concurrently initiates [microphone recorder](Resume.js.md#ResumeRecorder).

Provided identifier will pass to Resume server-sided callbacks - [sessionSIOOnConnection and onNewTranscriptSessionSyncCheck](#module_Resume-Socket-IO-Server..OptionSIO). Resume.js doesn&apos;t send it to Resume API.

```JS
/* Patient and Healthcare worker identifier for sending to local logging server, Not send to Resume API publically. */
let identifier = { HN: HN, TXN: TXN, Practioner: Practioner, Location:Location };

/* Document format  must follows C-CDA Document name. Please see README.md and CCDA.md documentation. */
let sectionFormat = "HistoryAndPhysical";

/* Section to logging on Resume usage DB */
let sectionID = "OPD1";

/* request for new session */
resume.newSession(Hint,
    identifier,
    sectionID,
    sectionFormat  
);
```
[ResumeOne.newSession method](Resume.js.md#ResumeOne+newSession)

  <br>

## Pause and Resume

```JS
// to pause
resume.pause();

// to resume
resume.resume();
```

  <br>

## Switch to other Form input (Form tag) in *Dictation Mode* and *Combination Mode*
To switch to Form input (in *Dictation Mode* and *Combination Mode*), or switch from Conversation transcription to Dictation transcription in *Combination Mode*.  <br>

**Please follow these steps**
1. Pause the recording
2. Assign name of form input to [`ResumeOne.tag`](Resume.js.md#ResumeChild)
3. Resume record

For good practice, [`ResumeOne.tag`](Resume.js.md#ResumeChild) should follow [section (property) name of C-CDA 1.1.0](CCDA.md).

```JS
// 1. pause
resume.pause();

// 2. set form tag
resume.tag = "medications_section";

// 3. resume
resume.resume();
```

Please pause before change the [`ResumeOne.tag`](Resume.js.md#ResumeChild) and then resume the session, in order to prevent unreliable [transcription result](https://github.com/pahntanapat/Resume-Node-REST-Connector/blob/main/docs/Resume-REST-API-Connect.md#module_Resume-REST-API-Connect..Transcript) during changing period.  

## Switch to Conversation transcription in *Combination Mode*

**Please follow these steps**
1. Pause the recording
2. Set [`ResumeOne.tag`](Resume.js.md#ResumeChild) to `null` or `undefined`
3. Resume record

For good practice, [`ResumeOne.tag`](Resume.js.md#ResumeChild) should follow [section (property) name of C-CDA 1.1.0](CCDA.md).

```JS
// 1. pause
resume.pause();

// 2. set form tag
resume.tag = null;

// 3. resume
resume.resume();
```

Please pause before change the [`ResumeOne.tag`](Resume.js.md#ResumeChild) and then resume the session, in order to prevent unreliable [transcription result](https://github.com/pahntanapat/Resume-Node-REST-Connector/blob/main/docs/Resume-REST-API-Connect.md#module_Resume-REST-API-Connect..Transcript) during changing period.  
  <br>


## Update the Result
The [`ResumeOne`](Resume.js.md#ResumeOne) object automatically updates the `Resume API` response to client. It will call the [onReceiveTranscript Callback](#onreceivetranscript-callback). It also stores result in [`transcript` property](Resume.js.md#ResumeChild).

```JS
/* Normally, ResumeOne always automatically calls onReceiveTranscript when recieved Resume API response. */
/* If you prefer to get result from property, */
let response = resume.transcript;
```
  <br>

### Stop recording and End session <a name="end-sess"></a>
***Warning!*** for better `Resume API` accuracy, please end session only if change the patients or close the page.

```JS
let userForm = _getUserTranscribe(); /* Get user form data */

resume.endSession(userForm);
```

The argument of `endSession` can be omitted, if you provide [getIntermediateUserTranscript Function](#getintermediateusertranscript-function).
```JS
resume.endSession();
```
  
-------  
<br/>  
  

## Differences between Resume Transcription Mode

| Part | Conversation Mode | Dictation Mode | Combination Mode |
| ---- | ---- | ---- | ---- |
| Prepare HTML page for script | [All are same.](#prepare) | [All are same.](#prepare) | [All are same.](#prepare) |
| [`transcript` ](Resume.js.md#Transcript) result | contains [`MlGroupTxt`](https://github.com/pahntanapat/Resume-Node-REST-Connector/blob/main/docs/Resume-REST-API-Connect.md#module_Resume-REST-API-Connect..GroupText) property - [details](#conv-call) | both [`MlGroupTxt`](https://github.com/pahntanapat/Resume-Node-REST-Connector/blob/main/docs/Resume-REST-API-Connect.md#module_Resume-REST-API-Connect..GroupText) and [`TagRawTxt`](https://github.com/pahntanapat/Resume-Node-REST-Connector/blob/main/docs/Resume-REST-API-Connect.md#module_Resume-REST-API-Connect..GroupText) - [details](#dict-call) | same to [Dictation Mode](#dict-call) |
| [`resumeOption.multiSpeaker`](Resume.js.md#new-resumeonesocket-resumeoption) argument in [`ResumeOne`](Resume.js.md#ResumeOne) constructor | [`true`](#conv-create) | ***[`false`](#dict-create)*** | [`true`](#conv-create) |
| Need to set [`tag` property](Resume.js.md#ResumeChild) | No | Yes: [before start](#dict-start) and [change input form](#dict-control) | Yes: If you need [Dictation Mode](#dict-start), set to `null` if you switch back to [Conversation mode](#conversation-mode) |
| Ending session | Only if **[change patients or close windows](#end-sess)** | same to [Conversation Mode](#end-sess) | same to [Conversation Mode](#end-sess) |

  
  <br/>



## More Information
 - [Read me file](../README.md)
 - [Quick Start Guide for Server-sided Node.JS](node.guide.md)
 - [Quick Start Guide for Client-sided Resume.js](js.guide.md)
 - [Resume Socket.IO Server](Resume-Socket-IO-Server.md)
 - [Client-sided Resume.js](Resume.js.md)

&copy; 2021 - copyright by Tanapat Kahabodeekanokkul - the founder of [Resume](https://sati.co.th).

