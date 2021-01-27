---
title: GraphQL naming conventions
nav_order: 5
parent: Well documented
grand_parent: Engineering
---

# GraphQL naming conventions

The GraphQL specification is flexible and doesn't impose specific naming guidelines. However, it's helpful to establish a set of conventions to ensure consistency across your organization. We recommend the following:

- Field names should use `camelCase` like, `currencyList`. Many GraphQL clients are written in JavaScript, Java, Kotlin, or Swift, all of which recommend camelCase for variable names.
- Type names should use `PascalCase` like, `ConversationParticipant`. This matches how classes are defined in the languages mentioned above.
- Enum names should use `PascalCase`.
- Enum values should use `ALL_CAPS`, because they are similar to constants.
- Folder and file names are lower case.
- File names include the name of the component the file represents. Plurals are used in folder names, but not in file names.
- Don’t prefix the files of subfolders with anything because they already live under the them and naming it like `user_profile_queries.rb` or `company_mutation.rb` would be redundant and wouldn't matter.
- Mutation should start with an action they are trying to perform, like, `createCompany`.
- Queries should describe what you are looking for, like, `userComments`.

```
/controllers
/jobs
/models
/channels
/graphql
    /queries
        /user_profile.rb
    /mutations
        /company.rb
```

Folder and file structure
{: .text-grey-dk-000.fs-3.d-flex.flex-justify-around.my-1}
