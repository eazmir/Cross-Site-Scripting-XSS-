
# Lab : DOM XSS in innerHTML Sink Using source location.search

**Lab URL**: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-innerhtml-sink
## Overview
This lab demonstrates a DOM-based Cross-Site Scripting (XSS) vulnerability found in the search blog functionality. The vulnerability occurs when user input from `location.search` is directly inserted into the DOM using `innerHTML` without proper sanitization.

## Vulnerability Description
The application uses an `innerHTML` assignment to change the HTML contents of a div element, using data directly from `location.search`. This creates a DOM-based XSS vulnerability because:

1. User input is taken from the URL search parameters
2. The input is directly inserted into the DOM using `innerHTML`
3. No input validation or sanitization is performed
4. Malicious scripts can be executed in the user's browser

## Objective
Perform a cross-site scripting attack that calls the `alert()` function to demonstrate the vulnerability.

## Payload
```html
<img src=x onerror='alert("XSS")'>
```

### Why This Payload Works
- The browser blocks `<script>` tags when inserted using `innerHTML`
- The `<img>` tag with an invalid `src` attribute triggers the `onerror` event
- The `onerror` event handler executes the JavaScript code `alert("XSS")`
- This bypasses the `<script>` tag restriction while still executing JavaScript

## Vulnerable Code

### HTML with JavaScript
```html
<script>
    function doSearchQuery(query) {
        document.getElementById('searchMessage').innerHTML = query;
    }
    
    var query = (new URLSearchParams(window.location.search)).get('search');
    if(query) {
        doSearchQuery(query);
    }
</script>
```

### Pure JavaScript Version
```javascript
function doSearchQuery(query) {
    document.getElementById('searchMessage').innerHTML = query;
}

var query = (new URLSearchParams(window.location.search)).get('search');
if(query) {
    doSearchQuery(query);
}
```

### How the Attack Works
1. User visits: `vulnerable-site.com/search?search=<img src=x onerror='alert("XSS")'>`
2. JavaScript extracts the `search` parameter from the URL
3. The malicious payload is passed to `doSearchQuery()`
4. `innerHTML` renders the payload as HTML, executing the JavaScript

## Protection Methods

### Primary Fix
Replace the vulnerable line:
```javascript
document.getElementById('searchMessage').innerHTML = query;
```

With the secure alternative:
```javascript
document.getElementById('searchMessage').textContent = query;
```

### Why This Fix Works
- **`innerHTML`**: Renders input as HTML/JavaScript, allowing script execution
- **`textContent`**: Displays input as plain text only, preventing script execution
- Even malicious payloads like `<img src=x onerror='alert("XSS")'>` will display as literal text

### Additional Protection Measures
1. **Input Validation**: Validate and sanitize all user inputs
2. **Content Security Policy (CSP)**: Implement CSP headers to prevent inline script execution
3. **HTML Encoding**: Encode special characters before inserting into DOM
4. **Use Safe DOM Methods**: Prefer `textContent`, `innerText`, or `createTextNode()`

## Key Learning Points
- DOM-based XSS occurs entirely in the client-side code
- `innerHTML` is dangerous when used with untrusted input
- Modern browsers block `<script>` tags in `innerHTML`, but other vectors exist
- Always sanitize user input before inserting into the DOM
- Use safe DOM manipulation methods when possible

## Lab Category
**Platform**: PortSwigger Web Security Academy  
**Vulnerability Type**: DOM-based Cross-Site Scripting (XSS)  
**Difficulty**: Beginner
