# RAML Update Guideline

Our APIs have to follow our
[API Design Guideline](https://github.com/commercetools/api-design-guidelines/blob/master/guidelines.md) to maintain a consistent developer experience.

## Table of Contents

- How to:
  - [Create a new Resources definition](#create-a-new-resources-definition)
    - [Create a Query Endpoint](#--create-query-endpoint)
    - [Create a Create Endpoint](#--create-a-create-endpoint)
    - [Create an Update Endpoint](#--create-an-update-endpoint)
    - [Create a Delete Endpoint](#--create-a-delete-endpoint)
  - [Create a Method](#create-a-method)
  - [Create a Type](#create-a-type)
  - [Create a Message](#create-a-message)
  - [Create a Field](#create-a-field)
  - [Create a Scope](#create-a-scope)
- Platform API folder structure
  - [/examples](#examples)
  - [/resources](#resources)
  - [/securitySchemes](#securityschemes)
  - [/traits](#traits)
  - [/types](#types)
    - [Resource Data with the related Draft](#--resource-data-with-the-related-draft)
    - [Paged Query Response](#--paged-query-response)
    - [Update and Update Action](#--update-and-update-action)
    - [/updates](#--updates)
    - [/common](#--common)
    - [/error](#--error)
    - [/message](#--message)
    - [annotations.raml](#--annotationsraml)
    - [types.raml](#--typesraml)
  - [api.raml](#apiraml)
- [Validator Rules](#validator-rules)
  - [AsMapRule](#asmaprule)
  - [CamelCaseRule](#camelcaserule)
  - [DatetimeRule](#datetimerule)
  - [DiscriminatorNameRule](#discriminatornamerule)
  - [DiscriminatorParentRule](#discriminatorparentrule)
  - [FilenameRule](#filenamerule)
  - [NamedBodyTypeRule](#namedbodytyperule)
  - [NamedStringEnumRule](#namedstringenumrule)
  - [NestedTypeRule](#nestedtyperule)
  - [PackageDefinedRule](#packagedefinedrule)
  - [PostBodyRule](#postbodyrule)
  - [PropertyPluralRule](#propertypluralrule)
  - [QueryParameterCamelCaseRule](#queryparametercamelcaserule)
  - [QueryParameterPlaceholderAnnotationRule](#queryparameterplaceholderannotationrule)
  - [ResourceCatchAllRule](#resourcecatchallrule)
  - [ResourceLowerCaseHyphenRule](#resourcelowercasehyphenrule)
  - [ResourcePluralRule](#resourcepluralrule)
  - [SdkBaseUriRule](#sdkbaseurirule)
  - [StringPropertySingularRule](#stringpropertysingularrule)
  - [SuccessBodyRule](#successbodyrule)
  - [UnionTypePropertyRule](#uniontypepropertyrule)
  - [UpdateActionNameRule](#updateactionnamerule)
  - [UriParameterDeclaredRule](#uriparameterdeclaredrule)

### How to:

#### Create a new Resources definition

1. Go to the [api.raml](./api/api.raml) file and add at the end preferably, like this:

```raml
/stores: !include resources/stores.raml
```

2. Then in the [/resources](./api/resources) folder, add a new file called with the name of the resource definition. This is just an example about the content:
   [stores.raml](./api/resources/stores.raml)

3. Add the different resource endpoints available to this file

##### Create a Query Endpoint

1. Go to the respective resources file in the [/resources](./api/resources) folder, add a new file called with the name of the endpoint and this is just an example about the content:
   [stores.raml](./api/resources/stores.raml), see in the [How to create a Method section](#create-a-method) to add methods.
2. In addition, create a folder in [/types](./api/types) which will contain the **updates** folder as well like here [/updates](./api/types/cart/updates).
3. Then, add the main types in the folder created see the [How to create a type section](#create-a-type).
4. Add, if already available, the update actions like [CartAddCustomLineItemAction.raml](./api/types/cart/updates/CartAddCustomLineItemAction.raml) and the related json file in [/examples](./api/examples) folder.

```raml
type:
  baseDomain:
    resourceType: Cart
    resourceQueryType: CartPagedQueryResponse
    resourceDraft: CartDraft
    whereExample: 'customerEmail = "john.doe@example.com"'
    sortExample: createdAt asc
(updateable): Cart
(deleteable): Cart
(createable): CartDraft
description: A shopping cart holds product variants and can be ordered.
get:
  (java-implements):
    'com.commercetools.api.models.PagedQueryResourceRequest<ByProjectKeyCartsGet,
    com.commercetools.api.models.cart.CartPagedQueryResponse>'
  securedBy: [oauth_2_0: { scopes: ['view_orders:{projectKey}'] }]
  queryParameters:
    customerId?:
      type: string
  responses:
    200:
      body:
        application/json:
          example: !include ../examples/carts.example.json
```

##### Create a Create Endpoint

In this case, it's pretty much like in the section [How to Create a Query Endpoint](#create-a-query-endpoint), the only difference is in the method to add.

So just as an example about the content:

1. in the respective resources file in the [/resources](./api/resources) folder, add a new file called with the name of the endpoint and this is just an example about the content, in this case was
   [orders.raml](./api/resources/orders.raml), and then add the **POST** method.
   The rest is like the points 2-3-4 of the section [How to Create a Query Endpoint](#create-a-query-endpoint). To have an overview see the example below:

```raml
type:
  baseDomain:
    resourceType: Order
    resourceQueryType: OrderPagedQueryResponse
    resourceDraft: OrderFromCartDraft
    whereExample: 'customerEmail = "john.doe@example.com"'
    sortExample: createdAt asc
(updateable): Order
(deleteable): Order
(createable): OrderFromCartDraft
description:
  An order can be created from a order, usually after a checkout process has
  been completed.
post:
  securedBy: [oauth_2_0: { scopes: ['manage_orders:{projectKey}'] }]
  is:
    - conflicting
  description: |
    Creates an order from a Cart.
    The cart must have a shipping address set before creating an order.
    When using the Platform TaxMode, the shipping address is used for tax calculation.
  body:
    application/json:
      example: !include ../examples/order-create.example.json
  responses:
    201:
      body:
        application/json:
          example: !include ../examples/order.example.json
```

##### Create an Update Endpoint

To create an update endpoint by URI parameters like Id or Key, it's necessary to add to the parts of the code mentioned in the sections [How to Create a Query Endpoint](#create-a-query-endpoint) and [How to Create a Create Endpoint](#create-a-create-endpoint), the following parts:

1. Add the URI parameter, in the example below is the **Id**
2. Add the **methodName**
3. Add the **post** method
   To have an overview see the example below:

```raml
/{ID}:
  (methodName): withId
  type:
    baseResource:
      uriParameterName: ID
      resourceType: Cart
      resourceUpdateType: CartUpdate
  post:
    securedBy: [oauth_2_0: { scopes: ['manage_orders:{projectKey}'] }]
    body:
      application/json:
        example: !include ../examples/cart-update.example.json
    responses:
      200:
        body:
          application/json:
            example: !include ../examples/cart.example.json
```

##### Create a Delete Endpoint

To create a delete endpoint by URI parameters like Id or Key, it's necessary to add to the parts of the code mentioned in the sections [How to Create a Query Endpoint](#create-a-query-endpoint) and [How to Create a Create Endpoint](#create-a-create-endpoint), the following parts:

1. Add the URI parameter, in the example below is the **Key**
2. Add the **methodName**
3. Add the **delete** method
   To have an overview see the example below:

```raml
/key={key}:
  (methodName): withKey
  type:
    baseResource:
      uriParameterName: key
      resourceType: Cart
      resourceUpdateType: CartUpdate
  delete:
    is:
      - dataErasure
    securedBy: [oauth_2_0: { scopes: ['manage_orders:{projectKey}'] }]
    responses:
      200:
        body:
          application/json:
            example: !include ../examples/cart.example.json
```

#### Create a Method

To create a new method in the endpoint, we need to:

1. modify the related file in the [/resources](./api/resources) folder;
2. add a section with the declaration of the new method in the **methodName** annotation;
3. add in the **type** section the **baseResource** definition which is composed by **uriParameterName** and **resourceType**;
4. add then, the **http method** response which is gonna be in the most cases **get** or **post**

Below an example:

```raml
/password-token={passwordToken}:
  (methodName): withPasswordToken
  type:
    baseResource:
      uriParameterName: passwordToken
      resourceType: Customer
  get:
    displayName: Get customer by password verification token
    securedBy: [oauth_2_0: { scopes: ['view_customers:{projectKey}'] }]
    responses:
      200:
        body:
          application/json:
            example: !include ../examples/customer.example.json
```

#### Create a Type

To create a new type, these are the files which have to be created:

1. The **BaseResources** which have the same name of the **package**, generally. There is also the definition of **updateType** and then there are in detail the definition of the **properties**. Here an example of the structure that each **BaseResources** have in common: [Category.raml](./api/types/category/Category.raml);
2. The **Draft**: it's the object submitted as payload to a create method. To follow the previous example [CategoryDraft.raml](./api/types/category/CategoryDraft.raml);
3. The **PagedQueryResponse**: here an example [CategoryPagedQueryResponse.raml](./api/types/category/CategoryPagedQueryResponse.raml), which is the extension of our Common API Type [PagedQueryResponse.raml](./api/types/PagedQueryResponse.raml);
4. The **Reference**: see [CategoryReference.raml](./api/types/category/CategoryReference.raml), which is the extension of our Common API Type [Reference.raml](./api/types/common/Reference.raml);
5. The **ResourceIdentifier**: see [CategoryResourceIdentifier.raml](./api/types/category/CategoryResourceIdentifier.raml), which is the "extension" of our Common API Type [ResourceIdentifier.raml](./api/types/common/ResourceIdentifier.raml) ;
6. The **Update**: this is the definition of the **updateType** declared in the **BaseResources**, see [CategoryUpdate.raml](./api/types/category/CategoryUpdate.raml), which is the "extension" of[Update.raml](./api/types/Update.raml);
7. The **UpdateAction**: this is the definition of the actions declared in the **Update** file, see [CategoryUpdateAction.raml](./api/types/category/CategoryUpdateAction.raml), which is the "extension" of [UpdateAction.raml](./api/types/UpdateAction.raml).
   To see the definition and the details of all of them, see our documentation.

##### Create a Message

To create a new Message, follow these steps:

1. Go in the [/types/message](./api/types/message) folder and create the file naming it _something_ + "Message.raml", like [CategoryCreatedMessage.raml](./api/types/message/CategoryCreatedMessage.raml)
2. Then, create the content like this below:

```raml
#%RAML 1.0 DataType
(package): Message
(docs-uri): https://docs.commercetools.com/http-api-projects-messages.html#categorycreatedmessage
type: Message
displayName: CategoryCreatedMessage
discriminatorValue: CategoryCreated
properties:
  category:
    type: Category
```

3. The MessagePayload type should be automatically created by the Github action. In case it has to be done manually create a file in the [/types/message/payload](./api/types/message/payload) folder and naming the file _something_ + "MessagePayload.raml",
   like [CategoryCreatedMessagePayload.raml](./api/types/message/payload/CategoryCreatedMessagePayload.raml)
4. And create the content like this below:

```raml
#%RAML 1.0 DataType
(package): Message
(docs-uri): https://docs.commercetools.com/http-api-projects-messages.html#categorycreatedmessage
type: MessagePayload
displayName: CategoryCreatedMessagePayload
discriminatorValue: CategoryCreated
properties:
  category:
    type: Category
```

5. Finally, include both files created in the [/types/types.raml](./api/types/types.raml). There is an automatism which in the CI pipeline to generate the related line in this file.

#### Add a new Field

To add a new field in the properties:

1. Go in the [/types](./api/types) folder and then in the resource related folder;
2. Add the name of the field accordingly to our naming convention guideline and in case it's **Optional** add `?`;
3. Identify if it's an object type and if it is, create a separated file which contains the fields of the object. Like for instance, in the Cart we have in the property the field **customLineItems** which is an object. So there is a file [CustomLineItem.raml](./api/types/cart/CustomLineItem.raml) which defines it in details.

```raml
  customLineItems:
    type: CustomLineItem[]
    description: ''
```

4. Add `(beta): true` in case of Beta feature related;
5. Add `(markDeprecated): true` in case of deprecation of a field;

#### Add a Scope

In the case you need to add a new **scope**, these are the steps that have to be done:

1. Go in the [/resources](./api/resources) folder and in the resource domain file like [categories.raml](./api/resources/categories.raml);
2. In the http methods which require to have it add the scope in the **oauth_2_0** par, like this:

```raml
get:
  securedBy:
    [
      oauth_2_0:
        {
          scopes:
            [
              'manage_project:{projectKey}',
              'view_products:{projectKey}',
              'view_categories:{projectKey}',
            ],
        },
```

3.  Then, you have to add the new **scope** in 4 files (see [/securitySchemes](#security-schemes) section to have more information about it):

         - [oauth2.raml](./api/securitySchemes/oauth2.raml),
         - [oauth2_password.raml](./api/securitySchemes/oauth2_password.raml),
         - [oauth2_refresh.raml](./api/securitySchemes/oauth2_refresh.raml)
         - [oauth2_anonymous.raml](./api/securitySchemes/oauth2_anonymous.raml)

    Precisely, in the **settings/scopes** section like in the example below:

```raml
settings:
  authorizationUri: https://auth.europe-west1.gcp.commercetools.com/oauth/token
  accessTokenUri: https://auth.europe-west1.gcp.commercetools.com/oauth/token
  authorizationGrants: [client_credentials]
  scopes:
    - 'manage_project:{projectKey}'
    - 'manage_products:{projectKey}'
    - 'view_products:{projectKey}'
    - 'manage_orders:{projectKey}'
```

### Platform API folder structure

#### /examples

This folder contains files in json format which are the response samples of the api call.
The update examples are divided per package, so like here: [CategoryChangeNameAction.json](./api/examples/Category/CategoryChangeNameAction.json),
while the generic api response are directly under the **examples** folder such as:

[category.example.json](./api/examples/category.example.json)

[category-create.example.json](./api/examples/category-create.example.json)

As convention, the names of the json files of the Actions are exactly the same of the related RAML in the [/types](#types) folder.

The RAML files related to the actions or the main endpoint, contain the path of these examples as described in details in the [/update](#update) section.

#### /resources

In the files of this folder, there are files which, the most of them, are named like our endpoint, and we define for each of our endpoints:

- Base Domain
- Resource Methods
- Query parameters
- URI parameters
- Responses

Like here: [categories.raml](./api/resources/categories.raml).

#### /securitySchemes

The common case, when these files in this folder have to be modified, is when we introduce a new scope under the **scopes** section in the **settings**.
In really rare cases, we change the **authorizationUri** and the **accessTokenUri**.

```raml
settings:
  authorizationUri: https://auth.europe-west1.gcp.commercetools.com/oauth/token
  accessTokenUri: https://auth.europe-west1.gcp.commercetools.com/oauth/token
  authorizationGrants: [client_credentials]
  scopes:
    - 'manage_project:{projectKey}'
    - 'manage_products:{projectKey}'
    - 'view_products:{projectKey}'
```

#### /traits

In this folder, there are some part of the code like query parameters which are redundant.
So they have to be created in this folder then declared in the [./api/api.raml](./api/api.raml) in the **traits:** section, so then they can be called everywhere.

These are the steps about how to create and set a trait:

- Create the file in the traits folder like this:
  [price-selecting.raml](./api/traits/price-selecting.raml).

- Define it in the [./api/api.raml](./api/api.raml), like here:

```raml
traits:
  priceSelecting: !include traits/price-selecting.raml
```

- After these steps, we can call the trait in the resource like here:
  [product-projections-search.raml](./api/resources/product-projections-search.raml).

```raml
get:
  is:
    - priceSelecting
```

#### /types

In this folder, there is the detailed data definition of all the resources that we are passing through our APIs.
In common in each of the file there are:

**(package)**: the package name matches the folder name and often the domain name, which it lives in;

**(docs-uri)**: the link of our documentation;

**displayName**: the name of the resource data;

**type**: it could be BaseResource, in case of the main resource data, or one of the type found in the common folder;

**property**: the fields of the resource data.

The most of the folders are named like our endpoints, but with the hyphens, and in each of them there are:

##### - Resource Data with the related Draft

The **resourceType** and the **resourceDraft** of our **baseDomain** are explained here in details such as the definition of the **properties** and also, there is the definition of the name of the **(updateType)**.

Here an example:

[CartDiscount.raml](./api/types/cart-discount/CartDiscount.raml)

[CartDiscountDraft.raml](./api/types/cart-discount/CartDiscountDraft.raml)

Our properties names are following our naming convention, see our
[API Design Guideline](https://github.com/commercetools/api-design-guidelines/blob/master/guidelines.md).

##### - Paged Query Response

The **resourceQueryType** of our **baseDomain** is written here in details.
Each new endpoint has to have this response defined, so this response has to be named: (_resource data name_) + _"PagedQueryResponse.raml"_ as implicit convention, here an example:
[CartDiscountPagedQueryResponse.raml](./api/types/cart-discount/CartDiscountPagedQueryResponse.raml)

##### - Update and Update Action

The Update is defined generically in the main resource data as **(updateType)** and here an example of update file:

```raml
(package): CartDiscount
type: object
displayName: CartDiscountUpdate
(java-extends): 'com.commercetools.api.models.ResourceUpdate<CartDiscountUpdate,
  CartDiscountUpdateAction>'
properties:
  version:
    type: number
    format: int64
  actions:
    type: array
    items: CartDiscountUpdateAction
```

The Update Action is defined in the Update file as actions properties,

```raml
properties:
  version:
    type: number
    format: int64
  actions:
    type: array
    items: CartDiscountUpdateAction
```

as well as in each Update actions as **type** which are in the [/updates](#--updates) folder:

```raml
(package): CartDiscount
(docs-uri): https://docs.commercetools.com/http-api-projects-cartDiscounts.html#change-name
type: CartDiscountUpdateAction
displayName: CartDiscountChangeNameAction
discriminatorValue: changeName
example: !include ../../../examples/CartDiscount/CartDiscountChangeNameAction.json
properties:
  name:
    type: LocalizedString
    description: ''
```

##### - /updates

As said in the previous point [Update and Update Action](#update-and-update-action), in this folders there are all the update actions available for each endpoint.
In each of them is defined the **examples**, so the json file which is present in the example folder and the properties.
There is an implicit convention about how to name these files: (_name of the package_) + (_update action_) + _"Action.raml"_ like _"CartDiscountChangeNameAction.raml"_.

##### - /common

In this folder, there is the definition of all of our object collections. Which is defined also in our documentation: [https://docs.commercetools.com/api/types](https://docs.commercetools.com/api/types).
Here a classic example

```raml
(package): Common
(docs-uri): https://docs.commercetools.com/http-api-types.html#localizedstring
displayName: LocalizedString
type: object
(asMap):
  key: string
  value: string
properties:
  /^[a-z]{2}(-[A-Z]{2})?$/:
    type: string
```

##### - /error

When we need to define an error, the related file is meant to be here in this folder.
The list of all of our error is also defined in our documentation [https://docs.commercetools.com/api/errors](https://docs.commercetools.com/api/errors).
There is an implicit convention about how to name these files: _name of the error_ + _"Error.raml"_.

##### - /message

As documented [https://docs.commercetools.com/api/message-types](https://docs.commercetools.com/api/message-types), we have messages that are defined in this folder.
If you write at least the Message, automatically it will generate the related Payload in the **/message/payload** folder.
There is an implicit convention about how to name these files: _name of the error_ + _"Message.raml"_.
The same dynamic is for the _Payloads_: _name of the error_ + _"MessagePayload.raml"_.

##### - annotations.raml

In this file, there is the definition of all the annotations which can be used in the RAML files.
They can be added in our RAML file like in this case and they are enclosed in parenthesis like here:

```raml
  lastModifiedBy?:
    type: LastModifiedBy
    (beta): true
```

If you see the need for additional annotations, please raise your request in the slack channel: **#raml**

##### - types.raml

Every new file created will be add automatically in this file.

#### api.raml

The api.raml file contains the base of our APIs and the definition of everything mentioned before, such as:

- the _baseUri_ is defined in base of the regions which are specified in the _baseUriParameters_;
- the _securitySchemes_: the path where we can find the default security scheme applied on all API endpoint (see [/securitySchemes](#security-schemes));
- the _annotationTypes_: where the annotations are defined (see [annotations.raml](#--annotationsraml));
- the _types_: where the types are defined (see [types.raml](#--typesraml));

### Validator Rules

We built a RAML validator tool which validates during the CI process the new code written related to our RAML files and for every error will show the related message.

If you want to run locally:
`yarn run raml:validate`.

The rules included are:

#### AsMapRule

This rule consists of checking the name of the property if it starts and end with _"/"_, it has to be annotated with **"(asMap)"** annotation.

```raml
types:
  LocalizedString:
    type: object
    (asMap):
        key: string
        value: string
    properties:
      /^[a-z]{2}$/: string
```

#### CamelCaseRule

In this case, it checks if the name of the property is written in camelCase, but it already excludes "error_description".
This means that the hyphens are not allowed on the property name level.

```raml
    properties:
      /a-z/: string
      camelCase: string
```

#### DatetimeRule

In the case the property type is **DateTime** or **TimeOnly** or **DateOnly**, this rule checks the name of the property:

- it has to finish with **At** or **From** or **To**;
- in case of date range, it has to have a property which finishes with **From** and a property which finishes with **To**.

```raml
  FooAtDateTime:
    type: object
    properties:
      fooAt: datetime
  FooRangeDateTime:
    type: object
    properties:
      fooFrom: datetime
      fooTo: datetime
```

#### DiscriminatorNameRule

In case we need to set a Discriminator Value, this validator rule checks if it has been set and if it contains it.
See here how to set up:

```raml
types:
  FooUpdateAction:
    type: object
    discriminator: type
    properties:
      type: string
  BarFoo:
    type: FooUpdateAction
    discriminatorValue: bar
```

#### DiscriminatorParentRule

This rule checks if the **Discriminator** type is set on the attribute level, it cannot be set as parent.
So, the discriminator type has not to be included in the base type.

```raml
types:
  Foo:
    type: object
    discriminator: type
    properties:
      type: string
  Baz:
    type: object
    properties:
      type: string
  FooBar:
    discriminator: type
    properties:
      type: string
  FooBaz:
    discriminator: type
    type: Foos
    properties:
      type: string
  Foos:
    properties:
      id: string
```

#### FilenameRule

This rule checks:

- if the new file has been included in the **types.raml** see the example below
- the **displayName** may be different
- name includes the domain/package

So we are creating a new file: FooBar.raml and this is what we will write in the types.raml file:

```raml
Foo: !include foo/FooBar.raml
```

#### NamedBodyTypeRule

This rule checks if it is defined for Success or for the request body in the methods of the body type.

```raml
get:
  body:
    application/json:
      example: !include ../examples/cart-create.example.json
  responses:
    201:
      body:
        application/json:
          example: !include ../examples/cart.example.json
```

#### NamedStringEnumRule

This rule validates that if the type is a **string**, and it does not have a pattern defined, it has to have **Enum** values like in this case:

```raml
types:
  Foo:
    type: string
    description: foo
    enum:
      - bar
  Bar:
    type: string
    pattern: /a-z/
```

#### NestedTypeRule

The nested types and properties are forbidden. So in the case an object type or an array type is defined, we accept only properties and the related types defined once.
In the example below, there are cases which are valid.

```raml
types:
  FooObject:
    type: object
    properties:
      meta:
        type: object
        description: valid
  Bar:
    type: object
    properties:
      valid: string
      validBaz: Baz
  FooArray:
    type: object
    properties:
      validArr:
        type: string[]
```

#### PackageDefinedRule

The **(package)** annotation has to be defined in case of creation of a new _Library_.

```raml
(package): Cart
displayName: Cart
(updateType): CartUpdate
type: BaseResource
properties:
  id:
    (identifier): true
    type: string
    description: The unique ID of the cart.
```

#### PostBodyRule

This rule is a complementary rule to the [Named Body Type](#named-body-type) Rule.
In this case check on for the _POST method_ if the body is correctly set.

```raml
/categories:
  get:
  post:
    body:
      application/json:
        type: object
```

#### PropertyPluralRule

This rule is valid not for all the properties, it's regulated in base of the English grammar.
But, in general, all the array property names have to be in plural.

```raml
types:
  Foo:
    type: object
    properties:
      validItems: string[]
      money: string[]
```

#### QueryParameterCamelCaseRule

This rule checks if the **Query Parameter** name is valid. It has to be composed by only alphanumeric and dot and lower camel cased.
It already excludes "error_description".

```raml
queryParameters:
  customerId?:
    type: string
  filter.query?: string
```

#### QueryParameterPlaceholderAnnotationRule

About this rule, this is how to define a **Query Parameter** with a Placeholder annotation:

- If the query parameter start and end with _"/"_ it has to have a **(placeholderParam)** annotation;
- The Placeholder object has to have declared the fields: _paramName_, _template_ and _placeholder_;

```raml
queryParameters:
  /text\.[a-z]{2}(-[A-Z]{2})?/:
    (placeholderParam):
      paramName: text
      template: text.<locale>
      placeholder: locale
```

#### ResourceCatchAllRule

This rule is on the resource level, it already excludes "inventory", "login", "me", "import" and "in-store".
It checks if for each path we have at least one parameter defined to catch the whole path.

```raml
/{projectKey}:
  /categories:
    /key={key}:
    /{id}:
```

#### ResourceLowerCaseHyphenRule

In this case, it checks if the resource is lowercased and hyphen separated

```raml
/{projectKey}:
  /valid:
    /key={key}:
    /{id}:
  /valid-resource:
    /{id}:
```

#### ResourcePluralRule

This rule is on the resource level, it already excludes "inventory", "login", "me", "import" and "in-store" and it's regulated in base of the English grammar.
But, in general, all the array resources names have to be in plural.

```raml
/{projectKey}:
  /categories:
  /inventory:
  /login:
  /me:
  /in-store:
```

#### SdkBaseUriRule

This rule checks on the API settings level if the **baseUri**, the annotation **(sdkBaseUri)** are declared.

```raml
annotationTypes:
  sdkBaseUri: string

baseUri: https://api.{region}.commercetools.com
(sdkBaseUri): https://api.europe-west1.commercetools.com
baseUriParameters:
```

#### StringPropertySingularRule

This rule is on the property level, and it's regulated in base of the English grammar.
The property which is not an array as a type, it has to be singular.

```raml
types:
  Foo:
    type: object
    properties:
      validItem: string
      /a-z/: string
      maxApplications: number
      hasChanges: boolean
```

#### SuccessBodyRule

On the http method level, it checks if it has the body type for success responses defined.

```raml
/categories:
  get:
    responses:
      200:
        body:
          application/json:
            type: object
  post:
    responses:
      201:
        body:
          application/json:
            type: object
```

#### UnionTypePropertyRule

This rule asserts that the usage of the Union type is not allowed for the property. To clarify better see the example below:

```raml
types:
  Foo:
    type: object
    properties:
      validItems: string
      invalidItem: string | number
      invalidItemDesc:
        description: test
        type: string | number
      /invalid[a-z]/: string | number
```

#### UpdateActionNameRule

The rule consists in verifying the name of the action, so it is valid if the type is referred to an **UpdateAction**.

```raml
types:
  UpdateAction:
    type: object
  ValidAction:
    type: UpdateAction
```

#### UriParameterDeclaredRule

In the resource the **uriParameters** have to be declared, so we have this rule which checks it.

```raml
resourceTypes:
    base:
        uriParameters:
            <<uriParameterName>>:
                type: string
        get:
```