# Inker - email development & transactional delivery workflow

[![Build Status](https://travis-ci.org/posabsolute/inker.png?branch=master)](https://travis-ci.org/posabsolute/inker) [![Code Climate](https://codeclimate.com/github/posabsolute/inker/badges/gpa.svg)](https://codeclimate.com/github/posabsolute/inker) [![Join the chat at https://gitter.im/posabsolute/inker](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/posabsolute/inker?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

## Basics

Big mess of tables no more. Inker provides all the mechanics for creating sane email templates and keeping a clean codebase.

Documentation [http://posabsolute.github.io/inker/](http://posabsolute.github.io/inker/)

Inker is :

Coding

* Built on top of Zurb Ink
* Sane CSS components structure with [sass](http://sass-lang.com/)
* Sane HTML components structure with [nunjucks](http://mozilla.github.io/nunjucks)
* Localization
* Auto generate template to HTML documents with inlined CSS
* Auto deployment on [litmus](https://litmus.com/) and [Email on Acid](https://www.emailonacid.com) for testing
* Auto deployment to any email address for testing

REST API Email Delivery

* Asynchronous for a warp speed response (you can use it in sync mode too)
* Generate emails with custom data on the fly
* Integrate all major email delivery providers, just put you account informations
* Failover, when one goes down we redirect the request to another provider
* Logs! hipchat, slack, logtenries, push notifications with push bullet are all integrated but you can easily add your own too


## Getting started

Inker requires **npm** & **grunt** to be already installed.

```bash
git clone https://github.com/posabsolute/inker.git
gem install premailer
cd inker && npm install
```

You have now everything you need to use inker. Your first stop would be the examples folder in src/templates to help you get started.


## Available grunt commands

* grunt watch *- Watch source folder for changes & generate dist files*
* grunt css *- Build CSS*
* grunt html *- Build HTML templates*
* grunt build *- Build css, html & text version*
* grunt connect *- test emails in your browser from the root folder (http://0.0.0.0:8555/)*
* grunt email *- Send a test email to any email inbox*
* grunt litmus *- Send a test email to litmus*
* grunt eoa *- Send a test email to Email on Acid*

## CSS with Inker

Inker use [Zurb Ink](http://zurb.com/ink/) responsive css framework, everything in Ink is available in inker, please refer to their [documentation](http://zurb.com/ink/docs.php). Inker also uses the meta framework [ITCSS](http://csswizardry.net/talks/2014/11/itcss-dafed.pdf) for the files and folders structure. The css can be found in **src/css**

**base.scss** is importing all needed files for inker, if you add a css component you must import it in base.scss.

*It is important to note that since we inline style to html nodes it makes no sense to pick and choose css components as it will makes no difference on the file size in the end*

### Responsive

Responsive rules are in the folder **8_trumps**, please note that these rules are added to the document's head instead of inlined using data-ignore="ignore" in the html templates.

### Ignore css inlining

```html
<!-- external styles -->
<link rel="stylesheet" data-ignore="ignore"  href="../css/style.css" />

<!-- embedded styles -->
<style data-ignore="ignore">
 /* styles here will not be inlined */
</style>
```

### Adding new themes

Open 7_themes, you will see there is already a folder called sidebarhero used for the sidebar hero template. Add a new folder in 7_themes for your template, your main css file will be automatically generated in dist/css/[your folder].

Then in your html, use:
```html
  {% block theme_css %}<link href="../../css/sidebarhero/sidebarhero.css" media="all" rel="stylesheet" type="text/css" />{% endblock %}
```

## HTML with inker

Inker uses [Mozilla Nunjucks](http://mozilla.github.io/nunjucks/) to build html templates, please see nunjucks' [documentation](http://mozilla.github.io/nunjucks/templating.html) for more information on how you can take inker even further.

Inker as an html component structure that uses nunjucks macros. Example of component:

```html
{% macro button(label='default', link='#', class='', align='left') %}
  <table class="button {{class}}" align="{{align}}">
    <tr>
      <td>
        <a href="{{link}}">{{label}}</a>
      </td>
    </tr>
  </table>
{% endmacro %}
```

```html
// Import the component in the base.html file in components folder.
{% from "components/component.button.html" import button %}
```


Usage in html template :
```javascript
{{ button('Go to google', 'http://www.google.com', 'button-green', 'left'); }}
```

**When creating new components, remember to add them to the base.html file situated in _src/templates/components/components_**


### Creating new html templates

Open the template's folder, you should see a folder transactional, You can add your own folder here.

#### A base template example

```html
{% extends "components/_base.html" %}
  {% block main_css %}
    <link href="../../css/main.css" media="all" rel="stylesheet" type="text/css" />
  {% endblock %}
  {% block theme_css %}
    <link href="../../css/sidebarhero/sidebarhero.css" media="all" rel="stylesheet" type="text/css" />
  {% endblock %}

  {% block responsive_css %}
    <link href="../../css/responsive.css" media="all" data-ignore="ignore" rel="stylesheet" type="text/css" />
  {% endblock %}

  {% block meta_title %}
    Email title in document head
  {% endblock %}

{% block mainContent %}
  {% block header %}
    {% include "/templates/sidebar_hero/header.html" %}
  {% endblock %}
  {% block content %}
    {% include "/templates/sidebar_hero/content.html" %}
  {% endblock %}
{% endblock %}
```


### Inker with your back-end templating engine & application

There is a bit of a duality with Inker, it uses a templating engine to generate templates but must not use it to render personalized data on first render so that it can be generated by another templating engine or by the server.

#### Passing data and custom template logic by string

If you have a conflict with nunjucks template syntax, you can add your variables and custom logics as strings likewise :

```html
  <h4>{{ '{{ name }}' }} last step and you're done!</h4>

  <p>
    {{ '{% for item in loop %}' }}
    {{ '{{ item }}' }} &nbsp;
    {{ '{% endfor %}' }}
  </p>
```

In the final email template, this will look like :

```html
  <h4>{{ name }} last step and you're done!</h4>

  <p>
    {% for item in loop %}
      {{ item }}
    {% endfor %}
  </p>
```

#### Using a custom syntax

Inker enables you to use a custom syntax so that it does not interfere with your templating engine of choice. If you do use a custom syntax, you will have to replace the current implementation with the new syntax.

```javascript
nunjucks: {
  options: {
    tags : {
      blockStart: '<%',
      blockEnd: '%>',
      variableStart: '<$',
      variableEnd: '$>',
      commentStart: '<#',
      commentEnd: '#>'
    }
  }
}
```

### Inker with dynamic custom data

Inker can use json files as a source of dynamic data. Please note that you can also use the provided rest api to test custom data.

This is a built-in feature of [grunt-nunjucks-2-html](https://www.npmjs.com/package/grunt-nunjucks-2-html).
```javascript
nunjucks: {
  options: {
    data: grunt.file.readJSON('data.json')
  }
}
```


## Included components
Current list of implemented components. (I am always looking to add more components to inker.)

### Buttons

*Options :*
* Label
* Link
* Class to add
* Alignment

Usage :
```javascript
button('Go to google', 'http://www.google.com', 'button-green', 'left');
```

Render :
```html
  <table class="button button-green" align="left">
    <tr>
      <td>
        <a href="http://www.google.com">Go to google</a>
      </td>
    </tr>
  </table>
```

### Progress Bar

*Options:*
* width of bars
* progress
* label
* Class to add


Usage :
```javascript
progressbar('100%', 70, 'Your progress so far', 'progressbar-green');
```

Render :
```html
  <table class='progressbar progressbar-green' cellpadding="0" cellspacing="0" width="100%">
      <tr>
          <td class='foreground' style='' width="70%">
              Your progress so far
          </td>
          <td class="background" width="30%">
              &nbsp;
          </td>
      </tr>
  </table>
```

### Video

Please refer to campaign monitor chart to see which email client supports video, fallback to an image.

*Options :*
* width
* height
* video_src
* video_link
* video_image_placeholder
* class

Usage :
```javascript
video(320, 176, 'http://www.google.com', 'http://www.google.com', 'http://www.google.com', 'video-big');
```

Render :
```html
  <div class="video_holder video-big">
      <video width="320" height="176" controls>
          <source src="{{video_src}}.mp4" type="video/mp4">
          <source src="{{video_src}}.ogg" type="video/ogg">
            <a href="{{video_link}}" ><img height="176"
              src="{{video_image_placeholder}}" width="320" /></a>
      </video>
  </div>
```

### Image Caption

*Options :*
* Label
* Width
* Class
* Align

Usage :
```javascript
caption('This is a cat', '320px', 'caption-red', 'left');
```

Render :
```html
  <table class="caption caption-red" cellpadding="0" cellspacing="0" width="320px" border="0", align='left'>
      <tr>
          <td align="center">
              <img src="http://placekitten.com/g/300/300" alt="" />
              <div>This is a cat</div>
          </td>
      </tr>
  </table>
```

### Panel

*Options :*
* Label
* size (default: twelve)
* Class

Usage :
```javascript
panel('This is a panel', 'twelve', 'panel-red');
```

Render :
```html
  <table class="twelve panel-red columns">
    <tr>
      <td class="panel">

        This is a panel

      </td>
      <td class="expander"></td>
    </tr>
  </table>
```

## Localization

Inker can generate the same template in multiple languages. Strings are defined in the folder *locales* in the yaml format. Don't forget that inker pre-generate templates, strings are not generated on the fly.

You can set default rendering languages in the gruntfile.js in the nunjucks section.
```javascript
nunjucks: {
  options:{
    langs : ["en_US"],
```

Follow the same conventions while creating your localization files, in this case it would be *locales/en_US.yml*.


## schema.org, Email Augmented Experience

By adding schema.org markup to the emails you send your users, you can make that information available across their Google experience, and make it easy for users to take quick action. Inbox, Gmail, Google Calendar, Google Search, and Google Now all already use this structured data.

Inker currently only implement actions. 

JSON LD components use dynamic data generally linked directly to the customer you are sending the email. JSON-LD complete implementation is available using the Inker API or you can customize the components to use your own templating engine.

**DKIM/SPF signature for email clients is required when using schema.org markup**

### Example:

JSON-LD content is a script tag embedded in the email body, this is already taken care of by Inker.

Usage in the template :
```javascript
{% block jsonLD %} jsonldGoto(); {% endblock %}
```

Then when sending your email using Inker API you would add the customer data like this:

```javascript
"POST /email"
// Post data
{
  "data" : {
    "collection": "transactional", // required, folder(s) in output
    "template": "alert", // required, template filename
    "locale" : "en_US", // optional, default to en_US
    "variables": {
      "jsonLD" : {
        "gotoType" : "ViewAction",
        "gotoTarget" : "https://watch-movies.com/watch?movieId=abc123",
        "gotoName" : "Watch the 'Avengers' movie online"
      }    
    }
  },

}
```
It's important to remember to put your JSON-LD content in variables -> jsonLD. As said before you can also modify the json-ld components to use your own templating engine. The components are in : *src/templates/components/jsonld*


### Go to Action

Go-To Actions take the user to your website where the action can be completed. Unlike One Click Actions, go-to actions can be interacted with multiple times.

https://developers.google.com/gmail/markup/reference/go-to-action

!(https://developers.google.com/gmail/markup/images/actions-go-to-action.png)

Usage in the template :
```javascript
{% block jsonLD %} jsonldGoto(); {% endblock %}
```

*Options :*
* gotoType {string} ViewAction || TrackAction
* gotoTarget {string} https://watch-movies.com/watch?movieId=abc123
* gotoName {string} Watch the 'Avengers' movie online

### One Click Action

You may add a one-click confirm button to emails requiring users to approve, confirm and acknowledge something. Once the user clicks on the button, an http request will be issued from Google to your service, recording the confirmation. ConfirmAction can only be interacted with once.

https://developers.google.com/gmail/markup/reference/one-click-action

Usage in the template :
```javascript
{% block jsonLD %} jsonldOneClick(); {% endblock %}
```

*Options :*

* action.type {string} ConfirmAction || SaveAction
* action.name {string} textshown in inbox
* action.url  {string} Url action
* description {string} Call description


### Review Action

Use to declare a review action. Gmail may show a review button next to the email, which will prompt the user for a numeric review value and or a user comment.

https://developers.google.com/gmail/markup/reference/review-action

Usage in the template :
```javascript
{% block jsonLD %} jsonldReview(); {% endblock %}
```

*Options :*

* reviewType {string} FoodEstablishment || Movie || Product
* reviewName {string} Joe's Diner
* reviewUrl  {string} Action Url
* reviewDescription {string} Hope you enjoyed your meal with us, please review your experience



### RSVP Action

You may add a one-click confirm button to emails requiring users to approve, confirm and acknowledge something. Once the user clicks on the button, an http request will be issued from Google to your service, recording the confirmation. ConfirmAction can only be interacted with once.

https://developers.google.com/gmail/markup/reference/rsvp-action

Usage in the template :
```javascript
{% block jsonLD %} jsonldRSVP(); {% endblock %}
```

*Options :*

* startDate {date} 2015-04-18T15:30:00Z
* endDate {date} 2015-04-18T16:30:00Z
* EventName {string} Google Party
* addressStreet {string} 24 Willie Mays Plaza
* addressLocality {string} San Francisco
* addressRegion {string} CA
* postalCode {string} 94107
* addressCountry {string} USA
* attendance http://schema.org/RsvpAttendance/Yes || http://schema.org/RsvpAttendance/No || http://mysite.com/rsvp?eventId=123&value=maybe



## Sending a test email to your inbox

Inker uses [grunt-nodemailer](https://github.com/dwightjack/grunt-nodemailer) to send tests.

You can easily change what email templates is being sent in the **gruntfile.js**.

However a better way to use it would be to change the path directly from the grunt command. This makes it possible to send tests really fast with different templates.

```bash
// Override default src provided in gruntfile
grunt email --fileSrc=dist/output/en_US/transactional/alert.html
```

Config example :
```javascript
nodemailer: {
  email: {
    options: {
      transport: {
        type: 'Sendmail',
      },
      recipients: [
        {
          email: 'your.email@gmail.com',
          name: 'Jane Doe'
        }
      ]
    },
    src: ['dist/output/*.html']
  }
}
```

## Testing with Email on Acid

Grunt litmus [documentation](https://github.com/jeremypeter/grunt-litmus).

### Email on acid configuration

You must set the testing email provided by Email on Acid in the gruntfile.

```javascript
emailonacid :{
   options: {
    transport: {
      type: 'Sendmail'
    },
    recipients: [
      {
        email: 'username@emailonacid.com',
        name: 'Email on Acid'
      }
    ]
  },
  src: ['dist/output/fr_CA/transactional/alert.html']           
}
```

### Overriding default files sent to litmus

grunt eoa --fileSrc=dist/output/fr_CA/transactional/alert.html


## Testing with litmus

Grunt litmus [documentation](https://github.com/jeremypeter/grunt-litmus).


### Overriding default files sent to litmus

grunt litmus:dist/output/sidebar_hero/index.html

### Default config
The most used email clients are already set in the gruntfile.

```javascript
litmus: {
  test: {
    src: ['email.html'],
    options: {
      username: 'username',
      password: 'password',
      url: 'https://yourcompany.litmus.com',
      clients: [
        //gmail
        'gmailnew', 'ffgmailnew', 'chromegmailnew',
        // outlook
        'ol2002', 'ol2003', 'ol2007', 'ol2010', 'ol2011', 'ol2013',
        // hotmail
        'outlookcom', 'ffoutlookcom', 'chromeoutlookcom',
        //Yahoo
        'chromeyahoo',
        //applemail
        'appmail6',
        //mobile
        'iphone6plus', 'iphone6', 'iphone5s', 'androidgmailapp', 'android4', 'ipad',
        // spam check
        'messagelabs'
      ]
    }
  }
},


```
## Email REST API

Inker comes with a nodejs rest api that can handle rendering templates with custom variables and send emails through SMTP to any email provider.

There is a public [Postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop) collection for your convenience for testing the api locally.
https://www.getpostman.com/collections/47367188a40bdd43de8d

### Starting the server
Install all dependencies
```javascript
npm install
cp src/server/configs/_configs.providers.auth.js src/server/configs/configs.providers.auth.js
cp src/server/configs/_configs.js src/server/configs/configs.js
```

Start the server
```javascript
node src/server/server.js
```

### Authentication

The server has a basic auth system. It expects a token for each request. This token is set in */src/server/config.js*

Default :
```javascript
X-Authorization-Token : asd98a7s9898asdaSDA(asd987asda*(&*&%))
```

### Configs



### API Templates rendering

The API also uses nunjucks to render custom variables. Remember that since we use the same rendering engine (nunjucks) for both front-end & server side, you must use strings in your template to add variables and custom template logics.

Example:
```html

  <h4>{{ '{{ name }}' }} last step and you're done!</h4>

  <p>
    {{ '{% for item in loop %}' }}
    {{ '{{ item }}' }} &nbsp;
    {{ '{% endfor %}' }}
  </p>
```

You can configure the template syntax in */src/server/config.js*.


"GET /templates/folder/template"

Base path is dist/output

Example:
```javascript
"GET /collections/:folders/templates/:filename?name=Cedric&loop[]=1&loop[]=2&loop[]=3"
// with localization
// Example with french
"GET /collections/:folders/templates/:filename/locale/fr_CA?name=Cedric&loop[]=1&loop[]=2&loop[]=3"

```

### API Email sending

The api can send emails to a variety of SMTP services, MailGun, Mandrill, SendGrig, Gmail, etc. You can see the full list in */src/server/serviceAuth.js*

Example :
```javascript
"POST /email"
// Post data
{
  "data" : {
    "collection": "transactional", // required, folder(s) in output
    "template": "alert", // required, template filename
    "locale" : "en_US", // optional, default to en_US
    "variables": {
      "name":"Cedric",
      "loop": ["1","2","3"]      
    }
  },
  "options" : {
    "textVersion" : true,
    "from": "sender@address",
    "to": "cedric.dugas@gmail.com",
    "subject": "hello",
    "text": "hello world!"
  },
  // optional, default to 'service' set in config.js
  "service" : {
    "name":"MailGun"
  }
}
```

### Sending a custom text version

When enabling the text version option, the api will automatically fetch the .txt file at the same folder level as the html version.

If the html template is in: *output.en_US/transactional/alert.html*

The txt version should be here: *output.en_US/transactional/alert.txt*

### Generate text version

You can also Generate automatically the text version from your html template on the fly,

Example :
```javascript
  "options" : {
    // Enable text version using a custom txt template.
    "textVersion" : true,
    // Generate text version from html template
    "textVersionFromHTML" : true,
    "from": "sender@address",
    "to": "cedric.dugas@gmail.com",
    "subject": "hello",
    "text": "hello world!"
  },

```


### Email delivery service Authentication

You must add your credentials in */src/server/configs.providers.auth.js*

### Failover Email Delivery Provider

You can set a default failover service provider in */src/server/configs.js*


```javascript
// main mailing service used to send email
"service" : "MailGun",
// failover service when main service is down
"failOver" : "SendGrid",
```


### Logs

Inker can log errors & alert you when one your main email sending provider is down, when an application request an inexistent template or when there is an error in a template.


Log services are defined in */src/server/configs.js*

```javascript
// active logs
"sendLogs" : ['hipchat','logentries'],
// logs configs
"logs" : {
  "hipchat":{
    "room" : "HIPCHAT_ROOM_ID",
    "token" : "MY_TOKEN"
  },
  "logentries" : {
    "token" : "MY_TOKEN"
  }
}
```

#### Adding a log provider

You can add a log provider in */src/server/configs.logs.js*. Implementation example:

```javascript
"hipchat": {
    "level":2,
    // Module required for the log provider 
    "module":"node-hipchat",
    // Executed after the module is loaded in
    // Generally used to instanciate the provider
    "afterRequire": function(loggers){
      loggers.hipchatLog = new loggers.hipchat(configs.logs.hipchat.token);
    },
    // Actual log function
    // loggers is passed bavk & contains all the logs providers.
    log : function(loggers, type, messageText, messageJson){
      var params = {
          room: configs.logs.hipchat.room,
          from: 'Email Server',
          message: messageText,
          color: 'yellow'
        };

      loggers.hipchatLog.postMessage(params, function(data) { });
    }
},
```

## Sponsored by
A big thank you for supporting Inker.

![Email on Acid](http://inker.position-absolute.com/eoa.png)

[Email on Acid](https://www.emailonacid.com) - Tools that simplify email testing and analytics.


## Contributions

I'm always happy to accept contributions, I'm currently looking for more components and examples, but please follow ITCSS guidelines, and please test your new components in the most used email clients.

## Licence

The MIT License (MIT)

Copyright (c) 2014 Cedric Dugas [http://www.position-absolute.com](http://www.position-absolute.com)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

