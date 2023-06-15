```js
//Function to update the URL without adding a new entry to the browser 
  function updateURLWithoutSubdirectory() {
    //gets URL of website
    var url = window.location.href;

    //finds the index of where the goal directory is 
    var subdirectoryIndex = url.indexOf('/e8d4c4d6bdaef0b4ead50d1196c3fbda8323f1f1/');

    // -1 state only happens if the directory is not found
    if (subdirectoryIndex !== -1) {

      // NewURL = 0 -> where subdirectoryIndex is.
      var newURL = url.substr(0, subdirectoryIndex);

        // pushes the new url to the topbar without forcing a refresh
      history.pushState({}, document.title, newURL);
    }
  }

  // Call the function to update the URL when the page loads
  window.addEventListener('DOMContentLoaded', updateURLWithoutSubdirectory);
```
