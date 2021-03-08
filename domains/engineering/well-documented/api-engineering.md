---
title: Api Engineering
nav_order: 1
parent: Well documented
grand_parent: Engineering
---
# API Engineering

## Principles

- APIs should be easy to understand that there shouldn't be a need for UI onboarding
- To follow a specific process/format to make the API understanding easier
- To set a standard for everyone to be in sync in API understanding
- Letting know the whole front-end team to be aware of new APIs or API changes without having to draft these (slack) messages and tumble on the message format everytime
- Avoid follow-ups in knowing if an API or API change is up on staging/prod
- Keeping the frontend and backend together to lessen the chances of such miscommunications, facilitating smooth application development
- Achieve complete coherence of the teams working on the project

---

## Have a [Changelog](https://changelog.commutatus.com/admin) project

**Goal**: Letting know developers of any new features, improvements, and changes that have been made to an API

**What is Changelog?**

- Managing public APIs where front-end developers or third party developers who need to be kept in the loop of new features or changes on the platform
- Store a list of updates to platforms for power users who are interested in feature updates

**Steps on how to use Changelog:**

1. Create a project aligning it to a company
2. Navigate to projects section
3. Create a log on entering
    - 'Title'        - Title of the log
    - 'Category'     - Category could be a New feature, Improvement, Bug, Breaking change, Fix
    - 'Description'  - Description of the log

    🎯 Regarding an API, API developers need to give (UI devs) full context in the description, on what to do with the API. This includes providing screenshots pointing to each field and describing what it needs to be used for (if not clear from the documentation).
    Below are two reference logs as such:
    - [Log 1](https://changelog.commutatus.com/companies/aiesec/projects/api/deprecation-of-contacted-rest-endpoint)
    - [Log 2](https://changelog.commutatus.com/companies/aiesec/projects/api/data-democratization-crm-data-download)

4. Subscribe to a project you would wish to receive updates in
<br />*Below is an image of a log with log title, type of update (category), description of the log*

  [![Changelog](/assets/images/changelog.png)](/assets/images/changelog.png)


---

## Documentation

Documentation is first‐class feature of GraphQL type systems. To ensure the documentation of a GraphQL service remains consistent with its capabilities, descriptions of GraphQL definitions are provided alongside their definitions and made available via introspection.

### **Description for field**

Descriptions can be added with the `field(...)` method as a *positional argument*, a *keyword argument*, or *inside the block*.

- 3rd positional argument

    ```ruby
    field :name, String, "name of this thing", null: false
    ```

- `description:` keyword

    ```ruby
    field :name, String, null: false,
      description: "name of this thing"
    ```

- Inside the block

    ```ruby
    field :name, String, null: false do
       description "name of this thing"
    end
    ```

### Description for **argument**

- Write description for argument

    ```ruby
    argument :first_name, String, "Description goes here - First name of the user", required: true
    ```

- Define arguments that are required vs optional

  Required arguments
    ```ruby
    argument :category, String, required:true
    ```

  Optional arguments
    ```ruby
    argument :category, String, required:false
    ```
Note: If all arguments are optional it leads to ArgumentError, to prevent this, you must either specify default_value for all keyword arguments or use the **args double splat operator argument in the method definition

<details markdown="1">
  <summary>Defining Enum Types</summary>
  {:.pointer}
  - In your application, enums extend {{ "GraphQL::Schema::Enum" | api_doc }} and define values with the `value(...)` method:

    ```ruby
    # app/graphql/types/base_enum
    class Types::BaseEnum < GraphQL::Schema::Enum
    end

    # app/graphql/types/media_category.rb
    class Types::MediaCategory < Types::BaseEnum
      value "AUDIO", "An audio file, such as music or spoken word"
      value "IMAGE", "A still image, such as a photo or graphic"
      value "TEXT", "Written words"
      value "VIDEO", "Motion picture, may have audio"
    end
    ```

    Each value may have:

    - A description (as the second argument or `description:` keyword)
    - A deprecation reason (as `deprecation_reason:`), marking this value as deprecated
    - A corresponding Ruby value (as `value:`), see below:

        By default, Ruby strings correspond to GraphQL enum values. But, you can provide `value:` options to specify a different mapping. For example, if you use symbols instead of strings, you can say:

        `value "AUDIO", value: :audio`

        Then, GraphQL inputs of `AUDIO` will be converted to `:audio` and Ruby values of `:audio` will be converted to `"AUDIO"` in GraphQL responses.

  - Defining GraphQL enums dynamically from Rails enums

    ```ruby
    module Types
      class IssuableSeverityEnum < BaseEnum
        graphql_name 'IssuableSeverity'
        description 'Incident severity'

        ::IssuableSeverity.severities.keys.each do |severity|
          value severity.upcase, value: severity, description: "#{severity.titleize} severity."
        end
      end
    end
    ```
</details>

### **Description for Type**

Write descriptions for Types

```ruby
class ExampleType < Types::BaseObject
  description "description goes here"
end
```

### **Description for Query**

Write description for Query

```ruby
class ExampleQuery < Queries::BaseQuery
  description "description goes here"
end
```

### **Descriptions for Mutations**

Write descriptions for Mutations

```ruby
class ExampleMutation < Mutations::BaseMutation
  description "description goes here"
end
```

---

## Handle deprecation

### **Fields deprecation**

- Deprecated fields can be marked by adding a `deprecation_reason:` keyword argument

    ```ruby
    field :name, String, null: true,
      deprecation_reason: "We split up the name into two; firstname and lastname"
    ```

- Have the deprecated field logged on Changelog

    Inlcude applicable Deprecated field, Reason, Date of deprecation, Date of removal in the description of the log.

    [![Deprecation](/assets/images/deprecation.png)](/assets/images/deprecation.png)

### **Arguments deprecation**

- Deprecated arguments can be marked by adding a `deprecation_reason:` keyword argument:

    ```ruby
    argument :name, String, required: false, deprecation_reason: "Use `first_name` instead."
    argument :first_name, String, required: false
    ```

    ⚠️ Note: argument deprecation is a stage 2 GraphQL [proposal](https://github.com/graphql/graphql-spec/pull/525) so not all clients will leverage this information.

- Use `as: :alternate_name` to use a different key from within your resolvers while exposing another key to clients.

    ```ruby
    field :post, PostType, null: false **do**
      argument :post_id, ID, required: true, as: :id
    **end

    def** **post**(id:)
      Post.**find**(id)
    **end**
    ```

- Have the deprecated argument logged on Changelog
