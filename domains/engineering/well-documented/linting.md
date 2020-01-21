---
title: Readme files
nav_order: 2
parent: Well documented
grand_parent: Engineering
---

# Linting

*Definition* - Linting is the process done by a program called as **Linter**. This process highlights the code
which has a potential problem, checked against the set of standard rules/guidelines integrated in the project.
It identifies and reports the potential syntax or semantic errors.

#### Linters 
- TSLint (*Angular*)
- ESLint (*Angular/React*)

### 1. TSLint

We will use AirBnb Javascript guidelines.
 
 To integrate TSLint in your Angular project follow steps below:
 1. Create a `tslint.json` file in the root of the project and paste the below set of standard
 rules in it.
 
    ```json
    {
      "rulesDirectory": [
        "node_modules/codelyzer"
      ],
      "rules": {
        "arrow-return-shorthand": true,
        "callable-types": true,
        "class-name": true,
        "comment-format": [
          true,
          "check-space"
        ],
        "curly": true,
        "eofline": true,
        "forin": true,
        "import-blacklist": [
          true
        ],
        "import-spacing": true,
        "indent": [
          true,
          "spaces"
        ],
        "interface-over-type-literal": true,
        "max-line-length": [
          true,
          140
        ],
        "member-access": false,
        "member-ordering": [
          true,
          {
            "order": [
              "static-field",
              "instance-field",
              "static-method",
              "instance-method"
            ]
          }
        ],
        "no-arg": true,
        "no-bitwise": true,
        "no-console": [
          true,
          "debug",
          "info",
          "time",
          "timeEnd",
          "trace"
        ],
        "no-construct": true,
        "no-debugger": true,
        "no-duplicate-super": true,
        "no-empty": false,
        "no-empty-interface": true,
        "no-eval": true,
        "no-inferrable-types": [
          true,
          "ignore-params"
        ],
        "no-misused-new": true,
        "no-non-null-assertion": true,
        "no-shadowed-variable": true,
        "no-string-literal": false,
        "no-string-throw": true,
        "no-switch-case-fall-through": true,
        "no-trailing-whitespace": true,
        "no-unnecessary-initializer": true,
        "no-unused-expression": true,
        "no-use-before-declare": true,
        "no-var-keyword": true,
        "object-literal-sort-keys": false,
        "one-line": [
          true,
          "check-open-brace",
          "check-catch",
          "check-else",
          "check-whitespace"
        ],
        "prefer-const": true,
        "quotemark": [
          true,
          "single"
        ],
        "radix": true,
        "semicolon": [
          true,
          "always"
        ],
        "triple-equals": [
          true,
          "allow-null-check"
        ],
        "typedef-whitespace": [
          true,
          {
            "call-signature": "nospace",
            "index-signature": "nospace",
            "parameter": "nospace",
            "property-declaration": "nospace",
            "variable-declaration": "nospace"
          }
        ],
        "typeof-compare": true,
        "unified-signatures": true,
        "variable-name": false,
        "whitespace": [
          true,
          "check-branch",
          "check-decl",
          "check-operator",
          "check-separator",
          "check-type"
        ],
        "directive-selector": [
          true,
          "attribute",
          "app",
          "camelCase"
        ],
        "component-selector": [
          true,
          "element",
          "app",
          "kebab-case"
        ],
        "use-input-property-decorator": true,
        "use-output-property-decorator": true,
        "use-host-property-decorator": true,
        "no-input-rename": true,
        "no-output-rename": true,
        "use-life-cycle-interface": true,
        "use-pipe-transform-interface": true,
        "component-class-suffix": true,
        "directive-class-suffix": true,
        "no-access-missing-member": true,
        "templates-use-public": true,
        "invoke-injectable": true
      }
    }
    
    ```
2. WebStorm IDE is intelligent enough to pick your TSLint, whereas to enable TSLint in VS Code
 [download this plugin](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-tslint-plugin) into your code editor.


### 2. ESLint
