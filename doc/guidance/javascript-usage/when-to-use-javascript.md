# When to use Javascript in applications    <!-- omit in toc -->

## A guide to appropriate use               <!-- omit in toc -->

- [Introduction](#introduction)
- [What we can do with Javascript](#what-we-can-do-with-javascript)
  - [Improving the functionality of otherwise functional pages](#improving-the-functionality-of-otherwise-functional-pages)
  - [Back-end Javascript](#back-end-javascript)
  - [Tooling](#tooling)
- [What we can't do with Javascript](#what-we-cant-do-with-javascript)
  - [Use frontend frameworks](#use-frontend-frameworks)
  - [Form validation](#form-validation)

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

#### Use frontend frameworks

We can't use systems like React or Angular on the front-end, as they don't render a page at all without javascript running.

We could use these types of systems with server side rendering systems, where available, but these have some limitations, and complications, and so this should be considered when assessing possibilities.

#### Form validation

It's possible to use Javascript for form validation and still remain acceptable to GDS, but only if the form validation is also done by the server, and the server presents the same error output as the Javascript would.  At that point, although we could use Javascript for the validation, we'd be writing it twice for a small gain, so this is _usually_ pointless and unneeded.

Javascript cannot be used as the sole form validation, except when run on the server side.


