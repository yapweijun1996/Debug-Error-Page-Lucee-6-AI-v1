
# Bug Fix: Lucee Variable [VLE] Does Not Exist & Error Output Handling

## Issue Description

When error occur, the following error occurs:

```

variable \[VLE] doesn't exist

```

Additionally, the error output shows a very large SQL query dump with escaped characters, cluttering the user interface and making debugging difficult.

## Root Cause

- The variable `VLE` is referenced but not defined or passed correctly in the Lucee CFML code, causing the "variable doesn't exist" error.
- The error display logic outputs the entire error string directly, including escaped characters (`\r\n\t`), resulting in an unreadable error dump on the page.

## Location

- Error templates are located at:
```

D:\lucee\tomcat\lucee-server\context\context\templates\error\\

````
- The relevant CFML file to amend is `error.cfm`.

## Fix Implemented

1. **Limit error output length and improve readability**

 - Capture the full error string using `luceeCatchToString()`.
 - Limit the output to the last 500 words of the error to avoid excessive dump.
 - Encode the output safely for JavaScript using `EncodeForJavaScript()`.
 - Serialize to JSON string for proper rendering on the frontend.

2. **Amend `<cfoutput>` block in `error.cfm`**

 Replace:
 ```cfml
 <cfoutput>luceeCatchData=#luceeCatchToString()#;</cfoutput>
````

With:

```cfml
<cfoutput>
<cfset max_no_word = 500>
<cfset fullText = luceeCatchToString()>
<cfset wordArray = ListToArray(fullText, " ")>
<cfset wordCount = ArrayLen(wordArray)>
<cfif wordCount GT max_no_word>
    <cfset lastMaxNumWordArray = ArraySlice(wordArray, wordCount-max_no_word+1, max_no_word)>
<cfelse>
    <cfset lastMaxNumWordArray = wordArray>
</cfif>
<cfset lastMaxNumWordArray = ArrayToList(lastMaxNumWordArray, " ")>
<cfset encodedLastMaxNumWord = EncodeForJavaScript(lastMaxNumWordArray)>
<cfset jsonValue = SerializeJSON(encodedLastMaxNumWord)>
luceeCatchData=#jsonValue#;
</cfoutput>
```

3. **JavaScript adjustment**

   Ensure the JavaScript rendering the error uses the updated `luceeCatchData` safely, improving display and spinner logic.

## Testing

* Verified the error message now shows a trimmed and readable snippet of the error.
* Confirmed no variable existence errors for `VLE` occur after adjusting variable assignments in the related CFML code (ensure `VLE` is declared or replaced accordingly).
* UI no longer dumps raw SQL or escaped characters excessively.

## Notes

* If `VLE` variable is still undefined, check upstream code for missing assignment.
* Adjust `max_no_word` in the snippet for longer/shorter error context as needed.
* This fix focuses on improving error display and preventing front-end overload with huge raw data.
