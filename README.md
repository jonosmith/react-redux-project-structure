# React/Redux Project Structure

The following is an example of how to structure a React/Redux application in a scalable way. This may be overkill for simple applications, but is more aimed at SPAs that are foreseen to grow quite large and a way to manage the complexity of the app is required.

## Table of Contents

* [Typical Strategies for Organising the Code](#typical-strategies-for-organising-the-code)
  * [Organise by File Type](#organise-by-file-type)
  * [Organise by Feature](#organise-by-feature)
* [The Hybrid Approach](#the-hybrid-approach)
* [Folder structure](#folder-structure)
  * [Index Files](#index-files)
  * [Using Data from Modules](#using-data-from-modules)
* [State Structure](#state-structure)

## Typical Strategies for Organising the Code

Typical strategies for organising code are organising by type and organising by feature.

### Organise by File Type

Typical Redux projects organise code by file type ie. all the `action` files will be in an `actions` folder, all the `reducer` files will be in a `reducers` folder etc. Example:

* actions
  * articles.js
* components
  * ArticlesList
* containers
  * ArticlesList
* reducers
  * articles.js
* selectors
  * articles.js

Some issues with this approach are:

* It doesn't scale. These folders balloon in size as more features are added
* Not immediately obvious what features the app consists of, what the screens are and how everything is connected
* The files for particular parts of the app are spread across a number of folders, meaning there is alot of switching between folders when working on a particular feature, but more importantly making it harder to reuse features of an app

### Organise by Feature

In this approach, files are grouped according to features ie. a dashboard, blog article, subscription payments etc. Example:

* articles
  * components
  * actions.js
  * constants.js
  * reducer.js
  * selectors.js
  * index.js
* user
* payment

Some issues with this approach are:

* Ties UI structure to data structure
* Lines between different features are often blurry. Quite easy to have a couple of very large features that are not clear how to break up
* Features often require data from other features, and thus become tightly coupled

## The Hybrid Approach

This approach is similar to organising by feature, but separates the data and view layers. We get the benefit of grouping files together into meaningful `modules` that isolate functionality, indicate app functionality just by looking at the code and can be easily reused, but avoiding the downside of tightly coupled features by separating out the data layer.

Key principles

* Data common to all pages goes in a `/modules` directory, with all other features nested in a `/pages` directory
* Each module has an `index.js` file that exports it's contents meant to be exposed eg. `actions`, `constants`, `reducer`, `select` (memoised selectors for accessing state owned by the module) etc.
* Unit tests colocated with the files they are testing
* Each module has a `constants.js` file which has a `NAME` constant for the name of the module. This `NAME` is used in action names for the module as well as for the key in redux state where the module is mounted

## Folder structure

Summary

* **/common** - Common helpers, constants, modules etc. that can be imported and used by any file in the app
* **/components** - Components common to the whole app, like primitives such as `Row`, `Col`, `Link` etc.
* **/modules** - App level data modules (no views) - control data that is applicable to multiple views
* **/pages** - The main pages (features) of the app. Has nested features (eg. `comment` module in an `article` page for instance). The top level folders correspond to top level app routes

NOTES:

* Features cannot access data from siblings - only parents! If two sibling features need access to the same data, the data should be moved up. In the case of pages, this means creating a new common module in the `/modules` folder. For nested features, this means moving the data up to the parent feature
* The aim should be to push as much down from the `/modules` folder to the nested features in the `/pages` folder as possible. This is to avoid cluttering up high level layers - there is no point creating a common data `module` that only a single page uses

Example:

* common
  * lib
  * utils.js
* components
* modules
  * user *- Data for the logged in user*
* pages
  * home
  * payment
  * user *- User's profile page. Displays list of articles*
    * \_\_tests\_\_
    * article
      * \_\_tests\_\_
      * components
      * actions.js
      * constants.js
      * reducer.js
      * selectors.js
      * index.js
    * components
    * actions.js
    * constants.js
    * reducer.js
    * selectors.js
    * index.js

### Index Files

An `index.js` file would look like this:

```javascript
import * as actions from './actions'
import * as constants from './constants'
import reducer from './reducer'
import sagas from './sagas'
import * as select from './selectors'

export default {
  ...constants,
  actions,
  reducer,
  sagas,
  select
}
```

### Using Data from Modules

As an example, when in a container component in a page and accessing some data from a higher level module you would do this:

```javascript
import user from '../../modules/user'

...

function mapStateToProps(state) {
  return {
    currentUserName: user.select.name(state)
  }
}
```

## State Structure

* Module names all uppercase for consistency with action names
* Every module/page is organised into `data` and `ui` sections
  * `data` - typically resources obtained from the backend eg. blog posts
  * `ui` - UI relevant data eg. ui lifecycle such as loading state, errors etc.

```javascript
{
  app: {
    USER: {
      data: {
        id: 1234,
        first_name: "John",
        last_name: "Smith"
      },
      ui: {
        isLoading: false,
        isSet: true
      }
    }
  },
  pages: {
    HOME: { ... },
    PAYMENT: { ... },
    USER: {
      ARTICLE: {
        data: {
          content: "<h2>Strategies for organising code</h2>There are a few..."
        },
        ui: {
          isInEditMode: false,
          isLoading: false
        }
      },
      data: {
        articles: [{ id: 1, title: "How to Structure React/Redux apps", ... }],
        totalArticles: 50,
        totalLoaded: 10
      },
      ui: {
        isLoading: false,
        isSet: true
      }
    }
  }
}
```

At a glance we can tell exactly what the application structure is from looking at the state. The same uppercased names used in the module's `constants.js` files are used in state as well as when naming the actions for consistency.

## Action Naming

Syntax is `NOUN_VERB`. Namespacing with `/`. App level actions namespaced with `APP/`. Actions in pages/feature modules are namespaced according to `NAME` of feature in `constants.js` files

Examples:

* `APP/USER_FETCH`
* `APP/USER_FETCH_SUCCESS`
* `USER/ARTICLES_FETCH`
* `USER/LOADING_START`
* `USER/ARTICLES_FETCH_SUCCESS`
* `USER/LOADING_STOP`

As we can see actions are grouped nicely according to the feature they relate to.