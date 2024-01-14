---
tags: ["code_snippet", "rails", "stimulus"]
date: 2023-12-31
description: The code snippet to add functionality copy text to clipboard using stimulus rails
published: true
category: code_snippet
id: 1c7ad89c-6e69-4ea8-b1e6-1e937a008942
title: "How to Create a Copy-to-Clipboard Action in Stimulus"
created_time: 2023-12-31T16:28:00+00:00
cover: 
icon: 
last_edited_time: 2024-01-14T15:35:00+00:00
archived: false
---

Hello, fellow coders! Today, we're going to explore a handy code snippet that allows you to copy content to the clipboard using Stimulus.

## **Pre-requisites**

To get the most out of this snippet, you should be familiar with JavaScript and have a basic understanding of the Stimulus framework.

## Code Snippet

Create a stimulus controller

```javascript
// app/javascript/controllers/clipboard_controller.js
import { Controller } from "stimulus";

export default class extends Controller {
  copyFromText(event) {
		event.preventDefault()
    const textToCopy = event.target.innerText

    navigator.clipboard.writeText(textToCopy).then(() => {
      // optional, you also can add toasr message
			alert('Copied to clipboard')
    }).catch(err => {
      console.error('Could not copy text: ', err)
    });
  }
}
```

This is a Stimulus controller with a single `copy` action. When this action is invoked (for example, by clicking a button), it prevents the default action from occurring with `event.preventDefault()`. It then uses the `navigator.clipboard.writeText` function to copy the text from the `data-clipboard-text` attribute of the target element to the clipboard.

### Usage

To use this controller in your Stimulus application, you would first add a `data-controller` attribute to the element that should trigger the copy action. You would also add a `data-action` attribute to specify that the `copy` action should be invoked on click.

```html
<div data-controller="clipboard">
  <p data-action="click->clipboard#copyFromText">
    Text to copy
	</p>
</div>
```

## **Conclusion**

And there you have it! With this simple Stimulus controller, you can easily add copy-to-clipboard functionality to your web application. Try it out and see how it can enhance your user experience.

Did you find this snippet useful? Do you have any questions or suggestions for improvement? Let us know in the comments below. And stay tuned for more code snippets coming your way!

That's it! You should now have a simple copy-to-clipboard feature in your Rails application using Stimulus. Adjust the code and styles as needed for your specific use case.
