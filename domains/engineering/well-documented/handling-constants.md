---
title: Handling constants
nav_order: 7
parent: Well documented
grand_parent: Engineering
---

# Handling constants

## What is a constant?
A constant is a data item whose value cannot change during the program’s execution. Thus, as its name implies – the value is constant.

## How to handle constants?
A constant that is required in multiple places in the code should be defined by assigning it to a variable at one place. This variable should then be called everywhere the constant is required.

## Why do constants need to be handled?
- Defining a constant in a single place will help you write clean and readable code.
- It will reduce the need to use comments to describe your code.
- It will help the project by ensuring that a standard set of constants are used.
- If the value of a constant needs to be changed, you need to make a change only where the constant was originally defined, and not everywhere it is being used.

## When do constants need to be handled?
- When they are required at more than one place in the code.
- When they minimise the readability of the code.

## Handling constants in ROR:
1. **SCSS constants**
  - Constants should be assigned to variables in a separate SCSS file which can be present in the ‘base’ folder.\
  Example:
    ```
    app/assets/stylesheets/base/_variables.scss
    ```
  - To make these variables available for use in another SCSS file, add this on the first line: 
    ```ruby 
    @import "base/_variables.scss";
    ```

2. **Ruby constants**
  - A constant that is to be used in the codebase across different files and folders can be defined inside a suitable model to make it globally available.
  - To define such a constant make sure the variable name is all uppercase.\
  Example:
    ```ruby
    THUMBNAIL_DIMENSIONS = [50, 50]
    ```
  - Syntax to call the variable-> ModelClassName::VARIABLE_NAME\
  Example:
    ```ruby 
    PhotoBooth::THUMBNAIL_DIMENSIONS
    ```
    
## Handling constants in NextJS:
1. **Localization constants**
  - These are meant for locale strings and generally would be in a standalone file called `localization.js`
  - As the name suggests it can be used to render it in different languages and have different locale versions for a website.
  - These can include routes, paths etc.
  - Example:
    ```javascript
    export const HTTPS = "https://";
    ```

2. **Constants**
  - Design system constants from SCSS. eg: color hexes, screen widths, etc.
  - ```javascript
    @import "@styles/design-system.scss";
    ```
  - Break these down into scoped files under a folder called constants.
  - These can contain :
          1. Redirects\
                  - These are your redirects file required for NextJS routing.\
                  - Example:
```javascript
export const pageNotFoundRedirect = {
destination: PAGE_NOT_FOUND_ROUTE,
permanent: false,
}
```
          2. Persistence, performance, variants, etc.

## Handling constants in Angular:
1. **Javascript constants**
  - Constants can be defined in a `constants.ts` file which exports each constant individually.
  - Global constants: Place these constants in the root folder.
  - Component level constants: Place these in a separate constants file in the component root.
  - Module level constants: Place these in a separate constants file in the module root.
  - Example export: 
    ```javascript
    export const FOO = 'bar';
    ```
  - Example usage: 
    ```javascript
    import { FOO } from '../constants';
    ```
    
2. **SCSS constants**
  - Each type of constant resides in a separate file. For example, `variables.scss`, `helpers.scss`, `mixins.scss`, etc.
  - These files can be imported in a `global.scss` file so that they can be accessed anywhere in the app.
  - Similar to JS constants, components and modules can have local constants as well.
  - Example export: 
    ```javascript
    $extra-dim-grey: #ECECEC;
    ```
  - Example import: 
    ```javascript
    @import '../variables.scss';
    ```