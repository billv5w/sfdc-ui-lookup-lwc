# Salesforce Lookup Component

[![Github Workflow](https://github.com/pozil/sfdc-ui-lookup-lwc/workflows/CI/badge.svg?branch=master)](https://github.com/pozil/sfdc-ui-lookup-lwc/actions) [![codecov](https://codecov.io/gh/pozil/sfdc-ui-lookup-lwc/branch/master/graph/badge.svg)](https://codecov.io/gh/pozil/sfdc-ui-lookup-lwc) ![a11y friendly](https://img.shields.io/badge/a11y-friendly-green)

> [!IMPORTANT]
> The Winter '24 release introduces a new Lightning base component that is similar to this lookup component: [`lightning-record-picker`](https://developer.salesforce.com/docs/component-library/bundle/lightning-record-picker/documentation) ([release announcement](https://developer.salesforce.com/blogs/2023/10/introducing-the-lightning-record-picker-component)).
> Here is a table to help you understand the key differences (non-exhaustive) between the two components:

|                     | This Lookup component                                                                                                                                    | `lightning-record-picker`                                                                                                                              |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Support**         | Custom component built as OSS. No official support.                                                                                                      | Base Lightning component with official Salesforce support.                                                                                             |
| **Mobile Support**  | None.                                                                                                                                                    | Built-in.                                                                                                                                              |
| **Data Source**     | You control the implementation (GraphQL or Apex) and you can work with any kind of data. It can list records or data that lives outside of the Platform. | You don't control the data source implementation (less code required). The component only selects Salesforce records with some filtering capabilities. |
| **Cache**           | You have to take care of caching as part of the implementation and you can use the Lightning Data Service.                                               | Built-in with Lightning Data Service caching and partial support for offline mode.                                                                     |
| **Permissions**     | Automatically enforced if using GraphQL. Can be enforced or bypassed if using Apex (use this with caution).                                              | Automatically enforced.                                                                                                                                |
| **Multi Object**    | Built-in. You can select records from multiple objects/sources.                                                                                          | Not supported but on the roadmap<sup>\*</sup>.                                                                                                         |
| **Multi Selection** | Built-in.                                                                                                                                                | Not supported but on the roadmap<sup>\*</sup>.                                                                                                         |
| **Default Results** | Customizable with the data of your choice: recently viewed records or other.                                                                             | Not customizable but recently viewed records are on the roadmap<sup>\*</sup>.                                                                          |
| **Custom Actions**  | Ability to create new records from the lookup.                                                                                                           | Not supported but on the roadmap<sup>\*</sup>.                                                                                                         |

_<sup>\*</sup> Forward-Looking Statements. No estimated date of delivery or guarantees._

---

<p align="center">
    <img src="screenshots/lookup-animation.gif" alt="Lookup animation"/>
</p>

</details>

<img src="screenshots/dropdown-open.png" alt="Lookup with dropdown open" width="350" align="right"/>

1. [About](#about)
1. [Installation](#installation)
1. [Documentation](#documentation)
    1. [Getting started](#getting-started)
    1. [Handling selection changes (optional)](#handling-selection-changes-optional)
    1. [Providing default search results (optional)](#providing-default-search-results-optional)
    1. [Saving form state when creating new records (optional)](#saving-form-state-when-creating-new-records-optional)
    1. [Passing custom data to JavaScript and Apex (optional)](#passing-custom-data-to-javascript-and-apex-optional)
1. [Reference](#reference)
1. [Special use cases](#special-use-cases)
    1. [Working with picklists values](#working-with-picklists-values)

## About

This is a generic &amp; customizable lookup component built using Salesforce [Lightning Web Components](https://developer.salesforce.com/docs/component-library/documentation/lwc) and [SLDS](https://www.lightningdesignsystem.com/) style.<br/>
It does not rely on third party libraries and you have full control over its datasource.

<b>Features</b>

The lookup component provides the following features:

-   customizable data source that can return mixed sObject types
-   single or multiple selection mode
-   client-side caching & request throttling
-   great test coverage
-   full accessibility (a11y) compliance
-   keyboard navigation
-   search term highlighting
-   ability to create new records

<p align="center">
    <img src="screenshots/selection-types.png" alt="Multiple or single entry lookup"/>
</p>

## Installation

The default installation installs the lookup component and a sample application available under this URL (replace the domain):<br/>
`https://YOUR_DOMAIN.lightning.force.com/c/SampleLookupApp.app`

If you wish to install the project without the sample application, edit `sfdx-project.json` and remove the `src-sample` path.

Install the sample app by running this script:

**MacOS or Linux**

```
./install-dev.sh
```

**Windows**

```
install-dev.bat
```

## Documentation

### Getting Started

Follow these steps to use the lookup component:

1. **Write the search endpoint**

    Implement an Apex `@AuraEnabled(cacheable=true scope='global')` method (`SampleLookupController.search` in our samples) that returns the search results as a `List<LookupSearchResult>`.
    The method name can be different, but it needs to match this signature:

    ```apex
    @AuraEnabled(cacheable=true scope='global')
    public static List<LookupSearchResult> search(String searchTerm, List<String> selectedIds) {}
    ```

1. **Import a reference to the search endpoint**

    Import a reference to the `search` Apex method in the lookup parent component's JS:

    ```js
    import apexSearch from '@salesforce/apex/SampleLookupController.search';
    ```

1. **Handle the search event and pass search results to the lookup**

    The lookup component exposes a `search` event that is fired when a search needs to be performed on the server-side.
    The parent component that contains the lookup must handle the `search` event:

    ```xml
    <c-lookup onsearch={handleSearch} label="Search" placeholder="Search Salesforce">
    </c-lookup>
    ```

    The `search` event handler calls the Apex `search` method and passes the results back to the lookup using the `setSearchResults(results)` function:

    ```js
    async handleSearch(event) {
        const lookupElement = event.target;
        try {
            const results = await apexSearch(event.detail);
            lookupElement.setSearchResults(results);
        }
        catch (error) {
                // TODO: handle error
        }
    }
    ```

### Handling selection changes (optional)

The lookup component exposes a `selectionchange` event that is fired when the selection of the lookup changes.
The parent component that contains the lookup can handle the `selectionchange` event:

```xml
<c-lookup onsearch={handleSearch} onselectionchange={handleSelectionChange}
    label="Search" placeholder="Search Salesforce">
</c-lookup>
```

The `selectionchange` event handler can then get the current selection from the event detail or by calling the `getSelection()` function:

```js
handleSelectionChange(event) {
    // Get the selected ids from the event (same interface as lightning-input-field)
    const selectedIds = event.detail;
    // Or, get the selection objects with ids, labels, icons...
    const selection = event.target.getSelection();
    // TODO: do something with the lookup selection
}
```

`getSelection()` always return a list of selected items.
That list contains a maximum of one element if the lookup is a single-entry lookup.

### Providing default search results (optional)

The lookup can return default search results with the `setDefaultResults(results)` function. This is typically used to return a list of recently viewed records (see sample app).

Here's how you can retrieve recent records and set them as default search results:

1.  Implement an Apex endpoint that returns the recent records:

    ```apex
    @AuraEnabled(cacheable=true scope='global')
    public static List<LookupSearchResult> getRecentlyViewed()
    ```

    See the [full code from the sample app](/src-sample/main/default/classes/SampleLookupController.cls#L59)

1.  In your parent component, create a property that holds the default results:

    ```js
    recentlyViewed = [];
    ```

1.  Write a utility function that sets your default search results:

    ```js
    initLookupDefaultResults() {
        // Make sure that the lookup is present and if so, set its default results
        const lookup = this.template.querySelector('c-lookup');
        if (lookup) {
            lookup.setDefaultResults(this.recentlyViewed);
        }
    }
    ```

1.  Retrieve the recent records by calling your endpoint:

    ```js
    @wire(getRecentlyViewed)
    getRecentlyViewed({ data }) {
        if (data) {
            this.recentlyViewed = data;
            this.initLookupDefaultResults();
        }
    }
    ```

1.  Initialize the lookup default results when the parent component loads:

    ```js
    connectedCallback() {
        this.initLookupDefaultResults();
    }
    ```

> [!NOTE]
> The `initLookupDefaultResults()` function is called in two places because the wire could load before the lookup is rendered.

### Saving form state when creating new records (optional)

The lookup component allows the user to create new record thanks to the optional `newRecordOptions` attribute. When users create a new record, they navigate away to the record edit form and they lose their current form input (lookup selection and more).

To prevent that from happening, you may provide an optional callback that lets you store the lookup state before navigating away. To do that, initialize the lookup new record options with a `preNavigateCallback` when the parent component loads:

```js
connectedCallback() {
    /**
     * This callback is called before navigating to the new record form
     * @param selectedNewRecordOption the new record option that was selected
     * @return Promise - once resolved, the user is taken to the new record form
     */
    const preNavigateCallback = (selectedNewRecordOption) => {
        return new Promise((resolve) => {
            // TODO: add some preprocessing (i.e.: save the current form state)

            // Always resolve the promise otherwise the new record form won't show up
            resolve();
        });
    };

    // Assign new record options with the pre-navigate callback to your lookup
    this.newRecordOptions = [
        { value: 'Account', label: 'New Account', preNavigateCallback },
        { value: 'Opportunity', label: 'New Opportunity', preNavigateCallback }
    ];
}
```

> [!TIP]
> Consider working with cookies to store information in a temporary state with an expiry date.

### Passing custom data to JavaScript and Apex (optional)

Sometimes, you may want to pass extra data from the lookup component to Apex. To do so, use [dataset](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset) attributes:

1. In the parent component that uses the lookup, add a dataset attribute (`data-custom` in this example):

    ```xml
    <c-lookup
        selection={initialSelection}
        onsearch={handleLookupSearch}
        label="Search"
        is-multi-entry={isMultiEntry}
        data-custom="My custom value"
    >
    ```

1. In the parent JS, use the dataset attribute that you just added:

    ```js
    handleLookupSearch(event) {
        const lookupElement = event.target;

        alert(lookupElement.dataset.custom); // My custom value

        // Actual search code
    }
    ```

## Reference

### Attributes

| Attribute             | Type                                                                                          | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Default         |
| --------------------- | --------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- |
| `disabled`            | `Boolean`                                                                                     | Whether the lookup selection can be changed.                                                                                                                                                                                                                                                                                                                                                                                                                                        | `false`         |
| `errors`              | `[{ "id": String, "message": String }]`                                                       | List of errors that are displayed under the lookup. When passing a non-empty list, the component is blurred.                                                                                                                                                                                                                                                                                                                                                                        | `[]`            |
| `isMultiEntry`        | `Boolean`                                                                                     | Whether the lookup is single (default) or multi entry.                                                                                                                                                                                                                                                                                                                                                                                                                              | `false`         |
| `label`               | `String`                                                                                      | Optional (but recommended) lookup label. Label is hidden if attribute is omitted but this breaks accessibility. If you don't want to display it, we recommend to provide a label but hide it with `variant="label-hidden"`.                                                                                                                                                                                                                                                         | `''`            |
| `minSearchTermLength` | `Number`                                                                                      | Minimum number of characters required to perform a search.                                                                                                                                                                                                                                                                                                                                                                                                                          | `2`             |
| `newRecordOptions`    | `[{ "value": String, "label": String, "defaults": String, "preNavigateCallback": Function }]` | List of options that lets the user create new records.<br/>`value` is an sObject API name (i.e.: "Account")<br/>`label` is the label displayed in the lookup (i.e.: "New Account").<br/>`defaults` is an optional comma-separated list of default field values (i.e.: "Name=Foo,Type\_\_c=Bar")<br/>`preNavigateCallback` is an optional callback used for [saving the form state](#saving-form-state-when-creating-new-records-optional) before navigating to the new record form. | `[]`            |
| `placeholder`         | `String`                                                                                      | Lookup placeholder text.                                                                                                                                                                                                                                                                                                                                                                                                                                                            | `''`            |
| `required`            | `Boolean`                                                                                     | Whether the lookup is a required field. Note: Property can be set with `<c-lookup required>`.                                                                                                                                                                                                                                                                                                                                                                                       | `false`         |
| `scrollAfterNItems`   | `Number`                                                                                      | A null or integer value used to force overflow scroll on the result listbox after N number of items.<br/>Valid values are `null`, `5`, `7`, or `10`.<br/>Use `null` to disable overflow scrolling.                                                                                                                                                                                                                                                                                  | `null`          |
| `selection`           | `[LookupSearchResult]` OR `LookupSearchResult`                                                | Lookup initial selection if any. Array for multi-entry lookup or an Object for single entry lookup.                                                                                                                                                                                                                                                                                                                                                                                 | `[]`            |
| `validity`            | `{ "valid": Boolean }`                                                                        | Read-only property used for datatable integration. Reports whether there are errors or not (see `errors`).                                                                                                                                                                                                                                                                                                                                                                          | `false`         |
| `value`               | `[LookupSearchResult]` OR `LookupSearchResult`                                                | Read-only property used for datatable integration. Alias of `selection`.                                                                                                                                                                                                                                                                                                                                                                                                            | `false`         |
| `variant`             | `String`                                                                                      | Changes the appearance of the lookup. Accepted variants:<br/>`label-stacked` - places the label above the lookup.<br/>`label-hidden` - hides the label but make it available to assistive technology.<br/>`label-inline` - aligns horizontally the label and lookup.                                                                                                                                                                                                                | `label-stacked` |

### Functions

| Function                     | Description                                                                                                                                    |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `focus()`                    | Places focus on the component an opens the search dropdown (unless this is a single selection lookup with a selection).                        |
| `blur()`                     | Removes focus from the component and closes the search results list.                                                                           |
| `getSelection()`             | Gets the current lookup selection as an array of `LookupSearchResult`.                                                                         |
| `setDefaultResults(results)` | Allows to set optional default items returned when search has no result (ex: recent items).<br/>`results` is an array of `LookupSearchResult`. |
| `setSearchResults(results)`  | Passes a search result array back to the lookup so that they are displayed in the dropdown.<br/>`results` is an array of `LookupSearchResult`. |

### Events

| Event             | Description                                                                                                                                                                                                                                                                           | `event.detail` Type                                                      |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `search`          | Event fired when a search needs to be performed on the server-side.<br/>`searchTerm` is the sanitized (lowercase, trimmed...) search term that should be sent to the server.<br/>`rawSearchTerm` is the unsanitized user input.<br/>`selectedIds` is an array of selected record Ids. | `{ searchTerm: String, rawSearchTerm: String, selectedIds: [ String ] }` |
| `selectionchange` | Event fired when the selection of the lookup changes. The event's `detail` property holds the list of selected ids.<br/>You can also use `target.getSelection()` to retrieve the selected lookup objects.                                                                             | `[ String ]`                                                             |

## Special use cases

### Working with picklists values

The lookup component can also be used to select values from picklist fields.

This Apex sample example demonstrates how you can query for the values of `Account.Industry`:

```apex
@AuraEnabled(cacheable=true scope='global')
public static List<LookupSearchResult> search(String searchTerm, List<String> selectedIds) {
    // Prepare query parameters
    searchTerm = '%' + searchTerm + '%';
    // Execute search query
    List<PicklistValueInfo> entries = [
        SELECT Label, Value
        FROM PicklistValueInfo
        WHERE
            EntityParticle.EntityDefinition.QualifiedApiName = 'Account'
            AND EntityParticle.QualifiedApiName = 'Industry'
            AND isActive = TRUE
            AND Label LIKE :searchTerm
            AND Value NOT IN :selectedIds
        LIMIT 5
    ];
    // Prepare results
    List<LookupSearchResult> results = new List<LookupSearchResult>();
    for (PicklistValueInfo entry : entries) {
        results.add(new LookupSearchResult(entry.Value, null, null, entry.Label, null));
    }
    return results;
}
```
