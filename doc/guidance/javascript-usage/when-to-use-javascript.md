# When to use Javascript in applications    <!-- omit in toc -->

## A guide to appropriate use               <!-- omit in toc -->

- [Introduction](#introduction)
- [What we can do with Javascript](#what-we-can-do-with-javascript)
  - [Improving the functionality of otherwise functional pages](#improving-the-functionality-of-otherwise-functional-pages)
  - [Back-end Javascript](#back-end-javascript)
  - [Tooling](#tooling)
- [What we can't do with Javascript](#what-we-cant-do-with-javascript)
  - [Use frontend frameworks*](#use-frontend-frameworks)
  - [Form validation](#form-validation)
  - [Animation](#animation)
- [Final notes](#final-notes)
  - [Turning off Javascript isn't the problem](#turning-off-javascript-isnt-the-problem)

### Introduction

As we are developing applications for a government organisation, we are required to apply particular standards to our use of Javascript within web applications, and we have to apply these to all web applications that we make, with _relatively few_ exceptions.  This doesn't mean, however, that we can't make use of Javascript.  This document aims to summarise where, and how, use of Javascript is appropriate.

### What we can do with Javascript

GDS has more complex guidance, but in distilled format, we can use Javascript for the following.

#### Improving the functionality of otherwise functional pages

Javascript functionality is acceptable where it is adding useful, but not essential, functionality to pages.  The following point must all be true:

- The page must be functional if Javascript is not loaded
- Additional functions added by Javascript must be 'nice-to-have', and non-essential to functionality
- Adding Javascript will not cause a detriment to user experience for any users

Regarding the point of 'detriment to users' bear the following in mind:

- Javascript functionality must be not make pages harder to use for those who view sites by alternative methods for accessibility reasons, including:
  - Screen readers
  - Magnifiers
- Javascript should avoid Flash Of Unstyled Content (FOUC)

FOUC is the effect on a page in which the page loads, renders, and is then changed suddenly and jarringly by a Javascript that loads and applies an alteration to the page.  This can be avoided by one of a few techniques:

- Deciding not to use JS for features that could cause this.
- Applying changes to pages in a gradual manner that doesn't ruin user experience
- Using page scroll tracking to maintain the view that the user is scrolled to (e.g if you fold up elements, or move elements, don't move those in viewport)

Some examples of good things we can do to improve pages with Javascript:

- Add search to `select` lists
- Add useful but non-essential features, such as embedded videos or maps, where this adds use, but the page still achieves its aims otherwise.
- Adding sorting or searching to tables, provided the functionality is either not essential, or backed up by a backend driven version.

#### Back-end Javascript

Javascript systems to provide back-end systems are not restricted to use, and shouldn't be confused with front-end Javascript rules.

#### Tooling

Javascript systems for tooling such as:

- Task runners like Gulp/Grunt/NPM Tasks
- Package managers like NPM and Yarn
- Bundlers like Webpack, Rollup, and Parcel

Can be used freely, as they don't affect page functionality.

### What we can't do with Javascript

For accessibility and browser support reasons, there are far more things you can't do with Javascript:

#### Use frontend frameworks*

We can't use systems like React or Angular on the front-end, as they don't render a page at all without javascript running.

> \* We could use these types of systems with server side rendering systems, where available, but these have some limitations, and complications, and so this should be considered when assessing possibilities.

#### Form validation

It's possible to use Javascript for form validation and still remain acceptable to GDS, but only if the form validation is also done by the server, and the server presents the same error output as the Javascript would.  At that point, although we could use Javascript for the validation, we could only do so if it makes an improvement on behaviour that already exists without, and as such it's not the first priority.

Javascript cannot be used as the sole form validation, except when run on the server side.

#### Animation

Animation should not be done using Javascript for a number of reasons:

- Animation is rarely justified anyway:
  - Animating elements to move around the page ruins user experience in a lot of cases
  - Moving elements around the screen can cause unexpected side-effects to rendering if the user behaves in unexpected ways
- Any animation that may be justified/not intrusive can likely be achieved with CSS

### Final notes

A few other points should be made about the required approach to Javascript in GDS compliant systems:

#### Turning off Javascript isn't the problem

It's a common comment that barely anyone disables Javascript, and so using it would not be harmful.  This misses the point that there are many reasons Javascript may not work, including:

- Network issues
- Slow loading times/poor connections
- Third party failures
- Blocking by systems outside of the control of users
- Lack of mobile bandwidth
- Security policy issues

#### Progressive enhancement and graceful degradation are the key

Pages should be made to work, by concentrating on their content in a certain order:

1. HTML
2. Images
3. Styling
4. Video and Audio
5. Javascript

Graceful degradation is the principle that if you use browser features that are only supported by some browsers that GDS requires support of, the outcome of a browser not supporting the feature should not:

- Show any obviously unintended effects, e.g. broken page layouts.
- Cease to function, nor break other scripts on the page (code must be error tolerant of the failed browser feature)
- Fail to offer vital features
