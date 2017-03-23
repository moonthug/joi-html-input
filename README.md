# joi-html-input

A [Joi](https://www.npmjs.com/package/joi) extension for sanitizing and validating html inputs.


## Installation

Installation using yarn:

If you're not familiar with yarn it's a faster more reliable drop-in replacement for npm [read more here](https://yarnpkg.com/)
```
$ yarn add joi-html-input
```

Installation using npm
```
$ npm install --save joi-html-input
```

## Sanitization

To remove unwanted tags from the user input you can use `.allowedTags()` to pass the input through [sanitize-html](https://www.npmjs.com/package/sanitize-html). By default this will strip things like `<script>` and `<iframe>` tags but leave in most other common tags.

```
const htmlInput = require('joi-html-input');
const Joi = require('joi').extend(htmlInput);

const htmlString = '<div>Test<script>alert(\'test\');</script></div>';
const joiSchema = Joi.htmlInput().allowedTags();
const results = Joi.validate(htmlString, joiSchema);

console.log(results);

/* Expected output:
{ error: null, value: '<div>Test</div>' }
*/

```

If you want more control over what html tags and attributes are allowed you can pass an options object to `.allowedTags()` which will be passed directly to [sanitize-html](https://www.npmjs.com/package/sanitize-html) so see thier documentation for details but here is an example.

```
const htmlInput = require('joi-html-input');
const Joi = require('joi').extend(htmlInput);

const sanitizeConfig = {
  'allowedTags': [
    'h1',
    'span'
  ],
  'allowedAttributes': {
    'span': [
      'style'
    ]
  }
};

const htmlString = '<h1><span class="align-left" style="color: red;">Test Link</span></h1>';

const joiSchema = Joi.htmlInput().allowedTags(sanitizeConfig);
const results = Joi.validate(htmlString, joiSchema);

console.log(results);

/* Expected output:
{ error: null,
  value: '<h1><span style="color: red;">Test Link</span></h1>' }
*/
```


## Additional Methods

`.htmlInput()` extends the builtin `Joi.string()` method so you can use any of the built in string methods including `.length()` `.min()` and `.max()` and they will work the same as you would expect when using `Joi.string()` these could be useful if you want to validate the maximum length of a string so that you don't exceed a character limit in your database however you may run into problems if you are using a WYSIWYG editor like TinyMCE or CKEditor and want to set a character limit but you don't want the generated html for bulletpoints, links or styling to count towards that character limit.

To help you validate your HTML strings based on the actual length they will be when displayed in the browser `.htmlInput()` provides several methods. These menthods will also account for html entities such as `&nbsp;` so that they only count as single character.

The tag stripping and the decoding of HTML entities for these methods is provided by the [string](https://www.npmjs.com/package/string) package.

### .displayLength(limit, [encoding])

Validates the length of a string ignoring HTML tags and converting HTML entities to characters. The return value remains unchanged.

```
const htmlInput = require('joi-html-input');
const Joi = require('joi').extend(htmlInput);

const htmlString = '<div><h1 class="align-center">Test Heading</h1></div>';
const joiSchema = Joi.htmlInput().displayLength(12);
const results = Joi.validate(htmlString, joiSchema);

console.log(results);

/* Expected output:
{ error: null,
  value: '<div><h1 class="align-center">Test Heading</h1></div>' }
*/
```

### .displayMin(limit, [encoding])

Validates the minimum number of characters in a string ignoring HTML tags and converting HTML entities to characters. The return value remains unchanged.

```
const htmlInput = require('joi-html-input');
const Joi = require('joi').extend(htmlInput);

const htmlString = '<div><h1 class="align-center"></h1></div>';
const joiSchema = Joi.htmlInput().displayMin(12);
const results = Joi.validate(htmlString, joiSchema);

console.log(results);

/* Expected output:
{ error:
  { ValidationError: "value" is not allowed to be empty
    ...
  },
  value: '<div><h1 class="align-center"></h1></div>' }
*/
```
Additionally `.displayMin()` also has support for an optional encoding parameter just like `Joi.string().length()` here is an example.

### .displayMax(limit, [encoding])

```
const htmlInput = require('joi-html-input');
const Joi = require('joi').extend(htmlInput);

const htmlString = '<div><h1 class="align-center">Test Heading</h1></div>';
const joiSchema = Joi.htmlInput().displayMax(12);
const results = Joi.validate(htmlString, joiSchema);

console.log(results);

/* Expected output:
{ error: null,
  value: '<div><h1 class="align-center">Test Heading</h1></div>' }
*/
```


## Additional Examples

Here are some more examples that you might find useful. If you have any suggestions for additional examples please submit them via a pull request on github.


### Character Encoding

Just like the builtin `Joi.string().length()` the `.displayLength()` `.displayMin()` and `.displayMax()` also have support for an optional encoding parameter, here is an example.

```
const htmlInput = require('joi-html-input');
const Joi = require('joi').extend(htmlInput);

const htmlString = '<div><span class="small-text">Copywrite \u00A9</span></div>';

// With out utf8 character encoding the length of this string will be 16
const joiSchema1 = Joi.htmlInput().displayLength(12);
const results1 = Joi.validate(htmlString, joiSchema1);

// So this will produce an error
console.log(results1);

/* Expected output:
{ error:
  { ValidationError: "value" length must be 12 characters long
    ...
  },
  value: '<div><span class="small-text">Copywrite ©</span></div>' }
*/

// With utf8 character encoding the length of this string will be 12
// Note: \u00A9 = 2 characters
const joiSchema2 = Joi.htmlInput().displayLength(12, 'utf8');
const results2 = Joi.validate(htmlString, joiSchema2);

// So this will validate
console.log(results2);

/* Expected output:
{ error: null,
  value: '<div><span class="small-text">Copywrite ©</span></div>' }
*/
```

### HTML Entities

HTML entites will be decoded before the string lengths are checked so entites like `&nbsp;` will only count as 1 character.

```
const htmlInput = require('joi-html-input');
const Joi = require('joi').extend(htmlInput);

const htmlString = '<div><h1>Test&nbsp;Heading</h1></div>';
const joiSchema = Joi.htmlInput().displayLength(12);
const results = Joi.validate(htmlString, joiSchema);

console.log(results);

/* Expected output:
{ error: null,
  value: '<div><h1>Test&nbsp;Heading</h1></div>' }
*/
```

### Sanitization And Validation Together

All the additonal methods provided by `.htmlInput()` can be chained with other methods including those provided by Joi. Here is an example of multiple methods being used together.

```
const sanitizeConfig = {
  'allowedTags': [
    'h1'
  ],
  'allowedAttributes': {
    'h1': [
      'id'
    ]
  }
};

const htmlString = '<h1 id="headline">Test Heading<script>alert(\'Test\')</script></h1>';
const joiSchema = Joi.htmlInput().allowedTags(sanitizeConfig).displayLength(12).max(50);
const results = Joi.validate(htmlString, joiSchema);

console.log(results);

/* Expected output:
{ error: null,
  value: '<h1 id="headline">Test Heading</h1>' }
*/
```


## Disclaimer

This package is not an official part of Joi nor is it produced by any member of the Joi team. It is not security tested, if you want to use this package in your project please read the full license first (link below) and review the code for yourself before using.


## License

[BSD-3-Clause](LICENSE.md)
